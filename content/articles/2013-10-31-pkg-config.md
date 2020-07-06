---
title: pkg-config
date: 2013-10-31
category: linux
tags: c
---

在编译例子的时候用到了 `pkg-config` 。例如
<!-- excerpt -->

    :::c
    $ gcc -o example example.c `pkg-config alsaplayer --cflags --libs`

这个 example 使用了 `libalsaplayer.so` ，只要运行上述命令，编译就可以通过。gcc 的使用已经熟悉了，但是 `pkg-config` 却不知道是什么东西。

最后通过 [wiki][wiki] 明白了这个家伙究竟是什么东西了。

`pkg-config` 和 `ls` 一样，是 可执行程序。作用是查询已安装库的各种信息。例如如果我们在终端输入

    :::c
    pkg-config alsaplayer --cflags --libs

就会输出下面的字符串

    :::c
    -I/usr/local/include/alsaplayer  -L/usr/local/lib -lalsaplayer -ldl

打印出了头文件的位置和连接库的位置和需要的链接库，再拼接之前的 `gcc -o example example.c` ，难怪能够编译了。

但是究竟 `pkg-config` 是如何得到这些信息的？

wiki 上面说了，在安装 alsaplayer 的时候，有一个 叫做 `alsaplayer.pc` 的文件被放到了 `/usr/local/lib/pkgconfig` 这个目录里面，打开这个 pc 后缀的文件，内容如下

    :::c
    prefix=/usr/local
    exec_prefix=${prefix}
    libdir=${exec_prefix}/lib
    includedir=${prefix}/include
    plugindir=${exec_prefix}/lib/alsaplayer
    alsaplayer_includedir=${prefix}/include/alsaplayer

    inputplugindir=${plugindir}/input
    outputplugindir=${plugindir}/output
    scopeplugindir=${plugindir}/scopes
    interfaceplugindir=${plugindir}/interface

    Name: AlsaPlayer
    Description: AlsaPlayer audio player with plugin support
    Version: 0.99.82
    Libs: -L${libdir} -lalsaplayer -ldl
    Cflags: -I${alsaplayer_includedir}

其中记录着各种信息，我们需要的 Libs 和 CFlags 就是在这里获得的。

最后还有一个问题 `\`` 这个符号起了什么作用？

这个符号不是单引号，是 tab 键 上面的反引号。被反引号括住的作用是执行反引号里面的内容。也就是执行命令 `pkg-config alsaplayer --cflags --libs`



[wiki]: http://zh.wikipedia.org/zh-cn/Pkg-config
