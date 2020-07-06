---
title: 源码编译已root的android rom
date: 2013-04-29
category: Android
---

世界上所谓的root，其实就是用修改过的su，替换系统中受限制的su，达到为所欲为的效果。
<!-- excerpt -->

这个修改过的su，由ChainsDD大神提供，慷慨地在github中开源了，由于su的权利太大，他同时提供了一个权限管理软件Superuser.apk，通常需要一并装入系统。

如果你有android系统源码，并且想编译一个原生带root的系统，可以按照以下步骤进行。

在源代码的`system/extras`目录，很容易可以找到`su`这个文件夹，这就是我们需要替换的地方，上这个网址[https://github.com/ChainsDD/su-binary][1]，下载源码替换之，编译后会在`system/xbin`生成su，但是程序通常会在`system/bin`查找su，所以我们需要在`init.rc`做一个链接到`system/xbin/su`

    :::sh
    symlink /system/xbin/su /system/bin/su

再把`Superuser.apk`，放到`system/app`里面，通常我的做法是在源码`device`目录的指定型号里面编写`mk`来达到这个目的。例如：

    :::sh
    PRODUCT_COPY_FILES += \
    	$(LOCAL_PATH)/apk/Superuser.apk:system/app/Superuser.apk

编译并烧写系统，你会发现你的系统已经root了。

如果不太明白什么是root，可以到这个大神的FAQ页面:[http://androidsu.com/superuser/faq/][2]

[1]:https://github.com/ChainsDD/su-binary
[2]:http://androidsu.com/superuser/faq/
