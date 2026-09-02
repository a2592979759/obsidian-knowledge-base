---
title: RTOS 与 TDMA 裸机调度
tags: [embedded, rtos, 调度, tdma, 裸机, 状态机]
---

# RTOS 与 TDMA 裸机调度

> 本文整理自 `Rtos与TDMA裸机调度.txt`，核心是回答一个问题：**一段业务，到底该用 TDMA 裸机协作式调度，还是上 RTOS 抢占式调度？** 原文的对比表与"决策分水岭"判断是精华，这里保留并扩展开——补了一个可落地的 TDMA 调度器骨架，以及"何时必须上 RTOS"的判别清单。

---

## 一、核心机制对比（RTOS vs. TDMA 裸机）

| 维度 | RTOS 抢占式调度 | TDMA 裸机协作式 |
|------|----------------|----------------|
| **控制权** | 内核自动强切：时间片耗尽或高优先级就绪，立即打断当前任务 | 业务自己让：时隙固定，任务执行完或主动分段交还 CPU |
| **实时性** | 硬实时（确定性的抢占延迟，通常 us 级） | 软实时（时隙边界响应，延迟取决于当前任务最长片） |
| **资源开销** | 重：每任务独立栈（RAM 开销大）、上下文切换消耗 CPU、内核代码复杂 | 极轻：单主栈，无上下文切换指令，代码就一个 `if(tick++)` 循环 |
| **代码复杂度** | 高（处理信号量、队列、优先级反转） | 极低（纯顺序逻辑 + 时间片判断） |

> **一句话：** RTOS 用"内核帮我把任务切开"换响应性；TDMA 用"我自己把任务切成段"换零开销。

---

## 二、为何「TDMA + 表驱动状态机」是裸机最强组合？

这套方案的精髓在于 **时间与业务解耦**：

- **调度器（纯时基）**：1ms Tick 只负责递进时间，不做任何业务判断。
- **业务（纯状态机）**：每个任务拆成 N 个非阻塞步骤，通过查表跳转执行。

**效果：** 把"长函数"拆成"微步"，每个时隙只跑一步，天然保证了最长执行时间可控，代码清爽且无动态内存风险。

### TDMA 调度器骨架

这是"时间与业务解耦"落地的最小骨架（前后台 + 表驱动状态机），命名遵循 `mcu-soft-architecture` / `embedded-c-naming`：

```c
/* sched.h — 前后台协作式调度器 */
typedef void (*sched_task_callback_t)(void);

typedef struct sSched_task
{
    unsigned int      uw_period;     /* 执行周期（tick 数） */
    unsigned int      uw_remaining;  /* 距下次触发还剩多少 tick */
    volatile unsigned char uc_ready; /* 就绪标志：1 = 该跑了 */
} sSched_task_t;

void sched_init(void);
int  sched_add_task(u32 u32Period, sched_task_callback_t pfRun);  /* 形参：位宽别名 + 驼峰 */
void sched_tick(void);   /* 1ms 节拍 ISR 里调用 */
void sched_poll(void);   /* 主循环里调用 */
```

```c
/* sched.c — 节拍 ISR 只递减计数、置就绪标志，不做业务 */
void sched_tick(void)
{
    unsigned int uw_idx;
    for(uw_idx = 0u; uw_idx < gs_uw_task_cnt; uw_idx++)
    {
        if(0u == gs_ast_task[uw_idx].uc_ready)
        {
            if(0u == gs_ast_task[uw_idx].uw_remaining)
            {
                gs_ast_task[uw_idx].uw_remaining = gs_ast_task[uw_idx].uw_period;  /* 重新装填 */
                gs_ast_task[uw_idx].uc_ready     = 1u;                             /* 置就绪 */
            }
            else
            {
                gs_ast_task[uw_idx].uw_remaining--;
            }
        }
    }
}

/* 主循环：先清标志，再执行任务 */
void sched_poll(void)
{
    unsigned int uw_idx;
    for(uw_idx = 0u; uw_idx < gs_uw_task_cnt; uw_idx++)
    {
        if(1u == gs_ast_task[uw_idx].uc_ready)
        {
            gs_ast_task[uw_idx].uc_ready = 0u;    /* 先清就绪，再跑（防止重入） */
            if(NULL != gs_apf_run[uw_idx])
            {
                gs_apf_run[uw_idx]();
            }
        }
    }
}
```

