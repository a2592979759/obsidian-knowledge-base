---
title: 嵌入式 C 编程思想
tags: [embedded, c, 架构, 设计模式, 命名规范]
---

# 嵌入式 C 编程思想

> 本文整理自 `嵌入式C编程思想.txt`，把**可落地的三条编程铁律、三种设计模式、以及嵌入式专属避坑提醒**组织成连贯文档。原话判断尽量保留，补充了扩展说明与符合命名规范的 C 片段对照。

---

## 一、编程思想（落地铁律）

### 1. 高内聚（文件域封装）

```c
/* xxx.c */
#include "xxx.h"

/* 内部子函数必须 static，绝不暴露到外部 */
static void prvDoWork(void) ...
static void prvParseFrame(void) ...

/* 对外只暴露 xxx_init / xxx_process */
void xxx_init(void) ...
void xxx_process(void) ...
```

**核心原则：数据私有化。** 模块内部的**大数组、状态变量一律 static**，绝对不暴露给外部直接修改。外部想改只能通过模块提供的 API —— 这是"数据被谁拥有、谁就有权改"的唯一确定方式。

> 一个模块 = 一个".c 提供能力 + .h 声明接口"的黑盒。内部实现随便改，外部调用方无感知。

### 2. 低耦合（句柄模式）

既然少用全局变量，就强制使用**不透明指针（Opaque Pointer）**：

```c
/* xxx.h — 只声明不完整类型，外部完全不知内部布局 */
typedef struct sXxx xxx_t;

/* 外部只传指针（句柄），不知内部内存布局 */
void xxx_init(xxx_t *pstXxx);
```

```c
/* xxx.c — 完整定义结构体 */
typedef struct sXxx
{
    unsigned int  uw_state;     /* 变量：字长标签 + 蛇形 */
    unsigned char uc_buffer[64];
} xxx_t;
```

**收益：** 修改结构体成员不影响外部调用方。外部拿到的只是一个不透明句柄，结构体内部怎么加字段、怎么重排，全部封装在 `.c` 里。

### 3. 开闭原则（对抗需求变更）

> 对扩展开放，对修改关闭。需求变了，不动老代码，只加新扩展点。

嵌入式最实用的两种落地方式：

- **回调钩子（Callback Hook）**：库代码写死 `__weak` 函数，老代码不动，用户在自己的 `.c` 里重写该函数即可扩展功能。
- **弱函数 `__weak`**：STM32 HAL 的经典手法，如 `__weak void HAL_UART_RxCpltCallback(...)`。默认空实现，用户重载即接管。

```c
/* 库代码：默认空实现，编译不报错 */
__weak void HAL_UART_RxCpltCallback(u32 u32Len)
{
    (void)u32Len;   /* 默认什么都不做 */
}

/* 用户的 .c：重写即扩展，库代码零改动 */
void HAL_UART_RxCpltCallback(u32 u32Len)
{
    /* user logic */
}
```

---

## 二、设计模式（C 语言极简实现）

### 1. 单例模式（Singleton）

注意**初始化守卫**，防止多任务环境下重复初始化。返回栈区静态指针绝对安全。

```c
typedef struct sUart
{
    int fd;
} Uart_t;

Uart_t* Uart_GetInstance(void)
{
    static Uart_t st_instance;          /* 局部静态：s_ 前缀，但类型静态这里用 snake 结构体 */
    static int     is_initialized;      /* 局部：无前缀 + 类型标签 */

    if(0 == is_initialized)
    {
        /* 只执行一次初始化 */
        st_instance.fd = 0;
        is_initialized = 1;
    }
    return &st_instance;                /* 返回栈区静态指针，绝对安全 */
}
```

> **避坑：** 这个例子是**单任务**下的单例。真正的多任务环境里 `is_initialized` 竞态需要加锁/关中断保护，否则两个任务可能各自初始化一次。单任务裸机场景下本文写法够用。

### 2. 抽象工厂 / 虚表 VTable（驱动分层核心）

这是**驱动分层（HAL 层与 BSP 层解耦）** 的核心。注意：为了省 RAM，把 `ops` 结构体放在 const 区（Flash），实例结构体只存数据。

