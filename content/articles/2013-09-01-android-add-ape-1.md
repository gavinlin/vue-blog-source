---
title: 向android系统添加ape音频格式（第一部分）
date: 2013-09-01
category: Android
---

要向Android系统添加一种音频格式，需要对stagefright有一定的了解，在android源码里的`libstagefright`文件夹里面，可以看到一大堆的Extractor，也就是各种格式的解码器了。我们就是要这里写一个`APEExtractor`。
<!-- excerpt -->

撇开Android框架不说，我们还不知道要如何把APE格式的数据解为pcm，所以重中之重是先找到解析APE的方法。

<br/>

##Monkey's audio

开源的力量很强大，APE的解码器不用自己苦逼的实现了，只要到[http://www.monkeysaudio.com/][1]下载源码，然后根据android平台修改下，编译成库，供框架调用就可以了。

说起来轻松，有些知识还是需要知道的。所以现在的目标就是把APE解码器编译成`libmonkeysaudio.so`

<br/>

##Android.mk

要编译成库，最主要是写好Android.mk。下载下来的解码器的源码版本是4.12，也就是`MAC_SDK_412.zip`，解压后发现其代码没有Makefile，而且project只有vs2012和xcode两个版本。忍了。

Readme.txt 中有这么一句话

>If you use C++, it's recommended that you simply statically link to maclib.lib

所以熟悉其源码后发现用到的代码目录只有两个`MACLib`和`Shared`，其他可以扔垃圾桶了。

把这两个文件夹放到一个叫做`libmonkeysaudio`文件夹中，然后移到android源码中external目录，开始做代码修改。

首先是在MACLib 目录把md5.h改名为MD5.h，在Shared目录的All.h 里定义好环境为LINUX。

在`libmonkeysaudio`文件夹下编写Android.mk文件，内容如下

    :::sh
    LOCAL_PATH:= $(call my-dir)

    include $(CLEAR_VARS)

    LOCAL_MODULE:=libmonkeysaudio

    LOCAL_SRC_FILES := \
        MACLib/APECompressCore.cpp \
        MACLib/APECompress.cpp \
        MACLib/APECompressCreate.cpp \
        MACLib/APEDecompress.cpp \
        MACLib/APEHeader.cpp \
        MACLib/APEInfo.cpp \
        MACLib/APELink.cpp \
        MACLib/APESimple.cpp \
        MACLib/APEtag.cpp \
        MACLib/BitArray.cpp \
        MACLib/MACLib.cpp \
        MACLib/MACProgressHelper.cpp \
        MACLib/md5.cpp \
        MACLib/NewPredictor.cpp \
        MACLib/NNFilter.cpp \
        MACLib/Prepare.cpp \
        MACLib/UnBitArrayBase.cpp \
        MACLib/UnBitArray.cpp \
        MACLib/WAVInputSource.cpp \
        Shared/CharacterHelper.cpp \
        Shared/CircleBuffer.cpp \
        Shared/GlobalFunctions.cpp \
        Shared/StdLibFileIO.cpp \
        Shared/WinFileIO.cpp 

    LOCAL_C_INCLUDES += \
        external/libmonkeysaudio/MACLib \
        external/libmonkeysaudio/Shared 

    include $(BUILD_SHARED_LIBRARY)

PS:好好利用下面命令可以快速整理好要编译的文件。

    :::sh
    ls -l | awk '{print $9}' > all.txt

使用mm编译，毫无以外的一大堆错误，看来代码还是不完善，只好手工修改了。

<br/>

###错误一
external/libmonkeysaudio/Shared/SmartPtr.h:67:5: error: '__forceinline' does not name a type

老老实实去提示的代码那里把`__forceinline`改外`inline`，谁叫你不是编来给windows用的呢。

<br/>

###错误二

has virtual functions and accessible non-virtual destructor

说是析构函数不是虚拟的。那就在MACLib.h 的100行后加个虚拟的虚构函数就可以了。

    :::cpp
	virtual ~IAPEProgressCallback(){};

<br/>

###错误三

error: exception handling disabled, use -fexceptions to enable

在Android.mk 加上下面代码开启异常捕获

    :::sh
    LOCAL_CPPFLAGS += -fexceptions

<br/>

###错误四

 error: extra qualification 'APE::CCircleBuffer::' on member 'GetDirectWritePointer' [-fpermissive]

在Android.mk 加入 

    :::sh
    LOCAL_CPPFLAGS += -fpermissive

<br/>

###错误五

一堆链接错误，例如

error: undefined reference to '__cxa_end_cleanup'

这些源代码需要一个完整的c++静态库，于是在Android.mk加上这些链接库

    :::sh
    LOCAL_LDFLAGS += -Lprebuilts/ndk/current/sources/cxx-stl/gnu-libstdc++/libs/armeabi-v7a -lsupc++

这样，在Android4.2.2上就能编过了。

    :::sh
    Install: /home/gavin/workspace/min210/cm10/out/target/product/tiny210/system/lib/libmonkeysaudio.so 

到此编译解码库成功了。


[1]:http://www.monkeysaudio.com/
