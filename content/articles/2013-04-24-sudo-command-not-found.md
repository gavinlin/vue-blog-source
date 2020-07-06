---
title: 运行sudo出现command not found
date: 2013-04-24
category: linux
tags: ubuntu
---

在使用ubuntu时发现一些奇怪的现象，明明定义了`PATH`，而且普通用户也可以调用，`sudo`却调用不了，报`command not found`的错误。
<!-- excerpt -->

google之后知道是编译sudo的时候加入了--with-secure-path，这个选项。

>--with-secure-path[=PATH]

>Path used for every command run from sudo(8).  If you don't trust the
>people running sudo to have a sane PATH environment variable you may
>want to use this.  Another use is if you want to have the "root path"
>be separate from the "user path."  You will need to customize the path
>for your site.  NOTE: this is not applied to users in the group
>specified by --with-exemptgroup.  If you do not specify a path,
>"/bin:/usr/ucb:/usr/bin:/usr/sbin:/sbin:/usr/etc:/etc" is used.

<br/>

知道了问题就好解决了，要么重新编译sudo，要么在环境配置文件里加一个alias

    :::sh
    alias sudo='sudo env PATH=$PATH'

附上sudo官网:[http://www.gratisoft.us/sudo/install.html][1]

[1]:http://www.gratisoft.us/sudo/install.html
