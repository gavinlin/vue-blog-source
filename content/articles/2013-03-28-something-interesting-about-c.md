---
title: 一些有趣的c题目
date: 2013-03-28 15:23
category: program language
tags: c
---

最近几天都发现了一些c语言的有趣的c题目，有些还蛮实用的。
<!-- excerpt -->

<br/>

##题目1

请问下面的程序打印的是什么？


    :::c
    #include <stdlib.h>
    #include <stdio.h>
    
    int main(int argc, const char *argv[])
    {
    	unsigned int a =6;
    	int b = -20;
    	(a+b > 6) ? printf("bigger than 6 \n") 
    		: printf("not bigger than 6 \n");
    	printf("a+b is %u\n",a+b);
    	return 0;
    }

答案是`bigger than 6`，然后a+b是`4294967295`

这个问题涉及到c语言算术运算的隐式转换，对于本例子，a+b的结果被转换为了无符号数再和6做比较，结果可想而知，关于隐式转换，这里有一篇不错的文章：[http://www.hookcn.org/2011/01/implicit-conversions-of-usual.html][1]

<br/>

##题目2

有个数组`a[100]`存放了100个数，这100个数取自1-99，且只有两个相同的数，剩下的98个数不同，写一个搜索算法找出相同的那个数的值。

答案：

    :::c
    int searchRepeatNumber(const int * a,int length){
    	int noRepeatNumber;
    	int totalNumber = 0;
    	int loopnumber;
    	if(length <= 0 || a == NULL){
    		return -1;
    	}
    
    	noRepeatNumber = (length*(length - 1)) >> 1;
    	for(loopnumber = 0; loopnumber < length; loopnumber++){
    		totalNumber += a[loopnumber];
    	}
    	return totalNumber - noRepeatNumber;
    }

我这个解答的时间复杂度是O(n)，因为有一个循环。思路很简单，计算数组里面的数的总和，减去没有重复数的总和，得到的就是那个重复数了。

<br/>

##题目3

上一题的变种：数组`a[101]`存放了101个数，其中100个数两两成对，有一个落单，找出落单的数。

答案：

    :::c
    int f(int array[], int length)
    {
        int ret = 0;
        int i = 0;
    
        for(i=0; i<length; i++)
        {
            ret = ret ^ array[i];
        }
    
        return ret;
    }

答案是用了异或运算的特性，要注意这是位运算，异或不记得的话请看下表：

第一个数|第二个数|结果
:-----:|:-----:|:-----:
1|0|1
0|0|0
1|1|0
0|1|1

<br/>

初级的就到这里了，一句话总结，基础很重要。

[1]: http://www.hookcn.org/2011/01/implicit-conversions-of-usual.html
