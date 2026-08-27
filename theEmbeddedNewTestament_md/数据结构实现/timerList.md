---
tags: [定时器, 链表, 嵌入式, 算法]
source: Data_Struct_Implementation/timerList
created: 2026-08-27
---

# 定时器链表框架（Timer Framework）

> 一个非常简化的定时器框架，作者 amallory@qnx.com。

源码文件：`timerList/timer_framework.c`

## 代码

```c
/* Very simplistic timer framework by amallory@qnx.com */
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <time.h> 
#include <sys/time.h>
#include <inttypes.h>
#include <sys/queue.h>

#define NUM_TIMERS 10
#define MAX_RANDOM_TIME_MS  20000

enum timer_callback_retval {
    CB_RETURN_NORMAL = 0,
    CB_RETURN_FREE_TIMER,
    CB_RETURN_INVALID,
};

enum timer_type {
    TT_RELATIVE = 0,
    TT_ABSOLUTE,
    TT_INVALID,
};

/* 定义定时器链表类型 */
TAILQ_HEAD(timer_list, timer_node);

/* 主滴答时钟/计数 */
uint64_t tick_cnt = 0;

/*
定时器数据结构：
- 链表节点
- 单调的触发时间（存为绝对时间）
- 定时器到期时运行的用户回调函数
- 传给用户回调的注册数据指针
*/
struct timer_node {
    TAILQ_ENTRY(timer_node) entries;
    uint64_t fire;
    int (*cb)(void* user_data);
    void *user_data;
};

/* 全局定时器链表 */
struct timer_list active_timers;
struct timer_list free_timers;
struct timer_node *timer_memory;

#ifdef DEBUG
/* 打印指定定时器链表的内容 */
void print_list(struct timer_list *list) {
    struct timer_node *np;
    TAILQ_FOREACH(np, list, entries) {
        printf("timer fire %llu\n", np->fire);
    }
}
#endif

/* 把一个定时器放到空闲链表 */
static void free_timer(struct timer_node *timer) {
    TAILQ_INSERT_HEAD(&free_timers, timer, entries);
}

/* 从空闲链表取出一个可用定时器 */
static struct timer_node* alloc_timer(void) {
    struct timer_node *np;
    if(TAILQ_EMPTY(&free_timers)) return NULL;

    np=TAILQ_FIRST(&free_timers);
    TAILQ_REMOVE(&free_timers, np, entries);
    return np;
}

/* 把定时器放到活动定时器队列 */
static void arm_timer(struct timer_node* timer) {
    struct timer_node *np;

    if(TAILQ_EMPTY(&active_timers)) {
        TAILQ_INSERT_HEAD(&active_timers, timer, entries);
    } else {
        for(np=TAILQ_FIRST(&active_timers) ; np ; np=TAILQ_NEXT(np, entries)) {
            if(timer->fire < np->fire) {
                TAILQ_INSERT_BEFORE(np, timer, entries);
                return;
            }
        }
        TAILQ_INSERT_TAIL(&active_timers, timer, entries);
    }
}

/* 从活动定时器队列移除定时器 */
static void disarm_timer(struct timer_node* timer) {
    TAILQ_REMOVE(&active_timers, timer, entries);
return;
}

/* 设置定时器属性：相对/绝对触发时间、触发时间、回调、传给回调的用户数据 */
static int set_timer(struct timer_node* timer, enum timer_type tt, uint64_t fire, int (*cb)(void*), void* user_data) {
    switch(tt) {
        case TT_RELATIVE:
            fire+=tick_cnt;
            break;
        case TT_ABSOLUTE:
            break; /* 不做处理 */
        case TT_INVALID:
        default:
            return -1;
    }
    timer->fire = fire;
    timer->cb = cb;
    timer->user_data = user_data;
    return 0;
}

/* 初始化定时器子系统 */
static void init_timers(void) {
    unsigned i;

    TAILQ_INIT(&active_timers);
    TAILQ_INIT(&free_timers);

    /*  我们预先分配内存并使用固定定时器池大小，以保持简单，
       避免用大量错误检查来应对糟糕的内存状况。
       这样要么成功，要么完蛋。 */
    if((timer_memory = malloc(sizeof(struct timer_node)*NUM_TIMERS)) == NULL) {
        perror("Fatal! Can't allocate our block of timers!");
        exit(EXIT_FAILURE);
    }

    /* 填充空闲定时器链表 */
    for(i=0 ; i < NUM_TIMERS; i++) {
        free_timer(&timer_memory[i]);
    }
}

/* 每个时钟滴答运行的时钟处理例程 */
static void clock_tick(int signo) {
    struct timer_node *np;

    tick_cnt++; 
    while(!TAILQ_EMPTY(&active_timers) && (np=TAILQ_FIRST(&active_timers)) && np->fire  <= tick_cnt) {
        disarm_timer(np);
        if(np->cb(np->user_data) == CB_RETURN_FREE_TIMER) free_timer(np);
    }

}

/*  设置模拟时钟滴答，其功能很像真实时钟中断。
    我们使用 *NIX 信号和进程定时器，因为在几乎任何 *NIX 系统上都可用。
    虽不完美，但足以说明问题。 */
static int init_ticker(unsigned ms) {
    struct itimerval it;
    struct timeval tv;

    tv.tv_sec = 0;
    tv.tv_usec = ms*1000;

    it.it_interval = it.it_value = tv;
    signal(SIGALRM, clock_tick);
    return setitimer(ITIMER_REAL, &it, NULL);
}

/* 用户定时器回调函数 */
int tcb(void *data) {
    struct timer_node *np = data;

    /* 通常你不会只因为定时器就 printf()
       但示例中足以说明问题。 */
    printf("Timer Callback : %llu\n", np->fire);
    return CB_RETURN_FREE_TIMER;
}

int main(int argc, char* argv[]) {
    struct timer_node *np;
    int i;

    init_timers(); /* 初始化定时器子系统 */
    init_ticker(1); /* 1ms 滴答，模拟硬件时钟 */

    /* 创建一批从 1 到 5000ms 的定时器并启动 */
    for (i=0 ; i < NUM_TIMERS ; i++) {
        if((np = alloc_timer()) == NULL) {
            perror("Fatal! we ran out of timers?");
            exit(EXIT_FAILURE);
        }

        if(set_timer(np, 0, (rand()+1) % MAX_RANDOM_TIME_MS , tcb, np) == -1) {
            perror("Fatal! Bad timer set!");
            exit(EXIT_FAILURE);
        }
        arm_timer(np);
    }

    /* 空转等待定时器到期 - 不优雅但简单 */
    while(1) {
        sleep(100);
    }

    /* 永远不会到达 */

    return 0;
}
```

## 分析

- 使用 BSD 的 `TAILQ`（尾队列）宏维护两个链表：`active_timers`（活动）与 `free_timers`（空闲）。
- 采用**固定内存池**：`timer_memory` 预先分配 `NUM_TIMERS` 个节点，避免了动态分配。
- `arm_timer` 按触发时间 `fire` **有序插入**，使链表按时间升序排列。
- 每次 `clock_tick` 递增 `tick_cnt`，依次触发所有 `fire <= tick_cnt` 的定时器。
- 回调返回 `CB_RETURN_FREE_TIMER` 时，把定时器归还空闲链表（可复用）。

## 相关文档
- [[timerWheel]] —— 单层时间轮
- [[taskScheduler]] —— 调度器
- [[memoryPoolAllocator]] —— 类似的固定内存池复用思路