```c
typedef struct sDev
{
    int (*pf_init)(void *pvDev);                        /* 函数指针成员：pf 前缀 + 蛇形 */
    int (*pf_read)(void *pvDev, unsigned char *pucBuf, int swLen);
} sDev_ops_t;

typedef struct sDevice
{
    const sDev_ops_t *pst_ops;      /* 指向 Flash 中的常量函数表 */
    unsigned int      uw_base_addr; /* 实例私有数据 */
} sDevice_t;
```

**要点：**
- `pst_ops` 指到 Flash 只读区 —— 每个实例共享同一份函数表，不重复占 RAM。
- 实例结构体只存"这一个设备的私有数据"（如基址、当前状态）。
- 换一个型号的设备，只需给一个新的 `ops` 函数表 + 新的实例数据，**上层驱动代码一行不用改**。

### 3. 状态 / 策略模式（函数指针数组）

不要 switch-case 满天飞，用状态迁移表。这是裸机调度与状态机的核心骨架。

```c
typedef int (*pf_state_action_t)(void *pvCtx);

/* 状态表：state × event → 动作 */
static pf_state_action_t gs_apf_state_table[ST_MAX][EVT_MAX] = {
    [ST_IDLE][EVT_START]   = DoStartAction,
    [ST_BUSY][EVT_TIMEOUT] = DoTimeoutAction,
};

/* 执行：查表 + 调用 */
pf_state_action_t pf_action = gs_apf_state_table[uwCurrentState][ueEvent];
if(NULL != pf_action)
{
    pf_action(pvCtx);
}
```

> 状态表把一个"状态 × 事件"的组合映射到一个动作函数。`switch-case` 在状态少（≤10）时直观；状态多了以后，**表驱动**更容易排障、更容易加状态，也是 `mcu-soft-architecture` 里 FSM 的标准实现之一。

---

## 三、嵌入式专属避坑提醒（关键）

### 1. 慎用 malloc（堆分配）

在单片机（MCU）上，**绝对不建议**在 Factory 或 GetInstance 里动态申请内存，极易产生碎片。

**推荐做法：静态对象池（Static Pool）。** 预先定义一个大数组，工厂函数返回池中空闲元素的指针，申请和释放由自己管理。

```c
#define MAX_OBJ  8u

/* 文件静态：gs_ 前缀 + 数组修饰符 a */
static sObj gs_ast_pool[MAX_OBJ];
static unsigned char gs_uc_used[MAX_OBJ];   /* 占用位图 */

/* 申请：找一个未占用的槽位返回，否则 NULL */
sObj* ObjFactory_Alloc(void)
{
    unsigned int uw_idx;
    for(uw_idx = 0u; uw_idx < MAX_OBJ; uw_idx++)
    {
        if(0u == gs_uc_used[uw_idx])
        {
            gs_uc_used[uw_idx] = 1u;
            return &gs_ast_pool[uw_idx];
        }
    }
    return NULL;   /* 池满 */
}
```

> **权衡：** 静态池能避免碎片和不确定性，代价是"最大数量固定、占用固定 RAM"。MCU 上这通常比 malloc 更可控。

### 2. const 修饰符即"配置"

开闭原则里说的"改配置"，强烈建议将配置参数集合定义为 const 结构体，并放在 `xxx_cfg.h` 中。**编译前修改宏或链接脚本，而非运行时修改。**

```c
/* xxx_cfg.h */
typedef struct sCfg
{
    unsigned int uw_baud_rate;
    unsigned char uc_mode;
} sCfg_t;

extern const sCfg_t gs_cfg_uart;   /* const 全局：gsc_ 前缀 */

/* xxx.c */
const sCfg_t gs_cfg_uart = {
    115200u,     /* baud rate */
    0u,          /* mode */
};
```

### 3. 耦合度的终极检测

如果你的 `.c` 文件需要 `#include` 超过 **3 个**其它模块的 `.h` 文件，说明耦合度过高，建议加一层 **Dispatcher（调度器）** 中间层来解耦。

> **判断标准：** 一个模块的头文件依赖越满天飞，越难单独复用和测试。超过 3 个是这条铁律的简化阈值 —— 超过就停下来想想：这些依赖里有多少其实是"我该只管接口、不该管实现"的。

---

## 一句话小结

**高内聚（static 封装数据 + 只露 init/process）、低耦合（不透明句柄）、开闭（回调/弱函数扩展），再配单例/虚表/状态表三个精简模式，避开 malloc 碎片 —— 这就是嵌入式 C 落地不返工的四根柱子。**
