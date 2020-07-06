---
title: 自编写程序作为android系统app的编译记录
date: 2013-04-24
category: Android
---

本blog记录了把自己编译的软件当做系统的app随着android的编译而加入到系统。

主要还是编写Android.mk 的问题，还有资源存放问题，这点系统编译比sdk编译要严格。
<!-- excerpt -->

<br/>

##关于资源

有些人喜欢在xml里面写硬编码，例如`android:text="hello world"`，这些在系统编译的时候一定会报错的，为了养成良好的习惯，方便以后做国际化，字符串应该放到`strings.xml`里面，而诸如style,color,甚至常用的数字常量都应该放入相应的文件夹。

<br/>

虽然能够编译了，可能还会发生资源找不到的错误。大多是因为图片资源乱放的原因，例如 `drawable` `drawable-ldpi` `drawable-mdpi` `drawable-hdpi` 这几个常用的文件夹，android的图片加载策略是先看屏幕密度究竟是ldpi还是mdpi,又或者是hdpi,然后在该文件夹下找图片资源，找不到的话，再到drawable里面找图片资源，再找不到的话就报错，它不会遍历所有文件夹来寻找图片的，这就要求我们图片要方正确了，要兼容好的话最好每个文件夹都放上对应分辨率的图片。

<br/>

##Android.mk

系统编译脚本可以参考其他的系统应用程序，例如Setting。其中有一个注意的地方是如果引用了第三方的包，可以使用`BUILD_MULTI_PREBUILT`来把包引进编译流程，下面就是一个例子，其中还有aidl的参考编译方法。

还有一点就是如果需要系统权限，加上`LOCAL_CERTIFICATE := platform`。

    :::java
    LOCAL_PATH:= $(call my-dir)
    include $(CLEAR_VARS)
    
    LOCAL_MODULE_TAGS := optional
    
    LOCAL_STATIC_JAVA_LIBRARIES := \
            android-common \
            guava \
            android-support-v13 \
            android-support-v4 \
    		android-log4jlib \
    		jcifslib \
    
    LOCAL_SRC_FILES := \
            $(call all-java-files-under, src) 
    
    LOCAL_PACKAGE_NAME := package-test 
    
    #LOCAL_PROGUARD_FLAG_FILES := proguard.flags
    
    LOCAL_CERTIFICATE := platform
    
    include $(BUILD_PACKAGE)
    
    include $(CLEAR_VARS)
    LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := android-log4jlib:libs/android-logging-log4j-1.0.3.jar \
    	jcifslib:libs/jcifs-krb5-1.3.17.jar \
    	
    include $(BUILD_MULTI_PREBUILT)
    
    include $(CLEAR_VARS)
    LOCAL_MODULE_TAGS := optional
    LOCAL_SRC_FILES := src/com/xx/xx.aidl
    LOCAL_MODULE := com.xx.aidl
    include $(BUILD_STATIC_JAVA_LIBRARY)

<br/>

##最后

不要忘记在`build/target/product/core.mk`，文件里添加你的应用名称，这样才能在编译后自动安装到`system/app`目录下
