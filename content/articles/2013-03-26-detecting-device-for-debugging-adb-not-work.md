---
title: ubuntu发现不了android设备的高级解决办法
date: 2013-03-26 
category: Android
---

首先如果你按照官方的做法，发现不了android设备的话，这个方法可能可以帮助你，官方的方法可以看这个网址[http://source.android.com/source/initializing.html][1]的`Configuring USB Access`
<!-- excerpt -->

或许有些朋友会觉得这个方法not work,就像我这样，`adb device`死活检测不到设备，我也百思不得其解，我开始怪这个设备不好，因为其他的设备都能检测出。

但是有点不心死，去google找找是不是别人你同样遭遇，结果，在stackoverflow找到相同遭遇的人，并有热心大牛给出了方法，网址可见这里:[http://stackoverflow.com/questions/6484279/detecting-device-for-debugging-adb-does-not-work][2]

我把方法搬运到这个blog里吧

首先是查看你的设备的vendorid,相信经过以前的折腾，你会记得`lsusb`这个命令

    :::sh
    Bus 002 Device 003: ID 0e0f:0002 VMware, Inc. Virtual USB Hub
    Bus 002 Device 002: ID 0e0f:0003 VMware, Inc. Virtual Mouse
    Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 001 Device 007: ID 2207:0006  
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

看到我的设备usb的vendorid是2207,然后打开`~/.android/adb_usb.ini`这个文件，没有就创建一个吧。添加下面的内容

    :::sh
    0x2207

没错就是填入vendorid。这样后`adb kill-server`然后再`adb devices`，应该就可以看到设备了。

然后大牛还有个友情提醒，更新sdk，这个文件可能会被覆盖，请重新上述步骤。

[1]: http://source.android.com/source/initializing.html
[2]: http://stackoverflow.com/questions/6484279/detecting-device-for-debugging-adb-does-not-work
