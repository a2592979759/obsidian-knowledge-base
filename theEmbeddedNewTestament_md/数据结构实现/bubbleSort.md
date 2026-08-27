---
tags: [排序, 算法, 嵌入式]
source: Data_Struct_Implementation/bubbleSort
created: 2026-08-27
---

# 冒泡排序（Bubble Sort）

#### 复杂度

- 最坏与平均时间复杂度：O(n²)。最坏情况出现在数组逆序时。
- 最好情况时间复杂度：O(n)。最好情况出现在数组已有序时。
- 辅助空间：O(1)

## 代码

源码文件：`bubbleSort/bubblesort.c`

```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>

typedef void (*sortAlgorithm)(int *array, int size);

void bubblesort(int *array, int size) {
    int i, j;
    int temp;

    for (i = size; i >= 0; i--) {
        for (j = 0; j < i-1; j++) {
            if (array[j] > array[j+1]) {
                temp = array[j];
                array[j] = array[j+1];
                array[j+1] = temp;
            }
        }
    }
}

void printArray(int arr[], int size) 
{ 
    int i; 
    for (i=0; i < size; i++) 
        printf("%d ", arr[i]); 
    printf("\n"); 
} 

void tests(int *nums, int size) {
    sortAlgorithm sort_method = bubblesort;
    clock_t start, end;
    double cpu_time_used;

    start = clock();
    sort_method(nums, size);
    end = clock();

    cpu_time_used = ((double) (end - start)) / CLOCKS_PER_SEC;

    printf("==== Sorted array test results ====\n"); 
    printArray(nums, size); 
    printf("CPU time used: %f\n\n", cpu_time_used);
}

int main() {
    // test 1
    int nums[] = {10, 7, 8, 9, 1, 5}; 
    int n = sizeof(nums)/sizeof(nums[0]); 
    tests(nums, n);

    // test 2
    int nums2[] = {1, 2, 4, 6, 8, 2, 3, 4, 0, -1, 10, 7, 8, 9, 1, 5}; 
    n = sizeof(nums2)/sizeof(nums2[0]); 
    tests(nums2, n);
    
    // test 3
    int nums3[] = {}; 
    n = sizeof(nums3)/sizeof(nums3[0]); 
    tests(nums3, n);

    // test 4
    int nums4[] = {10, 9, 8, 7, 6, 5, 4, 3, 2, 1}; 
    n = sizeof(nums4)/sizeof(nums4[0]); 
    tests(nums4, n);
    
    return 0; 
}
```

## 分析

- 每次外循环把当前未排序区间内的最大元素“冒泡”到末尾。
- 相邻元素比较并交换，一趟下来最大值到位。
- 原地排序，不占用额外空间。

## 对比与相关文档
- [[insertionSort]] —— 插入排序
- [[quickSort]] —— 快速排序（实践更快）
- [[mergeSort]] —— 归并排序（稳定，O(n·log n)）
- [[heapSort]] —— 堆排序