**配套的表驱动状态机**（把"长函数"切成"微步"）：

```c
typedef int (*pf_state_action_t)(void *pvCtx);
static pf_state_action_t gs_apf_state_table[ST_MAX][EVT_MAX];  /* state × event → action */

/* 每个任务在自己的时隙里，只推进状态机一步 */
void TaskPoll(void *pvCtx)
{
    pf_state_action_t pf_action = gs_apf_state_table[uwCurrentState][ueEvent];
    if(NULL != pf_action)
    {
        pf_action(pvCtx);            /* 只做一步，绝不阻塞 */
    }
}
```

> **闭环：** `1ms Tick`（调度器）→ 到期的任务被置 ready → 主循环 poll 执行该任务 → 任务内状态机只走一步（表驱动查表）→ 立刻交还 CPU。因为每步都不阻塞，整个系统最坏执行时间可控。

---

## 三、协作式的「死穴」与决策红线

**致命缺陷：** 一旦某个任务有 `while` 等待或 `delay` 不肯退出，**整个系统的时间轴全部漂移**，且没有任何后备机制能掐断它。

> 这就是协作式调度最要命的点：一个任务赖着不走，全体陪葬，RTOS 还能用抢占踢走它，TDMA 没有这个能力。

**你的决策分水岭（非常正确）：**

| 业务能否切分 | 结论 |
|-------------|------|
| **能切分**（IO 轮询、协议解析、按键扫描） | → **坚持 TDMA**，这是裸机的最优解 |
| **切不动**（擦写大容量 Flash 等待、硬件长耗时指令、外部芯片忙等待） | → **裸机时隙架构直接判死刑**，必须上 RTOS 用抢占来拯救响应性 |

---

## 四、破除幻想：关于「抢占补丁」

你的结论极其清醒：**抢占绝不是 `#define PREEMPT_ENABLE 1` 这种魔法宏。** 它需要：

- **硬件层**：定时器触发 PendSV/SWI 异常。
- **汇编层**：手工保存/恢复 R0-R15、PSP/MSP 栈指针。
- **内核层**：维护就绪列表、处理中断嵌套。

这本质上就是 RTOS 内核的活。**切忌在裸机中自己手撸抢占切换** —— 那等于重新发明半个 RTOS，且极易出栈溢出硬错误，不如直接移植成熟的微小内核（如 FreeRTOS 或 Zephyr）。

### 何时必须上 RTOS —— 判别清单

给一个可对照的 checklist，命中任意一条就该考虑上 RTOS：

- [ ] 存在**不可切分**的硬等待（Flash 擦写、外部芯片忙等、长延时指令）。
- [ ] 有**强实时**需求（中断到响应的确定性延迟要求微秒级）。
- [ ] 需要多个**独立阻塞栈**的任务（每个任务需要自己的运行上下文）。
- [ ] 需要**信号量、互斥量、消息队列**等 IPC 解决多任务协作。
- [ ] 存在**优先级反转**风险（低优先级任务持锁、高优先级等锁，需要互斥量 + 优先级继承）。
- [ ] 业务本身**天生并发**（协议收发、多路采集、多重实时闭环同时进行）。

---

## 一句话终极结论

> **业务能分段，闭眼用 TDMA；业务有硬等待，老实上 RTOS。裸机没有中间态，强行打补丁只会两头不讨好。**

你的这套认知已经比很多"贴吧式裸机调度"高出几个维度，放心用。如果有具体业务拿不准"能不能切分"，可以发出来把关。

---

## 附：决策速查

| 场景 | 推荐方案 | 理由 |
|------|---------|------|
| IO 轮询、按键扫描、协议解析 | TDMA 裸机 | 可切分，零开销，WCET 可控 |
| 电机速度环（需确定性 1ms 周期） | TDMA 裸机（高优先级放表前）或 RTOS | 周期短且可分段 |
| Flash 擦写、外部芯片忙等 | **上 RTOS** | 不可切分，必须抢占 |
| 多路并发 + 强实时 + 复杂度高 | **上 RTOS** | 需要 IPC + 抢占 + 独立栈 |
| 裸机手撸抢占切换 | **绝对禁止** | ≈ 重造半颗 RTOS，极易栈溢出 |
