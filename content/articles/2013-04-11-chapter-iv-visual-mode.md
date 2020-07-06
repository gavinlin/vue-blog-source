---
title: 第四章:可视模式
date: 2013-04-11
category: linux
tags: vim
Parts: vim
---

##遇见可视模式

什么是可视模式，那是一种可以让你任意选择文本的模式。最其他编辑器中，我们可能很习惯地用鼠标选择一段文本，复制，粘贴。但是在vim中，你会看到选择文本，可以玩出很多花样。
<!-- excerpt -->

怎么进入可是模式？我们可以在普通模式使用`v`进入，然后用`h` `j` `k` `l`来移动。然后可以用`c`来删除选择的文字并进入编辑模式，或者可以`y`来复制这一段文字，或者只是`d`来删除。

除了普通的`v`，还有其他的可视快捷键，分别如下：

命令|效果
:-----:|:-----:
v|字符选择模式
V|整行选择模式
\<C-v\>|列选择模式
gv|重选上次选过的字符

<br/>

我们可以在这几种模式之间切换，只要在选择模式下直接按那种模式的快捷键就可以了。而且我们可以用`o`来切换光标的位置。

现在我们可以来实践一下，相关命令如下

    :::java
    vbb
    o
    e

![select-mode]({filename}/images/forvim/chapter-iv/select-mode.gif)

<br/>

##可以的话请使用操作符

可视模式对于点操作有短板，这个短板可以使用操作符来解决。例子如下：

假设我们需要把标签里面的小写转为大写

    :::java
    <a href="#">one</a>
    <a href="#">two</a>
    <a href="#">three</a>

我们可以使用`vit`来选择`one`这样的被标签包围的字符。详细可以使用`:h it`来看到其详细的介绍。

接着，我们可以使用`U`来把选中的小写变大写，就像下面的操作那样

    :::java
    vit
    U
    j.
    j.

![to-upper-wrong]({filename}/images/forvim/chapter-iv/to-upper-wrong.gif)

看到了吗，在大三行的`three`，执行`.`只是把三个字符变成了大写。

**使用普通模式操作符**

看来可视模式解决不了问题，这个时候我们可以使用普通模式的操作符来解决这个问题。

在普通模式，使用`gU`来把小写变成大写，结合`it`，就可以改变标签包围着的字符的大小了。

    :::java
    gUit
    j.
    j.

![to-upper-right]({filename}/images/forvim/chapter-iv/to-upper-right.gif)

<br/>

##用列模式来格式化表格

列模式是我最喜欢的模式之一，这个模式估计其他编辑器很难模仿，就让我们来看看它可以做什么神奇的事情吧。

有下面一个表格，我们想把它变得好看一些，如何做？

    :::java
    Chapter	         Page
    Normal mode        15
    Insert mode        31
    Visual mode        44

可以使用下面的命令，看好了。

    :::java
    <C-v>3j
    x...
    gv
    r|
    yyp
    Vr-

![format-table]({filename}/images/forvim/chapter-iv/format-table.gif)

相信这里所有的命令都会用，但是将它们组合起来需要熟练和灵感了。

<br/>

##列替换

没错，又是列模式，这次演示一下替换。

例如有下面的一段文本，我们把路径写错了，images应该为videos，这样的事情时有发生，这个时候我们可以使用可视模式替换这些字符。

    :::java
    li.one   a{ background-image: url('/images/sprite.png'); }
    li.two   a{ background-image: url('/images/sprite.png'); }
    li.three a{ background-image: url('/images/sprite.png'); }

命令如下

    :::java
    <C-v>
    jje
    c
    videos<Esc>

![column-replace]({filename}/images/forvim/chapter-iv/column-replace.gif)

<br/>

##在所有行尾插入

举一个我们以前做过的例子，在所有行的最后加上`;`，那时候我们是使用`A;`和点操作符来做的，现在我们有更快的方式。

    :::js
    var foo = 1
    var bar = 'a'
    var foobar = foo + bar

这次使用的命令如下

    :::java
    <C-v>jj$
    A;
    <Esc>

![add-tail]({filename}/images/forvim/chapter-iv/add-tail.gif)

神奇吗？为了达到效果我们使用了`A`，最后一定要`Esc`来使效果生效，类似的还有`I`，这个是编辑一行，所有选中的列都执行同样的编辑操作。

可视模式，或者说是选择模式就讲到这里了。
