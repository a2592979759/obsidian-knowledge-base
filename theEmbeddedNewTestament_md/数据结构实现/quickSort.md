---
tags: [排序, 算法, 嵌入式]
source: Data_Struct_Implementation/quickSort
created: 2026-08-27
---

# 快速排序（Quick Sort）

### 复杂度

- 空间：O(n)
- 最好情况：O(n·log n)
- 最坏情况：所有元素相等，O(n²)
- 平均：O(n·log n)

快速排序的时间一般可写为：
`T(n) = T(k) + T(n-k-1) + θ(n)`

前两项是两次递归调用，最后一项是分区（partition）过程。k 是小于枢轴（pivot）的元素个数。

**最坏情形**：分区过程总是选取最大或最小元素作为枢轴。若采用上述“总是取最后一个元素为枢轴”的策略，最坏情况出现在数组已按升序或降序排列时，递推为：
`T(n) = T(0) + T(n-1) + θ(n)`，等价于 `T(n) = T(n-1) + θ(n)`，解为 θ(n²)。

**最好情形**：分区过程总是选取中间元素作为枢轴，递推为：
`T(n) = 2T(n/2) + θ(n)`，解为 θ(n·log n)，可由主定理第 2 种情形求得。

**平均情形**：需要考察所有可能的排列，不易直接计算。可考虑分区把 O(n/9) 个元素放到一边、O(9n/10) 放到另一边的情形，递推为：
`T(n) = T(n/9) + T(9n/10) + θ(n)`，解同样为 O(n·log n)。

尽管快速排序的最坏时间复杂度 O(n²) 高于归并排序与堆排序，但**在实践中它更快**，因为它的内层循环在多数架构上都能高效实现，且对多数真实数据表现良好。可以通过改变枢轴的选择方式（如随机选枢轴）来使最坏情形几乎不出现。然而当数据量巨大且存于外部存储时，一般认为归并排序更优。

## 为什么对数组排序更偏爱快速排序而非归并排序？
快速排序的一般形式是**原地排序**（不需要额外存储），而归并排序需要 O(N) 额外存储（N 为数组长度），这可能相当昂贵。为归并排序分配与释放额外空间会增加运行时间。比较平均复杂度，两者都是 O(N·log N)，但常数不同。对数组而言，归并排序因使用额外 O(N) 存储而落败。

## 为什么对链表排序更偏爱归并排序而非快速排序？
链表情形不同，主要源于数组与链表内存分配的差异。与数组不同，链表的节点在内存中不一定相邻。链表可以在中间以 O(1) 额外空间、O(1) 时间插入元素，因此归并排序的合并操作在链表上无需额外空间即可实现。

## 代码

源码文件：`quickSort/quicksort.c`

```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#include <string.h>

typedef void (*sortAlgorithm)(int *array, int size);

static void printArray(int arr[], int size) 
{ 
    int i; 
    for (i=0; i < size; i++) 
        printf("%d ", arr[i]); 
    printf("\n"); 
} 

void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

static void sort_helper(int *array, int st, int end) {
    if (st >= end)
        return;

    int left = st;
    int right = end;

    // 若选择中间元素作为枢轴，则结束时无需再交换枢轴位置
    int pivot = array[left + (right-left)/2];

    while(left < right) {
        while(left < right && array[left] < pivot)
            left ++;

        while(left < right && array[right] > pivot)
            right --;

        if (left < right) {
            swap(&array[left++], &array[right--]);
        }
    }

    sort_helper(array, st, left-1);
    sort_helper(array, right+1, end);
}

void quicksort(int *array, int size) {
    sort_helper(array, 0, size-1);
}

void tests(int *nums, int size) {
    sortAlgorithm sort_method = quicksort;
    clock_t start, end;
    double cpu_time_used;

    printf("==== Sorted array test results ====\n");
    printf("Original:\n");
    printArray(nums, size); 

    start = clock();
    sort_method(nums, size);
    end = clock();

    cpu_time_used = ((double) (end - start)) / CLOCKS_PER_SEC;

    printf("Sorted:\n");
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

    // test 5
    int nums5[] = {1, 2, -4, 6, -8, 2, 3, -4, 0, -1, 10, -1, 5}; 
    n = sizeof(nums5)/sizeof(nums5[0]); 
    tests(nums5, n);
    
    return 0; 
}
```

## 分析
- 本实现选择**中间元素**为枢轴，用左右双指针向中间逼近并交换，从而避免最坏情形下（有序数组）退化为 O(n²)。
- 原地排序，不占用额外大空间。

## 对比与相关文档
- [[mergeSort]] —— 归并排序
- [[heapSort]] —— 堆排序
- [[bubbleSort]] —— 冒泡排序
- [[insertionSort]] —— 插入排序
