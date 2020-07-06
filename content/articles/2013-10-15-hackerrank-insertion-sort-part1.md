---
title: 'HackerRank: Insertion Sort Part1'
date: 2013-10-15
category: algorithm
tags: c
---

基础水题，插入排序第一部分

<!-- excerpt -->

```script
Input Format
There will be two lines of input:

s - the size of the array
ar - the sorted array of integers
Output Format
On each line, output the entire array every time an item is shifted in it.

Constraints
1<=s<=1000 
-10000<=x<= 10000, x ∈ ar

Sample Input

5
2 4 6 8 3
Sample Output

2 4 6 8 8 
2 4 6 6 8 
2 4 4 6 8 
2 3 4 6 8 
Explanation

3 is removed from the end of the array.
In the 1st line 8 > 3, 8 is shifted one cell right. 
In the 2nd line 6 > 3, 6 is shifted one cell right. 
In the 3rd line 4 > 3, 4 is shifted one cell right. 
In the 4th line 2 < 3, 3 is placed at position 2.

Task

Complete the method insertionSort which takes in 1 parameter:

ar - an array with the value V in the right-most cell.
```

本节讲解如何移动元素并插入元素。

例如，2 4 6 8 都排好顺序了，但是 3 还没有排好。

*先把 3 从数组中抽出，以前一个元素 8 比较，发现少于 8 ，所以 8 后移。

*再和 6 比较，大于 3 ，后移。

*和 4 比较 ，大于 3 ，后移

*和 2 比较 ，小于 3 ， 所以 3 就插入 2 后的位置，也就是原来 4 的位置，到此排序结束。

题目要求答应排序的过程，照做就可以了。

 ```c
    #include <stdio.h>
    #include <string.h>
    #include <math.h>
    #include <stdlib.h>
    #include <assert.h>

    void printAr(int ar_size, int* ar){
        int i;
        for(i = 0;i < ar_size; i++){
            printf("%d",ar[i]);
            printf(" ");
        }
        printf("\n");
    }

    /* Head ends here */
    void insertionSort(int ar_size, int *  ar) {
        int i,j,k;
        int value;
        i = ar_size - 1;
        for(;i >= 0; i--){
            value = ar[i];
            j = i - 1;

            while(j >= 0 && ar[j] > value){
                ar[j + 1] = ar[j];
                j--;
                printAr(ar_size, ar);
            }
            ar[j + 1] = value;
        }
        printAr(ar_size, ar);
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
```