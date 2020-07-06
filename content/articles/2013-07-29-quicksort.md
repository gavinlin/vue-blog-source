---
title: 快速排序
date: 2013-07-29 11:10
category: algorithm
tags: c, algorithm
---

在比较算法中，快速排序可以算是明星算法了，因为他的常系数小，使得其总体性能比其他比较算法优秀。其性能基本上可以达到$\theta (n\lg n)$ ，但在某些极端情况其时间复杂度却是$\theta ({n}^{2})$。
<!-- excerpt -->

快速排序的基本思想是遵循分而治之的思想的，其基本思路是：

1. 分：在待排序的数据中选取一个数作为中心点，大于这个数的数统统放到右边，小于这个数的统统放在左边。这样就形成了左右两个子串。

2. 治：两个子串重复1的动作

3. 合并：无


<br/>

##分

所以最重要的是分这一步，这一步的伪代码如下

    :::c
    Partition(A,p,q) // A[p,q]
        x<-A[p] i<-p for j<-p+1 to q do if A[j] <= x // <= 是小于等于的意思
                then i<-i+1
                exhange A[i]<->A[j]
        exchange A[p]<->A[i]
        return i

时间复杂度为$\theta(n)$

下面给出一个例子，看如何分。

Ex.

    :::java
    6 10 13 5 8 3 2 11  选取 pivot＝6 i=0 j=1

    :::java
    6 10 13 5 8 3 2 11  pivot=6 i=0 j=3

    i=i+1 -> i=1

    exchange A[i]<->A[j] so:

    :::java
    6 5 13 10 8 3 2 11 此时 i=1,j=3, 继续

    :::java
    6 5 13 10 8 3 2 11 i=1,j=5

    i=2

    exchange A[i]<->A[j]

    :::java
    6 5 3 10 8 13 2 11 i=2,j=5 j继续自增

    :::java
    6 5 3 10 8 13 2 11 i=2,j=6 

    i=3

    exchange A[i]<->A[j]

    :::java
    6 5 3 2 8 13 10 11 i=3,j=6 直到结束，然后

    exchange A[p]<->A[i] : A[0] <-> A[3] , 最后结果为

    :::java
    2 5 3 6 8 13 10 11 

这样以6为轴，比6小的都在左边，比6大的都在右边，分结束。

##治

治的过程其实就是递归的过程。

伪代码如下

    :::c
    Quicksort(A, p, q)
        if p<q
            then r<-Partition(A, p, q)
                Quicksort(A, p, r-1)
                Quciksort(A, r+1, q)

要使用quicksort,只要调用

    :::c
    Quicksort(A, 1, n)

要实现quicksort不难，但是要分析其平均性能就有些难度了。

有两种情况使quicksort的性能最差，就是数据全在选取数的左边，或者全在右边。这时候$T(n)=\theta({n}^{2})$

但是这两种情况出现的情况很少，而且可以使用一些方法尽量避免，例如随机选取中心轴。

最后，附上一个简单的实现demo

    :::c
    #include <stdio.h>
    #include <stdlib.h>

    void exchange(int A[], int i, int j)
    {
        int swap = A[j];
        A[j] = A[i];
        A[i] = swap;
        printf("exchange \n");
        printf("A[%d]<-->A[%d]\n",i,j);
        printf("%d<-->%d\n",A[i],A[j]);
        printf("\n");
    }

    int Partition(int A[], int start, int end)
    {
        int pivot = A[start];
        int i = start;
        int j = start + 1;
        for(; j <= end; j++)
        {
            if(A[j] <= pivot)
            {
                i++;	
                if(i != j)
                    exchange(A, i, j);
            }	
        }
        exchange(A, start, i);
        return i;
    }

    void Quicksort(int A[], int start, int end)
    {
        if(end > start)
        {
            int r = Partition(A, start, end);
            Quicksort(A, start, r - 1);
            Quicksort(A, r + 1, end);
        }
    }


    int main(int argc, const char *argv[])
    {
        int i;
        int total;
        int test[] = {5,4,6,7,9,0,1,26,3,564,8,4512,25,2,97,45};
        
        total = sizeof(test)/sizeof(int);

        Quicksort(test, 0, total - 1);

        for( i = 0; i < total; i++)
        {
            printf("%d ",test[i]);
        }
        printf("\n");
        
        return 0;
    }
