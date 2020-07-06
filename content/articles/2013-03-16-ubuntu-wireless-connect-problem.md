---
title: ubuntu12.04不能连接cmcc等热点的解决办法
date: 2013-03-16
category: linux
tags: ubuntu
---

出现的现象是可以连接ChinaNet，但是连接不上cmcc，公司的路由也连接不上，上网搜索，说是ubuntu12.04的网卡驱动没有支持11n，具体没有深究，这里给出了解决方法。
<!-- excerpt -->

解决方发其实就是禁用无线的11n，首先是打开文件`/etc/modprobe.d/iwlwifi-disable11n.conf`，没有的话就建立一个把。填入以下内容。

    :::java
    options iwlwifi 11n_disable=1

接着重启计算机，连上应该就可以了，亲测可行。
