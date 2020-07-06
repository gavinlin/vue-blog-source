---
title: compile android4.4 on mavericks
date: 2013-11-09
category: Android
tags: mavericks
---

最新的 android4.4 终于出来了。曾着新安装的 mavericks ，在苹果的机器上体验了一下编译 android 系统，说实话，很愉快。
<!-- excerpt -->

官方没有提到 mavericks 中如何配置环境来编译 android 源代码，幸好还有强大的 xda 。

现在假设现在有一台刚开封的 osx mavericks ，我们将会一步一步配置成可以编译 android 源码。

#1.从苹果商店起步

我们需要从苹果商店下载 xcode5 ，因为我们需要 xcode 给我们带来需要的包和 command line tools 。这个 xcode 很大，小水管拖了几小时才拖了回来。

或者可以点击以下链接下载安装。

[Apple Xcode 5 for OS X 10.9 Mavericks][1]

[1]: https://developer.apple.com/xcode/

#2.安装 Java JDK 6

没错，是安装 JDK 6， 别贪新鲜下个 7 回来，会哭死你的。

下载和安装都很简单，apple 已经为我们准备好 dmg 了，点击以下链接。

[Java for OS X 2013-005][2]

[2]: http://support.apple.com/kb/DL1572?viewlocale=en_US

#3.安装 Brew 和相关的包

brew 就相当于 ubuntu 的 apt-get ，就算不是为了编译 android 也是程序员必装的软件。

安装 brew 也很简单 ，只要在控制台(Terminal)输入以下命令(除了官方提供的Terminal，强烈推荐iterm2 ) 

    :::c
    ruby <(curl -fsSk https://raw.github.com/mxcl/homebrew/go)

然后在 terminal 配置文件 (.bashrc [如果shell 是 bash 默认就是] 或者 .zshrc [如果 shell 是 zsh]) 加入下面变量

    :::c
    export BUILD_MAC_SDK_EXPERIMENTAL=1

接着在 Terminal 执行以下命令

    :::c
    brew outdated
    brew update
    brew doctor

如果屏幕出现 `Your system is ready to brew.` 恭喜，安装 brew 成功了。

接着在 Terminal 运行以下命令安装所需要的包。

    :::c
    brew install git coreutils findutils gnu-sed gnupg pngcrush python

#4.修复 Switch.pm 问题

如果不执行如下命令，在编译 android 源码的时候会出现错误。

> When upgrading to OS X Mavericks, Apple made some perl module deprecated, but we still need them to build Android!

意思就是 苹果丢弃某些 perl 模块，但是那些模块是编译 android 所必须的，所以我们要补回来。

在 Terminal 输入

    :::c
    brew install cpanm
    sudo cpanm Switch --force

#5.下载源码，编译系统

到了这里也没什么可以说的了，只要按照 android 的官方网址

[http://source.android.com/source/downloading.html][3]

[3]: http://source.android.com/source/downloading.html

简单的说下就是

1. 下载设置 repo
2. 划分一块大小写敏感的区域( [android官网有说][4] 看Creating a case-sensitive disk image 部分) , 42G才刚好够编译。
3. 跳到那个区域，执行 repo init 和 repo sync。等他妈的一两天。
4. source ，lunch，make

[4]: http://source.android.com/source/initializing.html
