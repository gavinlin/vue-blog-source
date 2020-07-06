---
title: android-jni技术-概述
date: 2013-03-05
category: Android
tags: jni
---

JNI(Java Native Interface)是java调用本地代码的接口技术，基本上我是用它来在android中调用C/C++，android提供了一套套件叫做NDK(Native Development Kit)，使我们可以很方便生成我们需要的目标文件。
<!-- excerpt -->

专门有网站是介绍使用JNI的，网址是[这里][1]

<br/>

##使用举例

<br/>

###JAVA端实现

<span class="label label-info">info</span> 首先需要明确apk还是运行在Dalvik虚拟机中的，我们暂时还不能编写纯粹C/C++的应用，除非不需要面向用户界面 <span class="label label-info">info</span>

我们可以指定一个类来完成JAVA和C的交互工作，例如`com.lingavin.jnisample.JavaToJni.java`

    :::java
    package com.lingavin.jnsample;
    
    public class JavaToJni {
    	static{
    		//系统加载 libnativejni.so  
    		System.loadLibrary("nativejni");
    	}
    	//对应native函数的接口，用native关键字来修饰	
    	public static native void intToJni(int number); 
    }

就这样一个简单的JAVA端部分完成了，程序通过调用`JavaToJni.intToJni(num)`，就可以将int传入C/C++处理。

<br/>

###native端实现

native端的实现才是重点内容，本次使用动态注册的方式和C语言来完成这项工作。

<span class="label label-info">info</span> jni本地代码的接口分为动态注册和静态注册，本文只体现动态注册 <span class="label label-info">info</span>

<span class="label label-info">info</span> 要注意用C和用C++写jni接口是有些区别的，主要是方法调用上，C调用env方法是(*env)->xxx(env,xxx),C++则为env->xxx(xxx) <span class="label label-info">info</span>

在本工程中创建一个`jni/`目录，进去后创建`native_sample_jni.c`，我们将在这个文件中完成代码编写。开始是JNI_OnLoad函数。

<span class="label label-info">info</span> JNI_OnLoad函数是必须要的，系统就是从这个函数开始注册工作的。这个可以被看做回调函数 <span class="label label-info">info</span>

    :::c 
    JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved){
    	JNIEnv* env = NULL;
    	jint result = -1;
    	if((*vm)->GetEnv(vm, (void**)&env, JNI_VERSION_1_4) != JNI_OK){
    		return result;
    	}
    
    	if(!registerNatives(env)){
    		return result;
    	}
    	result = JNI_VERSION_1_4;
    	return result;
    }

看到JNI_OnLoad函数中调用了registerNatives函数，那就看看这个函数干了些什么。

    :::c
    static int registerNatives(JNIEnv* env){
    	return registerNativeMethods(env,KClassPath,gMethods,
    			sizeof(gMethods)/sizeof(gMethods[0]));
    }

调用了registerNativeMethods函数，传入了`KClassPath`和`gMethod`，这两个是很重要的变量。先看看registerNativeMethods做了些什么。

    :::c
    static int registerNativeMethods(JNIEnv* env,const char* className,
    		JNINativeMethod* gMethods, int numMethods){
    	jclass clz;
    	clz = (*env)->FindClass(env, className);
    	if(clz == NULL){
    		return JNI_FALSE;
    	}
    
    	if((*env)->RegisterNatives(env, clz, gMethods, numMethods) < 0){
    		return JNI_FALSE;
    	}
    	return JNI_TRUE;
    }

从代码可以看到它的功能是根据存入的参数进行接口注册，那么这些参数是怎么定义的呢？重点来看看：

    :::c
    static const char* const KClassPath = "com/lingavin/jnisample/JavaToJni";
    // java_method,  (in)return,  native_method
    static JNINativeMethod gMethods[] = {
    	{"intToJni","(I)V", (void*)nativeIntToJni},
    };

kClassPath对应`java工程的包名+包含native函数的类名`，gMethods是一个数组，记录着`java端的方法+（传入参数）返回类型+本地对应函数`，这样我们的Java函数就和c函数对接起来了。

<span class="label label-info"> 写到这里，其实还有很多知识没有提及，例如gMethods中(I)V是什么意思，后面会讲的 <span class="label label-info">

但是本地代码只是声明，没有实现，我们需要自己写实现方法，例如

    :::c
    void nativeIntToJni(JNIEnv* env, jobject clazz, jint num){
    	__android_log_print(ANDROID_LOG_INFO,"TAG","native get num is %d",num);
    }

JNIEnv和jobject是需要的，这两个的意思是java环境和调用这个函数的对象，后面的jint就是java传进来的int。基本上这个工程就实现了。

<br/>

##具体知识

<br/>

###参数类型

JAVA和C的参数类型表如下

JAVA类型 | 描述符号 | 本地类型 | 备注
:------:|:------:|:------:|:------:
boolean|Z|jboolean|unsigned 8 bits
byte|B|jbyte|signed 8 bits
char|C|jchar|unsigned 16 bits
short|S|jshort|signed 16 bits
int|I|jint|signed 32 bits
long|J|jlong|signed 32 bits
float|F|jfloat|32 bits
double|D|jdouble|64 bits
int[]|[I|jIntArray|
String|Ljava/lang/String;|jstring|
Object|Ljava/lang/Object;|jobject|

<br/>

<span class="label label-warning">Warning</span> JAVA的char是16位无符号数，而C则是8位，如果你用char*来指向JAVA传进来的char数组，那就等着悲剧吧。<span class="label label-warning">Warning</span>


有一些数据类型可以不通过转换直接使用，但有一些则不能，例如数组，string和object等。

<br/>

##Android NDK

写完上面的代码，还需要把c文件编译成so,这个时候就要NDK出动了。NDK的介绍

>NDK包括了：

>1.从C / C++生成原生代码库所需要的工具和build files。

>2.将一致的原生库嵌入可以在Android设备上部署的应用程序包文件（application packages files ，即.apk文件）中。

>3.支持所有未来Android平台的一些列原生系统头文件和库

<br/>

为了编译so，需要编写Android.mk，在JNI目录新建`Android.mk`

    :::c
    LOCAL_PATH := $(call my-dir)
    include $(CLEAR_VARS)
    LOCAL_MODULE := nativejni
    LOCAL_SRC_FILES := native_sample_jni.c
    LOCAL_MODULE_TAGS :=debug
    LOCAL_ARM_MODE := arm
    LOCAL_LDFLAGS = -llog
    include $(BUILD_SHARED_LIBRARY)

编译出so后就可以编译apk,然后放到机器上调试看是否有打印信息了。

工程已经放到github上面，网址是:[https://github.com/gavinlin/jni-sample][2]

[1]: http://192.9.162.55/docs/books/jni/html/jniTOC.html
[2]: https://github.com/gavinlin/jni-sample
