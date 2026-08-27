---
tags: [数据结构, 算法, 嵌入式, 堆, 优先队列]
source: Data_Struct_Implementation/binaryHeap
created: 2026-08-27
---

# 二叉堆（binaryHeap）

在堆中，最高（或最低）优先级的元素总是存放在根节点，因此得名“堆”。堆不是有序结构，可视为**部分有序**。从图中可知，同一层级的节点之间（包括兄弟节点）没有特定的顺序关系。

由于堆是一棵**完全二叉树**，它具有尽可能小的树高——包含 N 个节点的堆，高度始终为 O(log N)。

当你需要移除最高（或最低）优先级对象时，堆是非常有用的数据结构。堆的常见用途是**实现优先队列**。

## 插入（Insert）
新元素先被追加到堆的末尾（作为数组的最后一个元素）。通过将新元素与父节点比较，并向上移动一层（与父节点交换位置）来恢复堆性质。这一过程称为“**上滤 / heapify up**”。比较会一直重复，直到父节点大于或等于正在上滤的元素。

## 弹出（Pop）
最小/最大元素位于根节点，即数组的第一个元素。我们移除根节点，并用堆的最后一个元素替换它，然后通过“**下滤 / heapify down**”恢复堆性质。与插入类似，最坏情况运行时间为 O(log n)。

## 复杂度

| 实现 | Insert | Pop | Remove | getMax/getMin |
|------|--------|-----|--------|---------------|
| 二叉堆 | O(log n) | O(log n) | O(log n) | O(1) |

## 用法
```
make
./binaryHeap
```

## 最大堆（Max Heap）

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

typedef struct max_heap {
    int curIndex;
    int capacity;
    int *data;
} MAX_HEAP, *pMAX_HEAP;

static void swap(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

#define LEFT_CHILD(i) (2*i+1)
#define RIGHT_CHILD(i) (2*i+2)
#define PARENT(i) ((i-1)/2)

pMAX_HEAP max_head_init(int size) {
    pMAX_HEAP new_heap = (pMAX_HEAP) malloc(sizeof(MAX_HEAP));

    new_heap->curIndex = 0;
    new_heap->capacity = size;
    new_heap->data = malloc(sizeof(int)*size);

    return new_heap;
}

int get_max(pMAX_HEAP heap) {
    if (heap->curIndex == 0) {
        printf("Heap is Empty!\n");
        return -1;
    }

    return heap->data[0];
}

void up_heapify(pMAX_HEAP heap, int index) {
    int parent_index = PARENT(index);

    if (index == 0 || heap->data[parent_index] >= heap->data[index])
        return;

    swap(&heap->data[parent_index], &heap->data[index]);
    up_heapify(heap, parent_index);
}

void down_heapify(pMAX_HEAP heap, int index) {
    int left_i = LEFT_CHILD(index);
    int right_i = RIGHT_CHILD(index);
    int target_index = index;

    if (left_i < heap->curIndex && heap->data[left_i] > heap->data[index])
        target_index = left_i;

    if (right_i < heap->curIndex && heap->data[right_i] > heap->data[index])
        target_index = right_i;

    if (target_index == index)
        return;

    swap(&heap->data[target_index], &heap->data[index]);
    down_heapify(heap, target_index);
}

int insert(pMAX_HEAP heap, int value) {
    if (heap->curIndex == heap->capacity) {
        printf("Heap is Full!\n");
        return -1;
    }

    heap->data[heap->curIndex] = value;
    heap->curIndex ++;

    up_heapify(heap, heap->curIndex-1);

    return 0;
}

int pop(pMAX_HEAP heap) {
    if (heap->curIndex == 0) {
        printf("Heap is Empty!\n");
        return -1;
    }

    swap(&heap->data[0], &heap->data[heap->curIndex-1]);
    heap->curIndex --;

    down_heapify(heap, 0);

    return 0;
}

int main(int argc, int **argv) {
    pMAX_HEAP max_heap = max_head_init(5);

    insert(max_heap, 1);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 2);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 3);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 4);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 5);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 6);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    pop(max_heap);
    printf("Max val: %d\n", get_max(max_heap));

    insert(max_heap, 6);
    printf("Max val: %d\n", get_max(max_heap));

    return 0;
}
```

## 优先队列（Priority Queue）

```c
#include <stdlib.h>
#include <stdio.h>
#include <limits.h>

