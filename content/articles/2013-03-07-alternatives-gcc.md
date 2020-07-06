---
title: 'ubuntu-gcc版本切换'
date: 2013-03-07 
category: linux
tags: ubuntu
---

<!-- ![gcc-logo]( http://farm9.staticflickr.com/8511/8536187790_2a20cc8577_m_d.jpg) -->

随着ubuntu版本的更新，内置的gcc也随着更新了，可是有时后我们用不到那么高版本的gcc，这时候我们就需要为ubunut切换低版本的gcc了，网上有些方法比较暴力，也比较麻烦，现在我推荐一种智能的方法。

<!-- excerpt -->

首先我们用`gcc -v`查看一下我们用的是什么版本的gcc，同时可以使用`ls /usr/bin/gcc*`来查看机器装了什么版本的gcc。

如果系统没有我们想要的gcc版本，可以通过`apt-get`来获得，例如:

    :::sh
    sudo apt-get install gcc-4.5 gcc-4.5-multilib g++-4.5 g++4.5-multilib

<p class="info">multilib 不能省略，一并安装吧</p>

下面就说说这么进行转换，例如我们想把gcc转换为4.4版本：

    :::sh
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 50
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.5 40

接这输入：

    :::sh
    sudo update-alternatives --config gcc

你会看到像下面的选项

    :::sh
    有 3 个候选项可用于替换 gcc (提供 /usr/bin/gcc)。
    
    选择 路径 优先级 状态
    * 0 /usr/bin/gcc-4.4 50 自动模式
    1 /usr/bin/gcc-4.4 50 手动模式
    2 /usr/bin/gcc-4.5 40 手动模式
    3 /usr/bin/gcc-4.6 30 手动模式

要维持当前值[*]请按回车键，或者键入选择的编号：

选择你想要的gcc版本就好了，**g++设置的方法相同**。

如果想删除某个gcc版本的选项的话，可以使用

    :::sh
    sudo update-alternatives --remove gcc /usr/bin/gcc-4.5

是不是觉得方便他可靠很多了呢，不需要做什么软链接，也方便管理，赶快使用`gcc -v`查看一下修改是否成功吧。

