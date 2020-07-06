---
title: 第二章:vim安装与普通模式
date: 2013-04-04 04:43
category: linux
tags: vim
Parts: vim
---

<br/>

##vim的安装

强烈推荐使用gvim而不是使用vim，因为gvim提供了比vim更丰富的功能，例如颜色和光标的表现。
<!-- excerpt -->

<br/>

###安装依赖库

    :::java
    sudo apt-get build-dep vim
    sudo apt-get install mercurial

<br/>

###下载vim 源程序

    :::java
    hg clone https://vim.googlecode.com/hg gvim
    cd gvim 
    hg tags
    hg update [tags]

<!-- more -->

<br/>

###配置与安装vim

    :::java
    ./configure \  
        --enable-multibyte \  
        --enable-perlinterp=yes \  
        --enable-pythoninterp=yes \  
        --enable-tclinterp \  
        --enable-rubyinterp \  
        --enable-cscope \  
        --enable-sniff \  
        --with-features=huge \  
        --enable-gui=gnome2 \
        --with-compiledby=gavin
    
    make && sudo make install

<br/>

###vim 插件和配置

可以查看我的配置，地址是:[https://github.com/gavinlin/vim-conf][1]

<br/>

##普通模式

当我们使用命令`gvim [file]`，来打开文件时，gvim是处于普通模式的。gvim基本上有三种常用的模式，普通模式，编辑模式和命令模式。

其他很多编辑器默认只有一种模式，就是编辑模式，可能很多人会奇怪，为什么vim要分这么多种模式，而且默认是普通模式，而不是常识中的编辑模式。其实因为普通模式实在太重要了，很多操作都可以在普通模式中完成，这也是vim不同于其他编辑器的魅力所在，越深入学习，我就越认同了这点。

<br/>

###undos

vim里面的undo，也就是撤销操作，可以通过在普通模式键入`u`来执行。如何从其他模式进入普通模式，我们只要按下`Esc`就可以了，通常是你键盘的左上角。`u`的强大之处在于，你对文本做的所有修改，都可以通过这个命令来撤销。这是我们经常会用到的命令。虽然人生不可以重来，但代码可以。

<br/>

###组成可重复的变化

有一段英文文本，我们想删除最后一个词，光标停留在最后一个词的最后位置，如何做才能做到高效率的删除单词动作？

    :::java
    例如我们要删除以下句子的最后一个单词。
    The end is nigh

有三种方法，分别是

    :::java
    db x
    b dw
    daw

这三种方法都只是打了三个按键就完成了操作，但是这其中有效率的问题，怎样比较？

对于第一种情况，`db`是删除了本光标之前的字母，直到空格，最后还要键入x来删除光标上的字。但我想重复动作的时候，`.`＝x。

对于第二种情况，先使用`b`移动到单词头，再使用`dw`删除整个单词，`.`=dw

对于第三种情况，daw中的`aw`是a word的意思，所以整个命令就是删除一个单词的意思，`.`=daw

显然，当我们想重复删除一个单词的时候，第三种方法是最高的，因为我们只要按下`.`就可以重复动作。

<br/>

###数数

vim下面的数数也是一大亮点，我们使用`ctrl-a`和`ctrl-x`来数数，什么意思呢？还是通过案例来解答。

下面有两行css代码，我们需要新增一行，并命名为.news 坐标改为－180px

    :::css
    .blog, .news { background-image: url(/sprite.png); }
    .blog { background-position: 0px 0px }

普通的做法是先`yyp`第二行，把`blog`改为`news`，然后`f0`跳转到第一个0出现的位置再用`i`，编辑模式来修改。

下面让我们来看看加速的方法。

    :::java
    yyp
    cW.news<Esc>
    180<ctrl-x>

![count]({filename}/images/forvim/count.gif)

我们使用`180<C-x>`，来完成这个动作，其中`<C-x>`直接跳到本行数字的位置并进行减操作，前面的180,就是减180次，所以最后结果是－180。

<br/>

###可以重复的话不要数数

有时候我们可以通过`2dw`来删除俩个词，也可以使用`dw.`来完成，这里更推荐后者，因为当只想删除一个词的时候，前者需要做的动作更多，而后者保证了撤销的粒度。但是这个不是绝对的，要根据实际问题来处理。

<br/>

###联手抗敌

大部分vim的操作和移动命令可以组合来使用。这里我们可以看到他们是如何组合的。

`d{动作}`的组合很常用，例如`dl`删除单一字母，当然，我们可以使用`x`来完成。`daw`来删除一个单词，`dap`来删除一个段落，同样的还有`c{动作}`和`y{动作}`。

`gU`命令可以使单词变为大写，例如`gUaw`，`gUap`，又或者`gUU`

下面列出一些操作命令

触发键|影响
:-----:|:-----:
c|删除再编辑
d|删除
y|复制
g~|大小写切换
gu|转为小写
gU|转为大写
\>|向右tab
\<|向左tab
=|对齐
!|通过外部程序过滤行

<br/>

[1]: https://github.com/gavinlin/vim-conf
 
