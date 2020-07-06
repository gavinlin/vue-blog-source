---
title: android-jni技术-回调
date: 2014-02-08
category: android
tags: jni
---

前面介绍了jni的基本概念和如何使用jni技术达到 Java 调用 C 的目的。

但是没有说 C 如何通过 jni 来调用 java 的方法，也就是如何实现回调。实现回调的作用很大，例如我们使用了 upnp 本地库，当库接收到消息便需要往 java 端传递。又例如使用了 C 实现了播放器，但是进度显示需要使用到 java。这些时候就需要用到回调了。
<!-- excerpt -->

##实现

先看看主要部分

    :::java
    static void callback_handler(char *s) {
        int status;
        JNIEnv *env;
        bool isAttached = false;
       
        status = gJavaVM->GetEnv((void **) &env, JNI_VERSION_1_4);
        if(status < 0) {
            LOGE("callback_handler: failed to get JNI environment, "
                 "assuming native thread");
            status = gJavaVM->AttachCurrentThread(&env, NULL);
            if(status < 0) {
                LOGE("callback_handler: failed to attach "
                     "current thread");
                return;
            }
            isAttached = true;
        }
        /* Construct a Java string */
        jstring js = env->NewStringUTF(s);
        jclass interfaceClass = env->GetObjectClass(gInterfaceObject);
        if(!interfaceClass) {
            LOGE("callback_handler: failed to get class reference");
            if(isAttached) gJavaVM->DetachCurrentThread();
            return;
        }
        /* Find the callBack method ID */
        jmethodID method = env->GetStaticMethodID(
            interfaceClass, "callBack", "(Ljava/lang/String;)V");
        if(!method) {
            LOGE("callback_handler: failed to get method ID");
            if(isAttached) gJavaVM->DetachCurrentThread();
            return;
        }
        env->CallStaticVoidMethod(interfaceClass, method, js);
        if(isAttached) gJavaVM->DetachCurrentThread();
    }

假设在 java 工程中有一个叫 callBack 的静态方法，接收 string 类型的参数。我们就可以通过调用例子中的方法实现。

    :::java
        jmethodID method = env->GetStaticMethodID(
            interfaceClass, "callBack", "(Ljava/lang/String;)V");

但是在这个主要的方法里面还有很多东西需要注意。

首先是 `gJavaVM`，看到 g 就知道是个全局变量指针。这个指针指向java虚拟机，这个程序运行时是不变的，所以我们可以使用全局变量保存它。而我们需要在 `JNI_OnLoad` 这个方法中获得这个指针。

第二个需要注意的变量是 `env` ，函数首先尝试使用 GetEnv 方法获取 env ，失败后再尝试使用 AttachCurrentThread 获得。这样做是因为 env 跟线程相关，每个线程拥有独立的 env 。

只要能拿到 env ，一切就变得简单了。但是还有一个变量没有提到，就是 `gInterfaceObject` ，这是个很重要的变量，其保存了其保存了 JNIExampleInterface 对象的指针。我们的 callBack 方法就是在这个对象中。在这个例子中，我们一开始就实例化了这个对象，并把对象保存到全局变量 gInterfaceObject 中。

    :::java

    jint JNI_OnLoad(JavaVM* vm, void* reserved)
    {
            ......
            initClassHelper(env, kInterfacePath, &gInterfaceObject);
            ......
    }

    void initClassHelper(JNIEnv *env, const char *path, jobject *objptr) {
        jclass cls = env->FindClass(path);
        if(!cls) {
            LOGE("initClassHelper: failed to get %s class reference", path);
            return;
        }
        jmethodID constr = env->GetMethodID(cls, "<init>", "()V");
        if(!constr) {
            LOGE("initClassHelper: failed to get %s constructor", path);
            return;
        }
        jobject obj = env->NewObject(cls, constr);
        if(!obj) {
            LOGE("initClassHelper: failed to create a %s object", path);
            return;
        }
        (*objptr) = env->NewGlobalRef(obj);

    }

initClassHelper 方法首先获得类的构造函数，接着通过 NewObject 实例化该类。

源代码可访问 github : [https://github.com/gavinlin/JNICallbackExample](https://github.com/gavinlin/JNICallbackExample)
