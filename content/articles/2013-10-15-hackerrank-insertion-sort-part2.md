---
title: 'HackerRank: Insertion Sort Part2'
date: 2013-10-15
category: algorithm
tags: [c]
---

第二部分要求我们使用插入排序来排列数组，并把结果打印出来。
<!-- excerpt -->

>Sample Input
>
>6
>1 4 3 5 6 2
>
>Sample Output
>
>1 4 3 5 6 2 
>
>1 3 4 5 6 2 
>
>1 3 4 5 6 2 
>
>1 3 4 5 6 2 
>
>1 2 3 4 5 6 

例子中是从前往后遍历元素并排序。插入排序的效率并不高，时间复杂度 $$\theta(n^2)$$

    :::c
    #include <stdio.h>
    #include <string.h>
    #include <math.h>
    #include <stdlib.h>
    #include <assert.h>

    void printAr(int ar_size, int* ar){
        int i;
        for(i = 0; i < ar_size;i++){
            printf("%d",ar[i]);
            printf(" ");
        }
        printf("\n");
    }

    /* Head ends here */
    void insertionSort(int ar_size, int *  ar) {
        int i,j,value;
        
        for(i = 1; i < ar_size; i++){
            j = i - 1;
            value = ar[i];
            while(j >= 0 && ar[j] > value){
                ar[j + 1] = ar[j];
                j--;
            }
            if(j != i - 1 )
                ar[j + 1] = value;
            printAr(ar_size, ar);
        }

    }

    /* Tail starts here */
    int main(void) {
       
       int _ar_size;
    scanf("%d", &_ar_size);
    int _ar[_ar_size], _ar_i;
    for(_ar_i = 0; _ar_i < _ar_size; _ar_i++) { 
       scanf("%d", &_ar[_ar_i]); 
    }

    insertionSort(_ar_size, _ar);
       
       return 0;
    }
