---
title: Android Studio For Experts
date: 2015-11-26
category: android
tags: android
---

## 自动完成 auto completion
<!-- excerpt -->

ctrl + space

选择后按 tab 可以覆盖原理的方法。

shift + ctrl + space

更加聪明的补全方式

## selection

extend selection

alt + ↑

## intention 

alt + enter 

对于构造函数，可以自动生成private 变量并赋值，增加变量可自动添加。

在 instanceCheck 中可以自动生成  cast 代码

suppress

## templates

fori

list.for

自动生成循环

logi logd loge

打印

logt

生成 TAG

logm 

方法开头用

logr

方法结尾用

command + n 

弹出Generate

## 布局

tools:showIn="@layout/fragment_sign_in"

可以包含另一个布局来预览

tools 关键字可以用在不同的参数上，用上后该参数只会在预览中有效。

public.xml 可以暴露想暴露的属性给 ide

格式为
```
<resources>
    <public name="ccl_app_name" type="string"/>
</resources>
```

ctrl + tab

Switcher

## How to use debugger

To find an action

shift + command + A

Analyze Stacktrace

可以分析错误
