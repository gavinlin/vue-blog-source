---
title: android:NDK调用系统接口的版本兼容问题
date: 2013-03-20 
category: Android
---

<!-- ![android_logo](http://farm9.staticflickr.com/8226/8572656190_46ea969040_m.jpg) -->

私有开源项目gplayer是一个android多功能播放器，其中画面显示调用本地接口surface和skia，音频播放调用了audiotrack。最大的问题是android每个版本的库和头文件都会不一样，不能做到编译一次，运行所有版本的目的。现在参考了另一个开源项目andless的做法，为调用android的本地库做一些兼容工作。

<!-- excerpt -->

<br/>

## 基本思想

开始时是直接调用surface和audiotrack的api，现在要把调用audiotrack和surface的api写在另外一个cpp文件中，就像给这些api包装了一下，然后单独给这个cpp编译成so，这个so是和系统库相关的，例如支持android2.3.3的so叫做`audiotrack10.so`和`surface10.so`。接着，我们的播放器根据系统的版本调用相应的so来完成功能。

<br/>

## 具体实现

现在以audiotrack为例子说明一下。这里只截取了部分代码，用来说明问题，有兴趣可到我的github页面下载整个工程。

 ```cpp
    #define FROM_ATRACK_CODE 1
    #include "audiotrack.h"
    #define TAG "AudioTrackWrapper"
    namespace android {
    	extern "C" {
    		static AudioTrack* track;
    
    		int AndroidAudioTrack_register() {
    			__android_log_print(ANDROID_LOG_INFO, TAG, "registering audio track");
    			track = new AudioTrack();
    			if(track == NULL) {
    				return ANDROID_AUDIOTRACK_RESULT_JNI_EXCEPTION;
    			}
    			__android_log_print(ANDROID_LOG_INFO, TAG, "registered");
    			return ANDROID_AUDIOTRACK_RESULT_SUCCESS;
    		}
    	}// extern "C"
    }; //namespace android
```

可以看到只要我们调用了`AndroidAudioTrack_regisger`就可以获得一个audiorack对象。注意到在`#include "audiotrack.h"`前面出现的宏定义了吗？这是一个重点内容，我们看看`audiotrack.h`里面是什么东东。

```cpp
    #ifdef __cplusplus
    extern "C" {
    #endif
    #ifdef FROM_ATRACK_CODE
    	int AndroidAudioTrack_register();
    #else
    	int (*AndroidAudioTrack_register)() __attribute__((weak));
    #endif
    
    #ifdef __cplusplus
    }
    #endif
```

<span class="label label-info">Info</span> 对于__attribute__不熟悉的话可以google一下，这里是声明了弱符号

好了，如果定义了`FROM_ATRACK_CODE`这个宏，包含的这个头文件就是简单的函数声明，如果没有这个宏，就定义一个`AndroidAudioTrack_register`的函数指针，这样做的作用是什么呢？看看我们是如何调用这个api的把。

```cpp
    #include "audiotrack.h"
    MediaPlayer::MediaPlayer(int sdkVersion){
    	sPlayer = this;
    	mSdkVersion = sdkVersion;
    	if(mSdkVersion >= ANDROID_JELLYBEAN_2){
    		libhandle = dlopen("/data/data/com.lingavin.gplayer/lib/libatrack17.so", RTLD_NOW);
    	}else if(mSdkVersion == ANDROID_GINGERBREAD){
    		libhandle = dlopen("/data/data/com.lingavin.gplayer/lib/libatrack10.so", RTLD_NOW);
    	}else if(mSdkVersion == ANDROID_JELLYBEAN){
    		libhandle = dlopen("/data/data/com.lingavin.gplayer/lib/libatrack16.so", RTLD_NOW);
    	}else{
    		libhandle = dlopen("/data/data/com.lingavin.gplayer/lib/libatrack14.so", RTLD_NOW);
    	}
    	if(libhandle){
    		AndroidAudioTrack_register = (typeof(AndroidAudioTrack_register)) dlsym(libhandle,"AndroidAudioTrack_register");
    	}
    }
```

看，我们在构造函数已经大干一场了。首先是根据传入的系统版本，打开相应的动态库，然后这个动态库的api到传给我们的函数指针，这样就对应上函数了。

最后就是Android.mk了，它告诉系统该如何编译文件。

```java
    include $(CLEAR_VARS)
    LOCAL_MODULE := atrack10
    LOCAL_CFLAGS += -O2 -Wall -DBUILD_STANDALONE -DCPU_ARM -DAVSREMOTE -finline-functions -fPIC -D__ARM_EABI__=1 -DOLD_LOGDH -DANDROID2_3
    LOCAL_SRC_FILES := audiotrack.cpp
    LOCAL_ARM_MODE := arm
    LOCAL_C_INCLUDES += \
    	$(LOCAL_PATH)/../include/android2_3 \
    	$(LOCAL_PATH)/../include/common
    LOCAL_LDLIBS := -llog \
    	/home/gavin/workspace/gplayer/jni/prebuilt/android2_3/libmedia.so \
    	/home/gavin/workspace/gplayer/jni/prebuilt/android2_3/libutils.so 
    include $(BUILD_SHARED_LIBRARY)
```

看到`LOCAL_C_INCLUDES`这一行，其引入的头文件就是android2.3的系统头文件，`LOCAL_LDLIBS`链接的系统库也是android2.3的。

<span class="label label-info">Info</span> LOCAL_LDLIBS写了绝对路径了，要不找不到动态库，为什么不用LOCAL_SHARED_LIBRARIE？因为不想程序包含这么多库了，直接调用系统的库就可以了。

<br/>

## 问题

可以遇见，随着android版本的更新，so会越来越多的。有没有其他更好的兼容解决办法？

