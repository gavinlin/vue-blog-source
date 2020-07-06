---
title: ubuntu增加新用户遇到的问题
date: 2013-04-27
category: linux
tags: ubuntu
---

使用`useradd`新增用户后使用`su [用户名]`，结果只有一个`$`，显然不是熟悉的bash，只好自己配了。
<!-- excerpt -->

用root用户来改变新增用户的终端为bash

    :::sh
    sudo passwd
    su
    usermod -s /bin/bash [username]

使用`echo $SHELL`看看是否修改成功。

然后你想在新用户里面使用`sudo`，结果跳出错误**xx is not in the sudoers file. This incident will be reported**

这时候还是需要请出root用户来解决。

    :::sh
    chmod u+w /etc/sudoers
    gedit /etc/sudoers
    #在root ALL=(ALL) ALL 下面添加
    [username] ALL=(ALL) ALL
    chmod u-w /etc/sudoers
