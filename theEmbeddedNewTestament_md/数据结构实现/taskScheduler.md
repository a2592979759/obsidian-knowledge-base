---
tags: [RTOS, 调度器, 嵌入式, 任务, 位图]
source: Data_Struct_Implementation/taskScheduler
created: 2026-08-27
---

# 示例 RTOS 任务调度器

这是一个采用**时间片轮转 + 优先级抢占**调度策略的示例 RTOS 任务调度器：

[什么是“实时”？抢占式、基于优先级的调度](https://www.youtube.com/watch?v=kLxxXNCrY60&list=PLC8H7gcKF_bSt4vGkUq6mwlPbf2fwnqy6&index=7&ab_channel=QuantumLeaps%2CLLC)

> 注：源码来自于 Quantum Leaps 的 MIROS 教学 RTOS（`taskScheduler/exampleRTOSScheduler/mirosExample/`），包括 `main.c`、`miros.c`、`miros.h`、`bsp.c`、`bsp.h`。

### 代码分析

#### **数据结构**
本示例有三种重要的数据结构：

1. ***OSThread*** 结构体，本质上是**任务控制块（TCB）**，记录栈指针地址、超时（timeout）和优先级信息。
2. ***OSThread \*OS_thread[32 + 1]***：OSThread 指针数组，按任务的优先级值索引。这意味着有 33 个静态优先级，且任何两个任务都不能有相同优先级。
3. ***uint32_t OS_readySet, OS_delayedSet***：位图，分别表示“就绪（Ready）”和“阻塞（Block）”集合中的可用任务数。意味着任何时候都可能有多个任务可供调度。

```c
/* 任务控制块 (TCB) */
typedef struct {
    void *sp; /* 栈指针 */
    uint32_t timeout; /* 超时延时的向下计数器 */
    uint8_t prio; /* 任务优先级 */
    /* ... 与任务相关的其它属性 */
} OSThread;

OSThread * volatile OS_curr; /* 指向当前任务 */
OSThread * volatile OS_next; /* 指向下一个要运行的任务 */

OSThread *OS_thread[32 + 1]; /* 已启动的任务数组 */
uint32_t OS_readySet; /* 就绪运行的任务位掩码 */
uint32_t OS_delayedSet; /* 被延时的任务位掩码 */
```

#### **方法声明**

```c
void OS_init(void *stkSto, uint32_t stkSize);

/* 处理空闲条件的回调 */
void OS_onIdle(void);

/* 此函数必须在中断被禁用时调用 */
void OS_sched(void);

/* 将控制权交给 RTOS 以运行任务 */
void OS_run(void);

/* 阻塞延时 */
void OS_delay(uint32_t ticks);

/* 处理所有超时 */
void OS_tick(void);

/* 配置并启动中断的回调 */
void OS_onStartup(void);

void OSThread_start(
    OSThread *me,
    uint8_t prio, /* 任务优先级 */
    OSThreadHandler threadHandler,
    void *stkSto, uint32_t stkSize);
```

#### **方法实现**

最重要的函数是 `OS_tick` 和 `OS_delay`。`OS_tick` 是每个 systick 事件的时间片轮转调度部分，它会把 `delaySet` 中任务的超时值减一。一旦超时值归零，任务会从 `delaySet` 移到 `readySet`，只需在两个位图变量中设置/清除对应比特位。

```c
OSThread *t = OS_thread[LOG2(workingSet)];
```

这是在位图中查找下一个要调度任务的函数。

```c
#define LOG2(x) (32U - __clz(x))  // __clz(x) 统计前导零的个数
```

此操作本质上计算从最高位（MSB）起第一个置位的位置，也就是**最高优先级任务的下标**。我们可以用这个下标到任务指针数组中查找对应的任务指针，因为数组是按优先级索引的。更多细节请参考下方链接。

`main.c`：

```c
#include <stdint.h>
#include "miros.h"
#include "bsp.h"

uint32_t stack_blinky1[40];
OSThread blinky1;
void main_blinky1() {
    while (1) {
        uint32_t volatile i;
        for (i = 1500U; i != 0U; --i) {
            BSP_ledGreenOn();
            BSP_ledGreenOff();
        }
        OS_delay(1U); /* 阻塞 1 个 tick */
    }
}

uint32_t stack_blinky2[40];
OSThread blinky2;
void main_blinky2() {
    while (1) {
        uint32_t volatile i;
        for (i = 3*1500U; i != 0U; --i) {
            BSP_ledBlueOn();
            BSP_ledBlueOff();
        }
        OS_delay(50U); /* 阻塞 50 个 tick */
    }
}

uint32_t stack_blinky3[40];
OSThread blinky3;
void main_blinky3() {
    while (1) {
        BSP_ledRedOn();
        OS_delay(BSP_TICKS_PER_SEC / 3U);
        BSP_ledRedOff();
        OS_delay(BSP_TICKS_PER_SEC * 3U / 5U);
    }
}

uint32_t stack_idleThread[40];

int main() {
    BSP_init();
    OS_init(stack_idleThread, sizeof(stack_idleThread));

    /* 启动 blinky1 任务 */
    OSThread_start(&blinky1,
                   5U, /* 优先级 */
                   &main_blinky1,
                   stack_blinky1, sizeof(stack_blinky1));

    /* 启动 blinky2 任务 */
    OSThread_start(&blinky2,
                   2U, /* 优先级 */
                   &main_blinky2,
                   stack_blinky2, sizeof(stack_blinky2));

    /* 启动 blinky3 任务 */
    //OSThread_start(&blinky3,
    //               1U, /* 优先级 */
    //               &main_blinky3,
    //               stack_blinky3, sizeof(stack_blinky3));

    /* 把控制权交给 RTOS 运行任务 */
    OS_run();

    //return 0;
}
```

`miros.c`（核心调度与上下文切换）：

```c
#include <stdint.h>
#include "miros.h"
#include "qassert.h"

Q_DEFINE_THIS_FILE

OSThread * volatile OS_curr; /* 指向当前任务 */
OSThread * volatile OS_next; /* 指向下一个要运行的任务 */

OSThread *OS_thread[32 + 1]; /* 已启动的任务数组 */
uint32_t OS_readySet; /* 就绪运行的任务位掩码 */
uint32_t OS_delayedSet; /* 被延时的任务位掩码 */

#define LOG2(x) (32U - __clz(x))

OSThread idleThread;
void main_idleThread() {
    while (1) {
        OS_onIdle();
    }
}

void OS_init(void *stkSto, uint32_t stkSize) {
    /* 把 PendSV 中断优先级设为最低 0xFF */
    *(uint32_t volatile *)0xE000ED20 |= (0xFFU << 16);

    /* 启动 idleThread 任务 */
    OSThread_start(&idleThread,
                   0U, /* 空闲任务优先级 */
                   &main_idleThread,
                   stkSto, stkSize);
}

void OS_sched(void) {
    /* OS_next = ... */
    if (OS_readySet == 0U) { /* 空闲条件？ */
        OS_next = OS_thread[0]; /* 空闲任务 */
    }
    else {
        OS_next = OS_thread[LOG2(OS_readySet)];
        Q_ASSERT(OS_next != (OSThread *)0);
    }

    /* 如有需要，触发 PendSV */
    if (OS_next != OS_curr) {
        *(uint32_t volatile *)0xE000ED04 = (1U << 28);
    }
}

void OS_run(void) {
    /* 配置并启动中断的回调 */
    OS_onStartup();

    __disable_irq();
    OS_sched();
    __enable_irq();

    /* 以下代码不应执行 */
    Q_ERROR();
}

void OS_tick(void) {
    uint32_t workingSet = OS_delayedSet;
    while (workingSet != 0U) {
        OSThread *t = OS_thread[LOG2(workingSet)];
        uint32_t bit;
        Q_ASSERT((t != (OSThread *)0) && (t->timeout != 0U));

        bit = (1U << (t->prio - 1U));
        --t->timeout;
        if (t->timeout == 0U) {
            OS_readySet   |= bit;  /* 插入到就绪集合 */
            OS_delayedSet &= ~bit; /* 从延时集合移除 */
        }
        workingSet &= ~bit; /* 从工作集合移除 */
    }
}

void OS_delay(uint32_t ticks) {
    uint32_t bit;
    __disable_irq();

    /* 永远不要从 idleThread 调用 OS_delay */
    Q_REQUIRE(OS_curr != OS_thread[0]);

    OS_curr->timeout = ticks;
    bit = (1U << (OS_curr->prio - 1U));
    OS_readySet &= ~bit;
    OS_delayedSet |= bit;
    OS_sched();
    __enable_irq();
}

void OSThread_start(
    OSThread *me,
    uint8_t prio, /* 任务优先级 */
    OSThreadHandler threadHandler,
    void *stkSto, uint32_t stkSize)
{
    /* 把栈顶向下取整到 8 字节边界
    * 注意：ARM Cortex-M 栈从高地址向低地址增长
    */
    uint32_t *sp = (uint32_t *)((((uint32_t)stkSto + stkSize) / 8) * 8);
    uint32_t *stk_limit;

    /* 优先级必须在范围内
    * 且该优先级必须未被占用
    */
    Q_REQUIRE((prio < Q_DIM(OS_thread))
              && (OS_thread[prio] == (OSThread *)0));

    *(--sp) = (1U << 24);  /* xPSR */
    *(--sp) = (uint32_t)threadHandler; /* PC */
    *(--sp) = 0x0000000EU; /* LR  */
    *(--sp) = 0x0000000CU; /* R12 */
    *(--sp) = 0x00000003U; /* R3  */
    *(--sp) = 0x00000002U; /* R2  */
    *(--sp) = 0x00000001U; /* R1  */
    *(--sp) = 0x00000000U; /* R0  */
    /* 另外伪造寄存器 R4-R11 */
    *(--sp) = 0x0000000BU; /* R11 */
    *(--sp) = 0x0000000AU; /* R10 */
    *(--sp) = 0x00000009U; /* R9 */
    *(--sp) = 0x00000008U; /* R8 */
    *(--sp) = 0x00000007U; /* R7 */
    *(--sp) = 0x00000006U; /* R6 */
    *(--sp) = 0x00000005U; /* R5 */
    *(--sp) = 0x00000004U; /* R4 */

    /* 把栈顶保存在任务的属性里 */
    me->sp = sp;

    /* 把栈底向上取整到 8 字节边界 */
    stk_limit = (uint32_t *)(((((uint32_t)stkSto - 1U) / 8) + 1U) * 8);

    /* 用 0xDEADBEEF 预填充栈的未使用部分 */
    for (sp = sp - 1U; sp >= stk_limit; --sp) {
        *sp = 0xDEADBEEFU;
    }

    /* 向操作系统注册任务 */
    OS_thread[prio] = me;
    me->prio = prio;
    /* 使任务就绪 */
    if (prio > 0U) {
        OS_readySet |= (1U << (prio - 1U));
    }
}

/* 这是上下文切换代码，每次任务被抢占时触发。 */
__asm
void PendSV_Handler(void) {
    IMPORT  OS_curr  /* 外部变量 */
    IMPORT  OS_next  /* 外部变量 */

    /* __disable_irq(); */
    CPSID         I

    /* if (OS_curr != (OSThread *)0) { */
    LDR           r1,=OS_curr
    LDR           r1,[r1,#0x00]
    CBZ           r1,PendSV_restore

    /*     push registers r4-r11 on the stack */
    PUSH          {r4-r11}

    /*     OS_curr->sp = sp; */
    LDR           r1,=OS_curr
    LDR           r1,[r1,#0x00]
    STR           sp,[r1,#0x00]
    /* } */

PendSV_restore
    /* sp = OS_next->sp; */
    LDR           r1,=OS_next
    LDR           r1,[r1,#0x00]
    LDR           sp,[r1,#0x00]

    /* OS_curr = OS_next; */
    LDR           r1,=OS_next
    LDR           r1,[r1,#0x00]
    LDR           r2,=OS_curr
    STR           r1,[r2,#0x00]

    /* pop registers r4-r11 */
    POP           {r4-r11}

    /* __enable_irq(); */
    CPSIE         I

    /* 返回到下一个任务 */
    BX            lr
}
```

## 分析

- **优先级抢占 + 就绪位图**：最高优先级任务通过 `LOG2(OS_readySet)` 在下标中找到（数值即最高置位对应的优先级）。用位图能 O(1) 找到最高优先级就绪任务。
- **时间片轮转延时**：`OS_delay` 把任务从 `readySet` 移到 `delayedSet` 并设置 `timeout`；`OS_tick` 在每个 systick 递减所有延时任务的 `timeout`，归零后移回 `readySet`。时间片轮转效果来自时间到期的多个同优先级任务依次被调度。
- **上下文切换**：`PendSV_Handler` 保存/恢复寄存器（r4-r11）并切换 `OS_curr` / `OS_next` 的 `sp`。属于 Cortex-M 的典型 RTOS 上下文切换实现。
- **伪栈帧初始化**：`OSThread_start` 在任务首次调度前，把各个 CPU 寄存器的初始值手工压入任务栈，模拟一次“异常返回”进入任务。

## 相关文档
- [[stateMachine]] —— 状态机
- [[binaryHeap]] —— 优先队列（调度器可用堆）
- [[timerList]] / [[timerWheel]] —— 定时器
