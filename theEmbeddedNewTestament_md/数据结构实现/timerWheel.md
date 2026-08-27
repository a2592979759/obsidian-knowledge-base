---
tags: [定时器, 时间轮, 嵌入式, 算法]
source: Data_Struct_Implementation/timerWheel
created: 2026-08-27
---

# 简单的单层时间轮（Simple One-layer Timing Wheel）

### 用法
```
make
./timer
```

### 分析

这是一个简单的单层时间轮设计：轮子包含 10 个槽位（wheel bin），每个槽位维护一个回调函数链表。当时间轮滴答（tick）发生且截止时间（deadline）到达时，这些回调函数会被触发。

### 代码

源码文件：`timerWheel/timer.c`

```c
#include<stdio.h>
#include<stdint.h>
#include<stdlib.h>
#include<time.h>
#include<unistd.h>

#define WHEEL_BIN_NUMBER 10
#define GRANULARITY 1000000

typedef void (*timeout_handler)();

typedef struct node {
    struct node *next;
    int timestamp;
    timeout_handler timeout_cb;
} Node, *pNode;

typedef struct timing_wheel {
    int cur_slot;
    int granularity;
    Node nodes[WHEEL_BIN_NUMBER];
} TWheel, *pTWheel;

pTWheel init_time_wheel(int gran) {
    pTWheel new_wheel = (pTWheel) malloc(sizeof(TWheel));
    new_wheel->granularity = gran;
    new_wheel->cur_slot = 0;
    int i;
    for (i; i < WHEEL_BIN_NUMBER; i++) {
        new_wheel->nodes[i].next = NULL;
    }
    
    return new_wheel;
}

int install_handler(pTWheel twheel, int deadline, timeout_handler new_cb) {
    int ret = 0;
    if (deadline/twheel->granularity >= WHEEL_BIN_NUMBER) {
        printf("Deadline exceed timing wheel size\n");
        return -1;
    }
    
    int index = (twheel->cur_slot + ((deadline/(twheel->granularity))))%WHEEL_BIN_NUMBER;
    pNode iterator = &(twheel->nodes[index]);
    
    while(iterator->next) {
        iterator = iterator->next;
    }
    
    pNode new_node = (pNode) malloc(sizeof(Node));
    new_node->timeout_cb = new_cb;
    new_node->timestamp = deadline;
    new_node->next = NULL;
    iterator->next = new_node;
    
    return ret;
}

void tick(pTWheel twheel) {
    int i;
    pNode iterator = &twheel->nodes[twheel->cur_slot];
    while(iterator->next) {
        pNode tmp = iterator->next;
        tmp->timeout_cb();
        printf("Callback at %d deadline triggers\n", tmp->timestamp);
        iterator->next = iterator->next->next;
        free(tmp);
    }
    twheel->cur_slot = (twheel->cur_slot+1)%WHEEL_BIN_NUMBER;
}

void print_task() {
    printf("Hi1\n");
}

void print_task2() {
    printf("Hi2\n");
}

void print_task3() {
    printf("Hi3\n");
}

int main(void) {
    int ret = 0;
    
    pTWheel new_wheel = init_time_wheel(GRANULARITY);
    timeout_handler cb = print_task;
    install_handler(new_wheel, 4*GRANULARITY, cb);
    install_handler(new_wheel, 4.25*GRANULARITY, cb);
    install_handler(new_wheel, 4.9*GRANULARITY, cb);
    
    cb = print_task2;
    install_handler(new_wheel, 8*GRANULARITY, cb);
    
    cb = print_task3;
    install_handler(new_wheel, 8*GRANULARITY, cb);

    install_handler(new_wheel, 12*GRANULARITY, cb);
    
    int count = 0;
    while(count < 15) {
        tick(new_wheel);
        usleep(GRANULARITY);
        printf("Time: %d\n", count++);
        if (count == 8) {
            install_handler(new_wheel, 4*GRANULARITY, cb);
        }
    }
    
    return 0;
}
```

## 分析

- 时间轮有 `WHEEL_BIN_NUMBER` 个槽位，`cur_slot` 指向当前槽。
- 安装定时器时，根据 `deadline / granularity` 计算出相对当前槽的偏移，并把回调节点插入对应槽位的链表尾部（长度限制为轮子槽数）。
- 每次 `tick`，对当前槽的链表逐个触发回调并释放节点，然后将 `cur_slot` 前进一格。
- **局限**：单层时间轮只能处理不超过“槽数 × 粒度”的延时；超长延时需分层时间轮。

## 相关文档
- [[timerList]] —— 定时器链表框架
- [[taskScheduler]] —— 调度器
- [[stateMachine]] —— 状态机
