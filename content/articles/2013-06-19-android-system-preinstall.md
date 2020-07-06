---
title: android系统预安装可卸载应用程序
date: 2013-06-19
category: Android
---

##基本思路

要编译预置程序的android系统，网上的方法是把程序放到`/system/app`目录中，但是这样做的话，程序不可卸载，而且如果不同程序的动态链接库名字相同的话，这两个应用程序就不能同时安装。
<!-- excerpt -->

这里使用的方法是把程序放在`/data/app`目录，这样程序既可卸载，也可升级，更可共存，重新开机程序不会还原，恢复出厂设置后自动安装程序，一举n得。

流程是这样的：开机运行脚本检测`/data/app`目录是否有`preinstall.txt`这个文件，没有的话表示预置的程序没有安装，这时就从指定目录`/system/media/app`里把预置的apk复制到`/data/app`目录。

<br/>

##实现

首先是实现那个脚本。我把它起名为`init.gavin.preinstall.sh`

    :::sh
    #init.gavin.preinstall.sh 
    #system/core/rootdir/etc/
    #!/system/bin/sh
    cd /system/media/app

    #android shell script: check if preinstall.txt is exist
    if [ -s /data/app/preinstall.txt ]; then
        echo "don't need to copy preinstall files"
    else
        # scan all apk files under system/media/app
        apklist="$(ls)"
        for apkfile in ${apklist}; do
            #copy all apk file to data/app
            dd if=${apkfile} of=/data/app/${apkfile}
            # change the permission
            chmod 666 /data/app/${apkfile}
        done
    fi

问题来了，这个文件放在系统哪里？放到`/etc/`目录。

问题又来了，编译的时候怎么自动把他放到根文件系统的`/etc/`目录？**在system/core/rootdir/Android.mk文件中加入下面的代码**

    :::sh
    #system/core/rootdir/Android.mk
    include $(CLEAR_VARS)
    LOCAL_MODULE       := init.gavin.preinstall.sh
    LOCAL_MODULE_TAGS  := optional eng 
    LOCAL_MODULE_CLASS := ETC
    LOCAL_SRC_FILES    := etc/init.gavin.preinstall.sh
    include $(BUILD_PREBUILT)

下一个问题是怎样启动脚本？答案是可以通过`init.rc`文件

<p class="info">这里用的是rockchip方案</p>

    :::sh
    #device/rockchip/rk30sdk/init.rc
    service preinstall /system/bin/sh /system/etc/init.gavin.preinstall.sh
        class core
        oneshot

如果想知道init.rc的详细信息，可以查看`system/core/init/readme.txt`，这里的意思是作为service启动脚本，`class core`的意思是，这个服务属于core类，这个类会随机器启动而启动，相应的还有main类。`oneshot`就是这个服务只运行一次，不驻留。

似乎一切都很美好，但是你还需要为这个脚本添加运行的权限。

    :::c  
    //system/core/include/private/android_filesystem_config.h 
    { 00555, AID_ROOT,      AID_SHELL,     "system/etc/init.gavin.preinstall.sh" },

这样基本就完成了。

<br/>

##准备数据

接下来就是要准备数据，还是以rockchip方案为例。在`device/rockchip/rk30sdk`建立文件夹`thirdparty_app`。预置的apk都放在这里，不要忘记还有`preinstall.txt`

    :::sh
    #device/rockchip/rk30sdk/thirdparty_app/preinstall.txt
    don't delete this file!

增加配置宏方便修改是否带第三方应用编译。

    :::sh
    #device/rockchip/rk30sdk/BoardConfig.mk 
    BUILD_WITH_THIRDPARTY := true

device.mk 负责判断宏，true的话把软件复制到指定目录

    :::sh
    #device/rockchip/rk30sdk/device.mk 
    ########################################################
    #  ThirdParty
    ########################################################
    ifeq ($(strip $(BUILD_WITH_THIRDPARTY)),true)
    thirdparty_apps_files := $(shell ls $(LOCAL_PATH)/thirdparty_app | grep .apk)
    PRODUCT_COPY_FILES += $(foreach file, $(thirdparty_apps_files), \
            $(LOCAL_PATH)/thirdparty_app/$(file):system/media/app/$(file))

    PRODUCT_COPY_FILES += $(LOCAL_PATH)/thirdparty_app/preinstall.txt:system/media/app/preinstall.txt
    endif


