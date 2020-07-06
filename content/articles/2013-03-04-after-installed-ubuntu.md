---
title: 安装ubuntu12.04后...
date: 2013-03-04 23:33
category: linux
tags: ubuntu
---

生命在于折腾，这次除了折腾博客，也折腾了一下系统，ubuntu12.04是一个非常棒的系统，因为它解决了很多以前无解的问题，例如：我笔记本的亮度调节问题，显卡发热问题。而且软件也做了很多调整，gvim竟然是7.3的，而且支持ibus输入中文，可以抛弃vimim插件了。
<!-- excerpt -->

但是这一切并不是一安装好就有的，需要去配置，这就是为什么写这篇blog。

<br/>

##显卡驱动，亮度调节和保存问题

笔记本的配置是双显卡，6750m和核心显卡，dv4-3115，现在的闭源驱动已经可以很好地工作了。

首先去[这里](http://www.amd.com/us/Pages/AMDHomePage.aspx)下载驱动。我下载的是13.01版本。

>还要使用 `sudo apt-get install ia32-libs` 安装32位库，如果你的系统是64位的话。

然后就可以像装windows软件一样安装驱动了，安装完后执行 `sudo aticonfig --inistial -f` ，然后重启，检测是否成功的命令是 `fglrxinfo` 和 `fgl_glxgears` 。

安装完后打开`/etc/default/grub` 改一行语句：

    :::sh
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
    改为
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash acpi_backlight=vendor"

接着运行`sudo update-grub`，然后重启电脑，这时系统可以调节亮度了，但是不会保存当前值

要保存值，先看看现在的亮度值是多少，可以通过`cat /sys/class/backlight/intel_backlight/brightness` ，我的值是2442，然后打开文件 `sudo gedit /etc/rc.local`，在exit0前面加一条语句

    :::sh
    echo 2442 > /sys/class/backlight/intel_backlight/brightness

<br/>

##安装触摸板驱动器

    :::sh
    sudo add-apt-repository ppa:atareao/atareao
    sudo apt-get update
    sudo apt-get install touchpad-indicator

<br/>

##其他软件的调整

<br/>

###1.gvim


使用`sudo apt-get install gvim-gnome`安装gvim，发现gvim是7.3，顿时欣喜若狂，可是配置了我自己的插件后，发觉打开好慢，而且有错误提示，上网搜索，得到修改方案，在`~/.bashrc`文件中追加下面信息

    :::sh
    function gvim () { (/usr/bin/gvim -f "$@" &) }
    alias gvim='UBUNTU_MENUPROXY= gvim'

<br/>

###2.git

git个性化设置

    :::sh
    git config color.ui true
    git config --global alias.co checkout
    git config --global alias.cm commit
    git config --global alias.st status

顺便设置一下`git lg`

    :::java
    git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

.gitconfig配置

    :::sh
    [color]
    	ui = true
    [user]
    	name = gavin
    	email = 261878441@qq.com
    [alias]
    	lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    	co = checkout
    	cm = commit
    	st = status
    [core]
    	editor = /usr/local/bin/vim
    [diff]
    	tool = bc3

<br/>

###3.java

java设置环境变量打开文件`/etc/profile`

    :::sh
    export JAVA_HOME=/usr/share/jdk1.6.0_41
    export PATH=$JAVA_HOME/bin:$PATH
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
