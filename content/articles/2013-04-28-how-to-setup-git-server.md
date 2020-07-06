---
title: 如何在局域网搭建git服务器
date: 2013-04-28
category: linux
tags: ubuntu
---

在创业公司，什么东西都要自己搞定，除了开发android系统和研究内核，还要自己来搭建一个代码管理服务器。
<!-- excerpt -->

其实搭建一个服务器非常简单，尤其只需要在局域网内搭建，网上已经提供好了我们需要的一切。

<br/>

##服务器的搭建

首先我们需要一个装有linux系统的计算机充当服务器，我这里是一台ubuntu12.04的系统。里面默认安装了perl和bash作为shell。

然后需要安装git-core,openssh-server等必备软件，在ubuntu 安装这些软件比较方便。

    :::sh
    sudo apt-get install git-core openssh-server

创建一个叫做git的用户。创建步骤和可能出现的问题可以参见这里:[http://lingavin.com/blog/2013/04/27/add-new-user/][1]

<br/>

##gitolite3.0

安装环境后，就可以使用gitlote来搭建一个服务器了。首先是下载gitolite源码。网址是:[https://github.com/sitaramc/gitolite][2]

安装这个软件，首先要保证`$HOME/bin`文件夹存在，然后准备好客户机的`xxx.pub`文件。

这里会涉及到一个问题，什么是`xxx.pub`和为什么需要用这个文件。`xxx.pub`是客户端用sshkeygen生成的公钥。至于为什么需要初始化的时候提供这个文件，那是因为初始化后，这个提供公钥的客户机将成为gitolite的管理员，在客户机里管理各个版本库，所以需要在初始化的时候提供。

<p class="info">版本库的管理不会在服务器上直接操作，而是在客户端，通过管理一个特殊的仓库来操作。</p>

具体步骤就是:

    :::sh
    git cloen git://github.com/sitaramc/gitolite

    #下面这一步其实是在/home/git/bin做了一个软连接
    gitolite/install -ln
    
    export PATH=/home/git/bin:$PATH
    
    gitolite setup -pk xxx.pub

完成了这几步，服务器的设置就算完成了。

<br/>

##客户端

客户端需要保证的是能够ping通服务器端，然后就是下载gitolite-admin这个仓库。命令如下:

    :::sh
    #请把host_ip替换为服务器的真实ip
    git clone git@host_ip:gitloite-admin.git

可以发现clone下来的版本库里面有两个文件夹，分别是`conf` `keydir`。

现在我们就通过添加一个开发者wang和添加一个仓库android4_0_3.git来演示如何操作这两个文件夹。

<br/>

###增加一个开发者

首先需要wang的公钥，不会生成公钥的话这里有个参考：[https://help.github.com/articles/generating-ssh-keys][4]

把wang的公钥改名为wang.pub放到keydir文件中，然后`git add` `git commit` `git push` 收工。

<br/>

##增加一个仓库

增加仓库需要到conf目录下，修改gitolite.conf

    :::java
    repo android4_0_3
    	RW+		=	xxx
    	RW		=	wang
    	R		=	@all

加上上面内容，同样是`git add` `git commit` `git push` 就可创建一个名叫android4_0_3的仓库了。

其中这个版本库赋予了不同人不同的权限。例如xxx用户有读写权限和删除等终极权限。而wang有读写权限和创建新分支的权限，但不可以删除远程版本。其他所有人则有读的权限，也就是只要你知道仓库，就可以clone下来。

<br/>

##开发者可以做的事情

你可以通过`ssh git@host_ip info`来查看你可访问的版本库以及其地址。

<br/>

##最后

上面这些信息都是通过[http://gitolite.com/gitolite/master-toc.html][3]来获得的，其中还有很多内容和细节大家可以参考。

[1]:http://lingavin.com/blog/2013/04/27/add-new-user/
[2]:https://github.com/sitaramc/gitolite
[3]:http://gitolite.com/gitolite/master-toc.html
[4]:https://help.github.com/articles/generating-ssh-keys
