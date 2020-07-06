---
title: Android MVC Part1 Intro
date: 2013-11-12
category: android
tags: mvc
Parts: mvc
---

##about

一直想寻找 android 关于基于 mvc 开发的教程， 但是国内的网站都讲得很空泛，最后在一个外国的 blog 里看到一系列的文章，介绍得很详细和具体，而且是基于项目的，所以转过来了。
<!-- excerpt -->

这系列的文章不仅简述 MVC，还涉及到了状态模式的运用，如何用 Data Access Objects 来持久化数据，使用 web service 的命令来传递数据。这系列文章假设你已经了解用这些技术的好处，所以不会详细讲解为什么要这么用。

你可以通过点击[这里][1]查看原blog。

[1]: http://www.therealjoshua.com/2011/11/android-architecture-part-1-intro/

##包管理

第一章先来建立工程并取好包名。好的包管理可以使明确你的类的大致作用，对编程有很大的帮助，试想下如果把所有 class 放到一个包中，当项目变大后，你想修改其中一个 class 将会是个噩梦。这里提供一个比较好的包的分类。

+ activities
+ controllers
+ daos
+ lists
+ models
+ utils
+ vos
+ widgets

注意到所有的包名都是复数，因为里面不止一个 class 。上面的包名如果项目用不到可以删除。由包的名字我们就可以知道里面的类大致的功能，这就是好处。

+ activities – 很明显，这里会放入所有的 activity
+ controllers – View 的大脑就放在这里面了
+ daos – 这里存放持久化逻辑
+ lists – 这里存放所有 list adapter
+ models – 这里存放 View 绑定的 Model
+ utils – 存放静态类和工具类
+ vos – 存放 Value Objects 也就是 POJOs 又或者 Data Transfer Objects
+ widgets – 借鉴于android 存放定制化 widget

part1 就到这里，源代码可以在这里下载 [https://github.com/gavinlin/tapcounter](https://github.com/gavinlin/tapcounter)
