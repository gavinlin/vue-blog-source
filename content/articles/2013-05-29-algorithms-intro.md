---
title: 算法-又一坑
date: 2013-05-29
category: algorithm
---

对于一个软件工程师，算法重要吗？如果想混日子，应该不重要。

但是想进步，这个基础却必须打好。
<!-- excerpt -->

君不见国外的工程师面试，数据结构和算法可是大头。

我们经常使用各种API，却不知道个中原理。即使看源码，还是容易云里雾里的。

算法，无论如何也要攻克。

所以，开个坑，学算法去。

<br/>

##传说

有一个关于高斯同学的故事，与算法有关，故事是这样的：

高斯上小学的时候，老师出了一个问题，做完就可以放学，这个问题很简单：

1+2+3+4+5+6......+100=?

结果，高斯很早就放学了。因为他会高效的算法，当其他同学一个一个算的时候，他却是这样算的。

    :::java
    1  +2 +3 +4 +5.....+97+98+99+100 +
    100+99+98+97+95....+4 +3 +2 +1 = 

101+101+101+101+101....+101+101+101+101=

101*100=10100

1  +2 +3 +4 +5.....+97+98+99+100 = 10100/2 = 5050

高斯真是个聪明的孩子，别人用的时间是O(n)，他用的是O(1)。当然快了。

<br/>

##一个例子

编程的思想，除了"KISS"，还需要高效，虽然现在的计算机和手持设备都足够强大了，但是还没有到为所欲为的地步，没有高效的算法，软件寸步难行。

有一个例子，计算`斐波那契数`，这是C语言程序设计一开始就介绍的。

    :::c
    double fib1(int n)
    {
        if(n <= 0)
            return -1;
        if(n == 1)
            return 0;
        if(n == 2)
            return 1;
        return fib1(n - 1) + fib1(n - 2);
    }

算法很简单，教科书答案，但是用了递归，效率像掉进了万象深渊。为什么这么说，可以用一张图来解释。

![fib]({filename}/images/img/fib.png)

其时间复杂度为$T(n) = \frac{1}{\sqrt{5}} ({( \frac{1 + \sqrt{5}}{2})}^{n} -  {( \frac{ 1 - \sqrt{5}}{2} )}^{n})$


<br/>

不用递归可以否，答案是可以的。

    :::c
    double fib2(int n)
    {
        double result;
        double * buf = NULL;
        int loopCount = 2;
        if(n <= 0)
            return -1;
        buf = (double*)malloc(sizeof(double)*n);
        if(buf == NULL)
            return -1;
        buf[0] = 0;
        buf[1] = 1;
        while(loopCount < n)
        {
            buf[loopCount] = buf[loopCount - 1] + buf[loopCount - 2];	
            loopCount++;
        }

        result = buf[n - 1];
        free(buf);
        return result;
    }

时间复杂度为$\theta(n)$，也就是线性的。

好像有点复杂？一点也不复杂，而且高效。不信？可以测试一下。

    :::c
    int main(int argc, const char *argv[])
    {
        clock_t start,end;
        double count;
        int n;
        double total;
        if(argc != 2)
        {
            printf("error \n");
            return 0;
        }
        n = atoi(argv[1]);
        start = clock();
        printf("fib1(%d) result is %f \n",n,fib1(n));
        end = clock();
        count = (double)(end - start)/ CLOCKS_PER_SEC;
        printf("%f seconds \n",count);

        start = clock();
        printf("fib2(%d) result is %f \n",n,fib2(n));
        end = clock();
        count = (double)(end - start)/ CLOCKS_PER_SEC;
        printf("%f seconds \n",count);
        return 0;
    }

结果如下

    :::java
    fib1(43) result is 267914296.000000 
    3.140000 seconds 
    fib2(43) result is 267914296.000000 
    0.000000 seconds 

这，就是算法的魅力，over。

