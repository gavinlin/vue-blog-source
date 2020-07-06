---
title: 'Hacker Rank: Pairs'
date: 2013-10-11
category: algorithm
tags: c
---

刚注册上 Hacker Rank ，出现了一条练手的题目，竟然不是著名的 Hello World ，而是这题 Pairs 。
<!-- excerpt -->

[连接][1]

题目是这样的

>Given N integers [N<=10^5], count the total pairs of integers that have a difference of K. [K>0 and K<1e9]. Each of the N integers will be greater than 0 and at least K away from 2^31-1 (Everything can be done with 32 bit integers).
>
>Input Format
>
>1st line contains N & K (integers).
>2nd line contains N numbers of the set. All the N numbers are assured to be distinct.
>
>Output Format
>
>One integer saying the number of pairs of numbers that have a diff K.
>
>Sample Input #00:
>
>5 2  
>1 5 3 4 2  
>Sample Output #00:
>
>3

题目的要求是找到数组里面间隔相等的数有多少对。输入有两行，第一行是数组的大小和间隔数，第二行是数组。

例如 5个数中，找出间隔为2的数对。数也数得出是3对。

##第一感觉

看到题目第一感觉，想当然的是遍历数组中所有数和其他数的相差数，符合条件的话结果加一。

但是，不用程序实现也知道这样做的效率非常慢，可以简单的认为需要的时间是指数级的。$$\theta (n^2)$$

##优化

既然第一感觉不行，就找找冗余，压缩运行时间。如何避免大量重复比较时间？

首先想到的是排序，可以先用 quicksort 来对数组排序，预期的时间是 $$\theta(n\lg n)$$

再对比每个数与它后面数的差距，只要差距大于要求的间隔数就说明它没有相应的数组成对，放弃循环。如此可以省掉大量的重复比较动作。最后通过测试

    :::c
    #include <stdio.h>
    #include <string.h>
    #include <math.h>
    #include <stdlib.h>

    void exchange(int A[], int i, int j)
    {
        int swap = A[j];
        A[j] = A[i];
        A[i] = swap;
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

    int main() {

        /* Enter your code here. Read input from STDIN. Print output to STDOUT */    
        int N ,K,i=0,j=1,lastJ=1;
        int result = 0;
        char* buf;
        char* token;
        int *set;
        int diff;
        scanf("%d %d \n",&N,&K);
        buf = malloc(13*N);
        set = malloc(sizeof(int)*N);
        
        gets(buf);
        token = strtok(buf," ");
        
        while(token != NULL)
        {
            sscanf(token, "%d", &set[i]);
            token = strtok(NULL, " ");
            i++;
        }
        
        Quicksort(set,0, N - 1);
        
        for(i = 0; i < N; i++)
        {
            j = i +1;
            if(set[lastJ]-set[i]<K)
                j = lastJ+1;
            for(; j < N; j++)
            {
                diff = set[j] - set[i];
                if(diff > K)
                    break;
                lastJ = j;
                if(diff == K)
                {
                    result++;
                    break;
                }
            }
        }
        printf("%d", result);
        
        return 0;
    }

最后，想在本地方便读入文件数据调试，可以使用以下函数

    :::c
    freopen("slyar.in", "r", stdin);

    //原样代码

    fclose(stdin);

[1]: https://www.hackerrank.com/challenges/pairs
