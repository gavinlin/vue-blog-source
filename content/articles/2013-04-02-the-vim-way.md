---
title: 第一章：vim思维
date: 2013-04-02
category: linux
tags: vim
Parts: vim
---

身边使用vim作为编辑器的同学比较少，非常可惜他们没用上这神器，现在我几乎所有的文本编辑都使用vim来完成了。
<!-- excerpt -->

vim的魅力在于快捷键，插件只是锦上添花，要知道vim如何神奇，还需要从具体示例入手。

<br/>

##遇见点命令

`.`在vim命令模式是重复上一个动作的意思（看帮助:h .），实际中我们可以这样用。

我们想把这个

    :::java
    Line one
    Line two 
    Line three
    Line four

变成这样

    :::java
    Line one
    	Line two 
    		Line three
    			Line four

<!-- more -->

使用的命令是

    :::java
    >G
    j.
    j.
    .

![dot]({filename}/images/forvim/dot.gif)

`>G`就是从光标到最后都增加一个tab，`j`是向下移动一格，`.`就是重复`>G`了。

<br/>

##拒绝重复

看到如下代码，如果要在最后加上`;`，你会怎样做？

    :::c
    int foo = 1;
    char * string = "hello"
    char bar = 'a'

或许你会`$` `a` `j` `ESC`然后`j` `$` `.`，直到所有修改完毕，但是有个更好的方法

    :::java
    A;
    j.
    j.

![tail]({filename}/images/forvim/tail.gif)

`A`的意思是移动到行尾然后编辑,代替了`$`+`a`，这样，只要我们移动下一行然后按下`.`，vim就会自动为我们在行的最后增加分号了。

这里列出一些二合一的命令，这些都是需要熟记和灵活运用的。

命令|相当于
:-----:|:-----:
C|c$
s|cl
S|^C
I|^i
A|$a
o|A\<CR\>
O|ko

<br/>

##退一步海阔天空

现在我们有一段代码如下

    :::js
    var foo = "method("+argument1+","+argument2+")";

我们觉得这段代码不美观，想在加号左右加一个空格，可以如何快速完成?

    :::java
    f+
    s + \<ESC\>
    ;
    .
    ;.
    ;.

![jump]({filename}/images/forvim/jump.gif)

`f+`就是快速跳到本行有`+`的字符上，`s`是删除本字符并插入，然后输入`\ +\ `，接着退出编辑模式输入`;`，这个分号的意思是跳到下一个`+`号的地方，也就是重复之前`f+`的动作，再结合`.`，很简单我们就可以把一行的+替换掉。

除了`.`和`;`，还有其他关于重复下一动作的命令，例如：

意图| 动作| 重复| 反向重复
:-----:|:-----:|:-----:|:-----:
修改|\{edit\}|.|u
搜索本行下一个字符|f\{char\}/t\{char\}|;|,
搜索本行上一个字符|F\{char\}/T\{char\}|;|,
搜索文本下一个匹配|/pattern\<CR\>|n|N
搜索文本上一个匹配|?pattern\<CR\>|n|N
执行代替|:s/target/replacement|\&|u
宏操作|qx\{changes\}q|@x|u

<br/>

##手动查找替换

首先声明vim有更好的查找替换方法，这里只是小量替换比较好用。

下面有一段文字，要把1和3行的content改为copy

    :::java
    ...We're waiting for content before the site can go live...
    ...If you are content with this, let's go ahead with it...
    ...We'll launch as soon as we have the content...

可以这么做

    :::java
    *
    cwcopy\<Esc\>
    n
    .

![find_replace]({filename}/images/forvim/find_replace.gif)

简单吧，`*`是查找的意思，`cw`是删除本光标单词再编辑，`n`就是继续查找下文

<br/>

##总结:一个按键移动，一个按键执行

上面的三个例子，我们巧妙地使用了一个按键替换复杂的移动动作，再用另一个按键来执行修改等动作，这就是vim的思维方式，简单，快捷。

