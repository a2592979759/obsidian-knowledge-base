---
tags: [排序, 算法, 嵌入式, 堆]
source: Data_Struct_Implementation/heapSort
created: 2026-08-27
---

# 堆排序（Heap Sort）

#### 用法
```
make
./heapSort
```

## 代码

源码文件：`heapSort/heapSort.c`

```c
#include <stdlib.h>
#include <stdio.h>


void heapify(int arr[], int n, int i) {
    int largest = i;
    int l = 2 * i + 1;
    int r = 2 * i + 2;

    if(l < n && arr[l] > arr[largest]) {
        largest = l;
    }

    if(r < n && arr[r] > arr[largest]) {
        largest = r;
    }

    if(largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;

        heapify(arr, n, largest);
    }
}

void heapSort(int arr[], int n) {
    for(int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }

    for(int i = n - 1; i > 0; i--) {
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        heapify(arr, i, 0);
    }
}

void printArray(int arr[], int n)
{
    for (int i = 0; i < n; ++i) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}
 
// 驱动代码
int main()
{
    int arr[] = { 12, 11, 13, 5, 6, 7 };
    int n = sizeof(arr) / sizeof(arr[0]);
 
    heapSort(arr, n);
    printArray(arr, n);
}
```

## 复杂度

时间复杂度：`heapify` 的时间复杂度为 O(log n)。把整个堆排序的时间为 O(n)，因此堆排序的总体时间复杂度为 O(n·log n)。

## 分析

1. **建最大堆**：从最后一个非叶子节点开始向前，对所有节点执行 `heapify`，把数组调整成最大堆。
2. **排序**：反复把堆顶（最大值）与末尾元素交换，堆大小减 1，再对堆顶 `heapify`，把新的最大值放到堆顶。

原地排序，空间 O(1)。

## 对比与相关文档
- [[binaryHeap]] —— 二叉堆（插入/弹出）
- [[quickSort]] —— 快速排序
- [[mergeSort]] —— 归并排序
