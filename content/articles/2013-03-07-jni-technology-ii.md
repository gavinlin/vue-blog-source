---
title: Android-jni技术-II
date: 2013-03-07
category: Android
tags: jni
---

在[概述][1]中基本搭建了一个jni的工程，但是有很多细节都没有讨论，这篇文件就是进一步讨论JNI技术的具体方面的，包括String的传递及使用，java的属性和方法的使用和异常抛出等。
<!-- excerpt -->

* [String的使用](#string)
* [java的属性和方法](#function_method)
* [类的使用](#object)
* [异常抛出](#exception)

<br/>

<span id="string"></span>

##String的使用

我们都知道JAVA有`String`这个类可以很方便地使用字符串，而在c中我们通常是用`char*`，那么JNI是怎么在这两种类型里转换的呢？

在之前的工程添加以下代码：

    :::c
    static JNINativeMethod gMethods[] = {
    	{"intToJni","(I)V", (void*)nativeIntToJni},
    	{"conversation","(Ljava/lang/String;)Ljava/lang/String;", (void*)nativeConversation},
    };
    
    jstring nativeConversation(JNIEnv* env, jobject clazz, jstring fromJava){
    	const char* message;
    	message = (*env)->GetStringUTFChars(env,fromJava,NULL);
    
    	__android_log_print(ANDROID_LOG_INFO,"TAG","message from java: %s", message);
    
    	(*env)->ReleaseStringUTFChars(env, fromJava, message);
    	return (*env)->NewStringUTF(env, "nice to see you , i am c");
    }

可以看到`fromJava`是从java传进来的string，`message`是`jstring`转换为`char*`的容器。使用方法`GetStringUTFChars`进行转换，使用`ReleaseStringUTFChars`释放资源。

<span class="label label-warning">Warning</span> GetStringUTFChars和ReleaseStringUTFChars需要成对出现，要不会有内存泄漏的，native想保存字符串可以使用memcpy复制一份

顺便说说，貌似getStringRegion不用释放。对于JAVA的数组，也是不能直接使用的，需要做像String一样的处理。

*JNI String 方法的汇总*

<br/>

JNI Function| Description | Since
:-----:|:-----:|:-----:
GetStringChars ReleaseStringChars|Obtains or releases a pointer to the contents of a string in Unicode format. May return a copy of the string.|JDK1.1
GetStringUTFChars ReleaseStringUTFChars|Obtains or releases a pointer to the contents of a string in UTF-8 format. May return a copy of the string.|JDK1.1
GetStringLength|Returns the number of Unicode characters in the string.|JDK1.1
GetStringUTFLength|Returns the number of bytes needed (not including the trailing 0) to represent a string in the UTF-8 format.|JDK1.1
NewString|Creates a java.lang.String instance that contains the same sequence of characters as the given Unicode C string.|JDK1.1
NewStringUTF|Creates a java.lang.String instance that contains the same sequence of characters as the given UTF-8 encoded C string.|JDK1.1
GetStringCritical ReleaseStringCritical|Obtains a pointer to the contents of a string in Unicode format. May return a copy of the string. Native code must not block between a pair of Get/ReleaseStringCritical calls.|Java 2 SDK1.2
GetStringRegion SetStringRegion|Copies the contents of a string to or from a preallocated C buffer in the Unicode format.|Java 2 SDK1.
GetStringUTFRegion SetStringUTFRegion|Copies the content of a string to or from a preallocated C buffer in the UTF-8 format.|Java2 SDK1.2

<br/>

<span id="function_method"></span>

##java的属性和方法

    :::java
    	public native void callMethod();
    
    	private void callback(){
    		Log.i("JavaToJni","in Java");
    	}

    :::c
    void nativeCallMethod(JNIEnv* env,jobject clazz){
    	jclass cls = (*env)->GetObjectClass(env, clazz);
    	jmethodID mid = (*env)->GetMethodID(env, cls, 
    			"callback", "()V");
    	if(mid == NULL){
    		return;
    	}
    	(*env)->CallVoidMethod(env, clazz, mid);
    }

我们可以在c中调用java的方法，只要我们能在c中得到对象，一切看起来都变得很简单，例子是在c中调用了jtj对象的callback方法。

还有调用静态方法，调用属性等，就不一一列举了，可以看官方的[网站][3]

<br/>

<span id="object"></span>

##类的使用

以使用date类为例子

    :::c
    {"printTime","()V", (void*)nativePrintTime},
    
    void nativePrintTime(JNIEnv* env, jobject clazz){
    	//获得date类
    	jclass clazzdate = (*env)->FindClass(env,"java/util/date");
    	//获得date类的无参构造方法
    	jmethodID outputdateID = (*env)->GetMethodID(env,clazzdate,"<init>","()V");
    	//运行构造方法，获得一个date对象
    	jobject dateObj =(*env)->NewObject(env,clazzdate,outputdateID);
    	//获得getTime方法的methodID
    	jmethodID dateGetTime = (*env)->GetMethodID(env,clazzdate, "getTime","()J");
    	//执行这个对象的getTime方法，返回一个long值，达到目的
    	unsigned times = (*env)->CallLongMethod(env,dateObj, dateGetTime);
      __android_log_print(ANDROID_LOG_INFO,"TAG","get time : %u", times);
    }

看到其实跟在java调用对象类似的，主要要知道怎么做。

<br/>

<span id="exception"></span>

##异常抛出

本地代码可以捕获异常和抛出异常，我们可以在一个例子里面体验到。

    :::java
    	public native void exceptionCall() throws IllegalArgumentException;
    	
    	private void callbackThrowException() throws NullPointerException{
    		throw new NullPointerException("JavaToJni.callbackThrowException");
    	}

    :::c 
    void nativeExceptionCall(JNIEnv* env, jobject clazz){
    	jthrowable exc;
    	jclass cls = (*env)->GetObjectClass(env, clazz);
    	jmethodID mid = 
    		(*env)->GetMethodID(env, cls, "callbackThrowException", "()V");
    	if(mid == NULL)
    		return;
    	(*env)->CallVoidMethod(env, clazz, mid);
    	exc = (*env)->ExceptionOccurred(env);
    	if(exc){
    		jclass newExcCls;
    		(*env)->ExceptionDescribe(env);
    		(*env)->ExceptionClear(env);
    		newExcCls = (*env)->FindClass(env,
    				"java/lang/IllegalArgumentException");
    		if(newExcCls == NULL)
    			return;
    		(*env)->ThrowNew(env, newExcCls, "thrown from C");
    	}
    }

上述代码都干些什么？首先，java调用`exceptionCall`，到了C端，c调用了java的`callbackThrowException`，这个函数抛出了NullPointerException，c捕获后再向java端抛出IllegalArgumentException，最后java端捕获异常并打印异常。

基本就介绍到这里了，如果还要深入理解，请看概述提供的原版教程。

工程源代码: [https://github.com/gavinlin/jni-sample][2]

[1]: http://lingavin.com/blog/2013/03/05/jni-technology/
[2]: https://github.com/gavinlin/jni-sample
[3]: http://192.9.162.55/docs/books/jni/html/fldmeth.html
