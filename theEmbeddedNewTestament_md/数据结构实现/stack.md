---
tags: [数据结构, 算法, 嵌入式, 栈]
source: Data_Struct_Implementation/stack
created: 2026-08-27
---

# 栈（数组实现）

#### 用法
```
make
./stack
```

## 分析

栈用**数组**实现（给定容量）。与队列相反，用数组实现栈只需用一块连续内存存放所有数据，并记录“最后元素索引”。因为栈只需要弹出最近压入的元素（LIFO，后进先出），实现时只需将最后元素索引减 1 即可。

需要注意的是，数组无法动态扩容，因此这种方式总会**预先分配一块固定大小的内存**。但其优点在于栈操作极快——我们只是操纵数组下标而已。

## 代码

```c
#include <stdlib.h>
#include <stdio.h>
#include <limits.h>

typedef struct stack {
    int topIdx;
    int capacity;
    int *data;
} STACK, *pSTACK;  // 同时定义了指向该结构体的指针，这是很好的写法

pSTACK createStack(int capacity) {
    pSTACK newStack = (pSTACK) malloc (sizeof(STACK));
    if(newStack == NULL) {
        printf("ERROR: cannot allocate memeory for stack\n");
        exit(EXIT_FAILURE);
    }
    newStack->data = (int *) malloc (capacity * sizeof(int));
    if(newStack->data == NULL) {
        printf("ERROR: cannot allocate memeory for stack data\n");
        exit(EXIT_FAILURE);
    }

    newStack->topIdx = -1;
    newStack->capacity = capacity;
    return newStack;
}

int isFull(pSTACK curStack) {
    return curStack->topIdx >= curStack->capacity - 1;
}

int isEmpty(pSTACK curStack) {
    return curStack->topIdx == -1;
}

void push(pSTACK curStack, int data) {
    if(isFull(curStack)){
        return;
    }
    curStack->data[++(curStack->topIdx)] = data;
    printf("%d pushed to stack\n", data);
}

int pop(pSTACK curStack) {
    if (isEmpty(curStack)){
        printf("stack is empty\n");
        return INT_MIN;
    }

    return curStack->data[(curStack->topIdx)--];
}

int top(pSTACK curStack) {
    if (isEmpty(curStack)){
        printf("stack is empty\n");
        return INT_MIN;
    }
    return curStack->data[curStack->topIdx];
}

int main() {
    pSTACK myStack = createStack(10);
    push(myStack, 10);
    push(myStack, 20);
    push(myStack, 30);

    printf("cur top %d\n", top(myStack));
    printf("popped element %d\n", pop(myStack));
    printf("popped element %d\n", pop(myStack));
    printf("popped element %d\n", pop(myStack));

    return 0;
}
```

## 复杂度
- push / pop / top / isFull / isEmpty：O(1)
- 空间：O(capacity)，预先分配固定大小数组

## 相关文档
- [[queue]] —— 队列用链表实现，与栈对比
- [[circularRingBuffer]] —— 另一种常用的线性数据结构
- 参考：https://www.geeksforgeeks.org/stack-data-structure-introduction-program/
