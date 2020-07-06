---
title: 内存侧漏的那些事儿
date: 2013-10-29
category: program language
tags: c
---

在编写 `c/c++` 程序的时候需要随时提防内存泄漏的问题。而且有时候内存泄漏了却不知道是哪里的问题。这些多是对内存管理不熟悉导致的。
<!-- excerpt -->

直接开炮，从四道面试题开始

在解答之前先补了一下内存分配的知识。

一个程序通常分为三个区域存储数据。

+ 静态存储区，存储全局变量和static变量，程序退出自动释放内存。
+ 栈存储区，存储临时变量，函数结束自动释放内存。
+ 堆存储区，向系统调用 malloc 和 new 将在这里划分存储空间，需要手动释放。

第一题

    :::c
    void GetMemory(char *p) 
    { 
        p = (char *)malloc(100); 
    } 
    void Test(void)   
    { 
        char *str = NULL; 
        GetMemory(str);  
        strcpy(str, "hello world"); 
        printf(str); 
    } 
 

这题的意图是想在子函数里面分配堆空间。但是程序运行却奔溃了。根据预期效果，即使没有写 free 程序也不会奔溃的吧。除非 strcpy 的时候 str 还是NULL。

调试验证了 str 还是NULL 的问题。问题出在调用函数的值传递问题。

在 c/c++ 中 值传递的意思是分配一个栈空间来存储传入子函数的值，函数结束的时候释放其空间。

知道了值传递后我们分析一下程序

+ 首先 `*str` 指向 NULL
+ 然后执行`GetMemory(str)`，系统就开辟了一个栈 `*p` 来存储这个NULL(值传递)
+ 然后 `p` 分配到了一段堆内存
+ 退出子函数， 释放存储 p 地址的栈(内存泄漏，因为指向p的堆内存不可能释放了)。这个时候 str 还是 NULL ，因为 p 和 str 是两个地址完全不同的指针，p 分配的内存跟 str 无关。
+ 运行strcpy 直接奔溃了。

<br/>

    :::c
    char *GetMemory(void) 
    {  
        char p[] = "hello world"; 
        return p; 
    } 
    void Test(void) 
    { 
        char *str = NULL; 
        str = GetMemory();   
        printf(str); 
    } 

+ p 可视为指向栈内存 `hello world` 的指针。
+ 返回 p 但是其栈内存已被回收，会输出不确定的内容。

<br/>

    :::c
    Void GetMemory2(char **p, int num) 
    { 
        *p = (char *)malloc(num); 
    }  
    void Test(void) 
    { 
        char *str = NULL; 
        GetMemory(&str, 100); 
        strcpy(str, "hello");   
        printf(str);  
    }  

比起第一题好像是改善了，能打印 hello 出来，但是最后忘记 free 了。

<br/>

    :::c
    void Test(void) 
    { 
        char *str = (char *) malloc(100);
        strcpy(str, “ hello ” ); 
        free(str);      
        if(str != NULL) 
        { 
            strcpy(str, " world " );  
            printf(str); 
        } 
    }

在 `free(str)` 后没有运行 `str ＝ NULL` ，所以会执行 `strcpy(str, "world")` ，结果是奔溃了。

<br/>

最后来个开源项目的例子。

    :::c
    char *get_prefsdir(void)
    {
            static char prefs_path [PATH_MAX];
            static int prefs_path_init = 0;
            char *homedir;

            if (prefs_path_init) {
                    return prefs_path;
            }
            homedir = get_homedir();
            snprintf(prefs_path, sizeof (prefs_path), "%s/.alsaplayer", homedir);
            prefs_path_init = 1;
            return prefs_path;
    }

+ 程序在静态存储区开辟了大小为 `PATH_MAX` 的存储空间
+ 经过一系列操作后把该静态存储区的地址回传。由于字符串存储在静态存储区，除非程序退出，内存不会被回收。
+ 还有就是静态存储区默认清零，数组不需要清零的步骤。不需要担心释放的问题。