typedef struct priorityQueue {
    uint32_t curIdx;
    uint32_t capacity;
    int *data;
} PRIORITY_QUEUE, *pPRIORITY_QUEUE;

void printPriorityQueue(pPRIORITY_QUEUE pQ) {
    for (int i = 0; i < pQ->curIdx; i++) {
        printf("%d ", pQ->data[i]);
    }
    printf("\n");
}

pPRIORITY_QUEUE createPriorityQueue (uint32_t size) {
    pPRIORITY_QUEUE newPq = (pPRIORITY_QUEUE) malloc (sizeof(PRIORITY_QUEUE));
    if(!newPq) {
        printf("ERROR: Unable to create new priority queue\n");
        exit(EXIT_FAILURE);
    }

    newPq->capacity = size;
    newPq->curIdx = 0;
    newPq->data = (int *) malloc(size * sizeof(int));
    if(!newPq->data) {
        printf("ERROR: Unable to allocate data for new priority queue\n");
        exit(EXIT_FAILURE);
    }

    return newPq;
}

int parent(int i) {
    return (i - 1) / 2;
}

// 返回左孩子下标
int left_child(int i) {
    return 2*i + 1;
}

// 返回右孩子下标
int right_child(int i) {
    return 2*i + 2;
}

void heapify(pPRIORITY_QUEUE pQ, uint32_t eleIdx){
    // 找左孩子节点
    int left = left_child(eleIdx);

    // 找右孩子节点
    int right = right_child(eleIdx);

    // 找三者中最大的
    int largest = eleIdx;

    // 检查左节点是否比当前节点大
    if (left < pQ->curIdx && pQ->data[left] > pQ->data[largest]) {
        largest = left;
    }

    // 检查右节点是否比当前节点大
    if (right < pQ->curIdx && pQ->data[right] > pQ->data[largest]) {
        largest = right;
    }

    // 将最大节点与当前节点交换
    // 并重复该过程直到当前节点大于左右子节点
    if (largest != eleIdx) {
        int temp = pQ->data[eleIdx];
        pQ->data[eleIdx] = pQ->data[largest];
        pQ->data[largest] = temp;
        heapify(pQ, largest);
    }
}

void insert(pPRIORITY_QUEUE pQ, int newEntry) {
    int newElementIdx;
    if(pQ->curIdx == pQ->capacity){
        printf("priority queue is full. Unable to insert data.\n");
        return;
    }

    pQ->data[pQ->curIdx++] = newEntry;
    newElementIdx = pQ->curIdx - 1;
    
    while(newElementIdx != 0 && pQ->data[parent(newElementIdx)] < newEntry) {
        int parentIdx = parent(newElementIdx);
        int temp = pQ->data[parentIdx];
        pQ->data[parentIdx] = newEntry;
        pQ->data[newElementIdx] = temp;
        newElementIdx = parentIdx;
    }
}

uint32_t pop(pPRIORITY_QUEUE pQ) {

    int result;
    if(pQ->curIdx == 0) {
        printf("priority queue is empty");
        return INT_MIN;
    }

    result = pQ->data[0];
    pQ->data[0] = pQ->data[pQ->curIdx - 1];
    pQ->curIdx--;

    heapify(pQ, 0);

    return result;
}

int main() {
    int temp;
    pPRIORITY_QUEUE pQ = createPriorityQueue(10);
    insert(pQ, 1);
    insert(pQ, 5);
    insert(pQ, 3);
    insert(pQ, 2);
    insert(pQ, 4);
    insert(pQ, 6);
    printPriorityQueue(pQ);

    printf("---\n");
    temp = pop(pQ);
    printf("popped %d\n", temp);
    temp = pop(pQ);
    printf("popped %d\n", temp);
    temp = pop(pQ);
    printf("popped %d\n", temp);
    temp = pop(pQ);
    printf("popped %d\n", temp);

    return 0;
}
```

## 参考
[CMU binary Heap](https://www.andrew.cmu.edu/course/15-121/lectures/Binary%20Heaps/heaps.html)

## 相关文档
- [[heapSort]] —— 基于堆的排序
- [[BST]] —— 二叉搜索树
- [[taskScheduler]] —— 调度器常用到优先队列/堆思想
