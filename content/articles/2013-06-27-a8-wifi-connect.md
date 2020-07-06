---
title: 嵌入式开发板配置无线上网总结
date: 2013-06-27 
category: linux
tags: ubuntu
---

手头有一块tiny210，但是没有网线，不能有线上网，只能为其设置无线网卡上网了。
<!-- excerpt -->

配置上网的环境有这些

1. 开发板用的debain根文件系统和kernel linux 3.0.8

2. wifi网卡，芯片型号是RTL8188CUS

3. 串口(主要交互工具了)


首先是要去下载无线网卡的驱动和相关工具：[点这里][1]，找到RTL8188CUS，这一项的Unix(Linux)，下载文件，看介绍支持Linux Kernel 2.6.18~2.6.38 和 Kernel 3.0.8

得到`RTL8192xC_USB_linux_v3.4.4_4749.20121105.zip`这个文件，使用`unzip`解压它。

进去后发现东西很齐全。主要使用的文件夹有三个`driver` `wireless_tools`和`wpa_supplicant_hostapd`

<br/>

##driver

进去后又是一个压缩包，如果使用的是友善提供的内核源码，里面已经包含了这款芯片的驱动了，无需再折腾，否则的话，就需要这个文件夹里面的文件了。此处先占个坑。


<br/>

##编译wireless_tools

编译wireless 比较简单，修改makefile的相关变量就可以了。

    :::sh
    #由于这些工具需要安装到debain文件系统中，所以需要配置PREFIX，让Makefile知道编译好的文件放在哪里
    ifndef PREFIX
      PREFIX = /home/gavin/workspace/min210/debain_rootfs
    endif

    ## Compiler to use (modify this for cross compile).
    #如果配置了PATH，使用友善提供的交叉工具链，才可以这样写。
    CC = arm-none-linux-gnueabi-gcc
    ## Other tools you need to modify for cross compile (static lib only).
    AR = arm-none-linux-gnueabi-ar
    RANLIB = arm-none-linux-gnueabi-ranlib

简单的`make && make install`就可以了，执行完后发现文件系统多了下面一些文件

    :::sh
    include/
    lib/libiw.a
    man/
    sbin/ifrename
    sbin/iwconfig
    sbin/iwevent
    sbin/iwgetid
    sbin/iwlist
    sbin/iwpriv
    sbin/iwspy

这些都是无线网卡有用的工具，后面会用到。

<br/>

##编译wpa_supplicant

wpa_supplicant 这个工具主要是用来配置ap和wap加密的无线网络连接，可以说是无线网卡必不可少的工具。

但是编译这个工具有点麻烦。首先解压缩`wpa_supplicant_hostapd-0.8_rtw_20120803.zip`，主要关于两个文件夹`patches` `wpa_supplicant`。

要编译wpa_supplicant，首先需要openssl的支持，所以我们先编译openssl，patches里面的补丁是为openssl的编译而打的。

首先到[http://www.openssl.org/source/][2]，下载openssl源码，我下载的是openssl-0.9.8e.tar.gz 。

新建一个文件夹`openssl`，把压缩文件放到里面解压，同时把patches里面的`openssl-0.9.8e-tls-extensions.patch`也放到`openssl`文件夹中，然后进入解压缩好的`openssl-0.9.8e`执行以下操作

    :::sh
    patch -p1 < ../openssl-0.9.8e-tls-extensions.patch

如果你看到以下信息

    :::sh
    patching file ssl/Makefile
    patching file ssl/s3_clnt.c
    patching file ssl/s3_srvr.c
    patching file ssl/ssl.h
    patching file ssl/ssl_err.c
    patching file ssl/ssl_sess.c
    patching file ssl/t1_ext.c
    patching file ssl/t1_lib.c
    patching file ssl/tls1.h
    patching file util/ssleay.num

表示补丁已经打好了。

然后编辑Makefile

    :::sh
    INSTALL_PREFIX=/home/gavin/workspace/min210/debain_rootfs
    CC= arm-none-linux-gnueabi-gcc
    AR=arm-none-linux-gnueabi-ar $(ARFLAGS) r
    RANLIB= arm-none-linux-gnueabi-ranlib

接着就是例牌的`make && make install`，会在指定目录生成以下文件夹

    :::sh
    usr/local/ssl/

openssl总算编译完了，接着是主角，进入`wpa_supplicant`目录

    :::sh
    cp defconfig .config

编辑`.config`

    :::sh
    #CONFIG_DRIVER_NL80211=y

    CC= arm-none-linux-gnueabi-gcc -L/home/gavin/workspace/min210/debain_rootfs/usr/local/ssl/lib/
    CFLAGS += -I/home/gavin/workspace/min210/debain_rootfs/usr/local/ssl/include/
    LIBS += -L/home/gavin/workspace/min210/debain_rootfs/usr/local/ssl/lib/

然后就可以`make`了，编译好的工具在源码目录，需要把`wpa_cli` `wpa_passphrase` `wpa_supplicant`拷贝到根文件系统的`sbin`目录。

到这里，所有编译的工作就完成了。

<br/>

##上网实践

如果驱动正常工作，应该可以看到`/proc/net/wlan0`。我们使用命令`iwlist wlan0 scan`，可以看到一串的信息，这些就是无线网卡检测到的附近的热点。

如果热点是wep加密的，只要使用命令`iwconfig wlan0 essid "name" key xxxx`，就可以连接上了，其中name是热点名称，xxxx是密码。

但是如果是wpa加密的话，上面的命令就用不了了，这个时候就要出动到我们辛苦编译出来的的`wpa_supplicant`了。

先到源码目录找到`wpa_supplicant.conf`这个文件，把所有example注释掉，大概在564行以下。加上自己热点的信息，例如

    :::sh
    network={
        ssid="ap_name"
        key_mgmt=WPA-PSK WPA-EAP
        psk="passwordhere"
        priority=5
    }

ssid就是热点名字，key_mgmt是加密方式，psk就是热点密码了，修改好后，放到开发板的`/etc/`目录。

接着在开发板运行这个命令

    :::sh
    wpa_supplicant -iwlan0 -B -c /etc/wpa_supplicant.conf

这个时候应该就可以连接上了。

但是还是访问不到网络？那就运行一下这个命令，使用dhcp获取ip吧。


    :::sh
    dhclient wlan0

开机启动貌似编辑以下文件，没仔细研究，挖坑。

    :::sh
    /etc/network/interfaces


[1]: http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PFid=48&Level=5&Conn=4&ProdID=228&DownTypeID=3&GetDown=false&Downloads=true

[2]: http://www.openssl.org/source/
