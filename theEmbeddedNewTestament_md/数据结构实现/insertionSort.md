---
tags: [排序, 算法, 嵌入式]
source: Data_Struct_Implementation/insertionSort
created: 2026-08-27
---

# 插入排序（Insertion Sort）

### 复杂度

- 辅助空间：O(1)
- 最好情况：O(n)（已排好序）
- 最坏情况：O(n²)

## 代码（数组实现）

源码文件：`insertionSort/insertionSort.c`

```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>

typedef void (*sortAlgorithm)(int *array, int size);

void insertionSort(int *array, int size) {
    int i, j;
    int temp;

    for (i = 0; i < size; i++) {
        temp = array[i];
        for (j = i; j >= 1; j--) {
            if (array[j-1] <= temp) {
                break;
            } 
            array[j] = array[j-1];
        }
        array[j] = temp;
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
    sortAlgorithm sort_method = insertionSort;
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

## 代码（链表实现）

[Leetcode 147](https://leetcode.com/problems/insertion-sort-list/)

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */

struct ListNode* insertionSortList(struct ListNode* head){
    struct ListNode *dummy = (struct ListNode *) calloc (1, sizeof(struct ListNode));
    
    while(head) {
        struct ListNode *nextElement = head->next;
        struct ListNode *iter = dummy;
        
        while(iter->next != NULL && iter->next->val < head->val) {
            iter = iter->next;
        }
        
        head->next = iter->next;
        iter->next = head;
        head = nextElement;
    }
    
    struct ListNode *result = dummy->next;
    free(dummy);
    
    return result;
}
```

## 分析

- 数组版：每次取一个未排序元素 `temp`，把它插入到前面已排序部分的正确位置，通过后移腾出空位。
- 链表版：构造一个哑元头 `dummy`，遍历链表，把每个节点插入到 `dummy` 链表中合适的位置。
- 对“近似有序”的数据表现很好（最好 O(n)）。

## 对比与相关文档
- [[bubbleSort]] —— 冒泡排序
- [[quickSort]] —— 快速排序
- [[mergeSort]] —— 归并排序
