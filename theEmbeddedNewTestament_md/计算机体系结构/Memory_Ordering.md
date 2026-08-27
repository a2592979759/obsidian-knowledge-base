---
tags:
  - 嵌入式
  - 内存
  - 并发
source: "Computer_architecture/Memory_Ordering.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入学习
>
> 将这些体系结构概念掌握为带参考答案的排序式面试题，并配有交互式深度学习指南。
>
> 👉 **[浏览 MCU 与体系结构相关题目 →](https://embeddedinterviewlab.com/questions/domain/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=computer_architecture)** &nbsp;·&nbsp; **[浏览 MCU 与体系结构指南 →](https://embeddedinterviewlab.com/categories/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=computer_architecture)**

---

# 内存排序 (Memory Ordering)

> **理解内存屏障与原子操作**
> 全面覆盖内存排序模型、同步原语与并发编程

---

## 📋 **目录**

- [内存排序基础](#memory-ordering-fundamentals)
- [内存排序模型](#memory-ordering-models)
- [内存屏障](#memory-barriers-and-fences)
- [原子操作](#atomic-operations)
- [同步原语](#synchronization-primitives)
- [多核系统中的内存排序](#memory-ordering-in-multi-core-systems)
- [性能影响](#performance-implications)
- [最佳实践与指南](#best-practices-and-guidelines)

---

## 🏗️ **内存排序基础**

### **什么是内存排序？**

内存排序指的是规范不同线程或核心的内存操作相对于彼此执行方式的规则。在现代多核系统中，硬件和软件优化可能会重排内存操作，使得实际执行顺序与程序顺序不同。

理解内存排序对于编写正确的并发程序至关重要，因为多线程应用的行为取决于底层硬件和编程语言提供的内存排序保证。

### **内存排序理念**

内存排序体现了宽松一致性的原则，即只要重排不违反指定的内存排序约束，就允许硬件和编译器为性能优化而重排操作。

这种方法有几个好处：
1. **性能**：允许硬件和编译器优化
2. **可扩展性**：在多核系统中实现更好的性能
3. **灵活性**：为不同使用场景提供不同的排序保证
4. **效率**：最小化不必要的同步开销

### **为什么内存排序很重要**

在单线程程序中，内存操作表现为按程序顺序执行。然而，在多线程程序中，这种假设可能导致微妙的错误：

```
Memory Ordering Problem Example:
┌─────────────────────────────────────────────────────────────────┐
│  Thread 1                    Thread 2                          │
│  ┌─────────────────────────┐ ┌─────────────────────────────┐   │
│  │ x = 1;                  │ │ while (flag == 0) { }       │   │
│  │ flag = 1;               │ │ print(x);                   │   │
│  │ └─────────────────────────┘ └─────────────────────────────┘   │
│                                                                 │
│  Problem: Without proper memory ordering, Thread 2 might      │
│  see flag = 1 before x = 1, printing an uninitialized value   │
└─────────────────────────────────────────────────────────────────┘
```

### **内存重排类型**

内存操作可以通过多种方式重排：

1. **编译器重排**：重排操作的编译器优化
2. **硬件重排**：CPU 乱序执行和内存子系统重排
3. **缓存一致性**：不同核心以不同顺序看到操作
4. **内存控制器重排**：内存控制器优化访问模式

---

## 📊 **内存排序模型**

### **顺序一致性**

顺序一致性是最强的内存排序模型，要求所有内存操作看起来以遵循各线程程序顺序的单一顺序执行。

```
Sequential Consistency Example:
┌─────────────────────────────────────────────────────────────────┐
│  Thread 1:        Thread 2:                                    │
│  x = 1;           y = 1;                                       │
│  r1 = y;          r2 = x;                                       │
│                                                                 │
│  Possible outcomes under sequential consistency:                │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ x    │ y    │ r1   │ r2   │  Description                    │ │
│  │ 1    │ 1    │ 0    │ 0    │  Both reads see initial values  │ │
│  │ 1    │ 1    │ 1    │ 0    │  Thread 1 sees Thread 2's write │ │
│  │ 1    │ 1    │ 0    │ 1    │  Thread 2 sees Thread 1's write │ │
│  │ 1    │ 1    │ 1    │ 1    │  Both see each other's writes   │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  Impossible outcome:                                            │
│  x=1, y=1, r1=0, r2=0 (if both reads happen after both writes)│
└─────────────────────────────────────────────────────────────────┘
```

### **总存储排序 (TSO)**

总存储排序（Total Store Ordering）允许存储-加载重排，但保持其他排序约束。该模型被 x86 处理器使用，在性能与程序员预期之间提供了良好的平衡。

```
TSO Memory Ordering:
┌─────────────────────────────────────────────────────────────────┐
│  TSO Constraints:                                              │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ Load-Load   │ Store-Store │  Store-Load                     │ │
│  │ Ordering    │ Ordering    │  Ordering                       │ │
│  │ Maintained  │ Maintained  │  May be reordered               │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  Example of allowed reordering:                                │ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ Original:   store A, load B                                │ │
│  │ Reordered:  load B, store A                                │ │
│  │ (Store-Load reordering allowed)                            │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### **部分存储排序 (PSO)**

部分存储排序（Partial Store Ordering）允许存储-加载和存储-存储重排，为硬件优化提供更多灵活性，但需要更小心地编程。

### **弱排序**

弱排序（Weak Ordering）允许大多数重排，但需要显式同步操作来建立排序约束。该模型为硬件优化提供最大灵活性。

### **释放一致性**

释放一致性（Release Consistency）在特定点（获取与释放操作）提供同步，同时允许在这些点之间重排。

---

## 🚧 **内存屏障**

### **内存屏障基础**

内存屏障（也称为围栏，fences）是强制内存操作排序约束的指令。它们防止某些类型的重排，并确保内存操作以预期顺序对其他线程可见。

### **内存屏障类型**

#### **加载-加载屏障 (LL)**

加载-加载屏障确保屏障之前的所有加载在屏障之后的任何加载开始之前完成。

```
Load-Load Barrier Example:
┌─────────────────────────────────────────────────────────────────┐
│  Without Barrier:                                              │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ load A      │ load B      │  load C                         │ │
│  │             │             │                                 │ │
│  │ Possible reordering: load B, load A, load C                 │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  With Load-Load Barrier:                                       │ │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ load A      │ LL Barrier  │  load B                         │ │
│  │             │             │                                 │ │
│  │ load C      │             │  load D                         │ │
│  │             │             │                                 │ │
│  │ A and C must complete before B and D begin                  │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### **存储-存储屏障 (SS)**

存储-存储屏障确保屏障之前的所有存储在屏障之后的任何存储开始之前完成。

```
Store-Store Barrier Example:
┌─────────────────────────────────────────────────────────────────┐
│  Without Barrier:                                              │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ store A     │ store B     │  store C                        │ │
│  │             │             │                                 │ │
│  │ Possible reordering: store B, store A, store C              │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  With Store-Store Barrier:                                     │ │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ store A     │ SS Barrier  │  store B                        │ │
│  │             │             │                                 │ │
│  │ store C     │             │  store D                        │ │
│  │             │             │                                 │ │
│  │ A and C must complete before B and D begin                  │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### **加载-存储屏障 (LS)**

加载-存储屏障确保屏障之前的所有加载在屏障之后的任何存储开始之前完成。

#### **存储-加载屏障 (SL)**

存储-加载屏障确保屏障之前的所有存储在屏障之后的任何加载开始之前完成。

#### **全内存屏障 (MF)**

全内存屏障确保屏障之前的所有内存操作在屏障之后的任何内存操作开始之前完成。

### **内存屏障实现**

内存屏障在不同体系结构上实现方式不同：

#### **x86 内存屏障**

```c
// x86 内存屏障
#include <immintrin.h>

void x86_memory_barriers() {
    // 编译器屏障（防止编译器重排）
    __asm__ __volatile__("" ::: "memory");
    
    // 全内存屏障
    _mm_mfence();
    
    // 存储屏障
    _mm_sfence();
    
    // 加载屏障
    _mm_lfence();
}
```

#### **ARM 内存屏障**

```c
// ARM 内存屏障
void arm_memory_barriers() {
    // 全内存屏障
    __asm__ __volatile__("dmb ish" ::: "memory");
    
    // 存储屏障
    __asm__ __volatile__("dmb ishst" ::: "memory");
    
    // 加载屏障
    __asm__ __volatile__("dmb ishld" ::: "memory");
}
```

---

## ⚛️ **原子操作**

### **原子操作基础**

原子操作是看起来作为单一、不可分割单元执行的操作。它们对于实现同步原语和确保并发程序的正确行为至关重要。

### **原子操作类型**

#### **读-改-写操作**

读-改-写操作原子地读取一个值、修改它并写回：

1. **比较并交换（Compare-and-Swap, CAS）**：原子地比较并有条件地交换
2. **取并加（Fetch-and-Add）**：原子地加一个值并返回旧值
3. **测试并设置（Test-and-Set）**：原子地设置一个值并返回旧值

```
Atomic Compare-and-Swap:
┌─────────────────────────────────────────────────────────────────┐
│  CAS Operation:                                                │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ bool CAS(T* ptr, T expected, T desired)                    │ │
│  │ {                                                           │ │
│  │     if (*ptr == expected) {                                 │ │
│  │         *ptr = desired;                                     │ │
│  │         return true;                                        │ │
│  │     }                                                       │ │
│  │     return false;                                           │ │
│  │ }                                                           │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  Usage Example:                                                │ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ while (!CAS(&lock, 0, 1)) {                                │ │
│  │     // Spin until lock is acquired                          │ │
│  │ }                                                           │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

#### **加载与存储操作**

原子加载和存储操作确保操作原子地执行：

```c
// C11 原子操作
#include <stdatomic.h>

void atomic_operations_example() {
    atomic_int counter = ATOMIC_VAR_INIT(0);
    
    // 原子加载
    int value = atomic_load(&counter);
    
    // 原子存储
    atomic_store(&counter, 42);
    
    // 原子取并加
    int old_value = atomic_fetch_add(&counter, 10);
    
    // 原子比较并交换
    int expected = 42;
    int desired = 43;
    bool success = atomic_compare_exchange_weak(&counter, &expected, desired);
}
```

### **原子操作中的内存排序**

原子操作可以指定内存排序约束：

```c
// 带内存排序的原子操作
void atomic_with_ordering() {
    atomic_int flag = ATOMIC_VAR_INIT(0);
    atomic_int data = ATOMIC_VAR_INIT(0);
    
    // 释放存储：确保所有先前操作可见
    atomic_store_explicit(&data, 42, memory_order_release);
    
    // 获取加载：确保所有后续操作可见
    int value = atomic_load_explicit(&data, memory_order_acquire);
    
    // 宽松操作：无排序保证
    atomic_fetch_add_explicit(&flag, 1, memory_order_relaxed);
}
```

---

## 🔒 **同步原语**

### **带内存排序的互斥锁实现**

互斥锁可以用原子操作和内存屏障实现：

```c
// 简单自旋锁互斥锁实现
typedef struct {
    atomic_int locked;
} spinlock_t;

void spinlock_init(spinlock_t* lock) {
    atomic_store(&lock->locked, 0);
}

void spinlock_acquire(spinlock_t* lock) {
    while (atomic_exchange_explicit(&lock->locked, 1, 
                                   memory_order_acquire)) {
        // 自旋直到获得锁
        while (atomic_load_explicit(&lock->locked, 
                                   memory_order_relaxed)) {
            // 可选：yield 或 pause
        }
    }
}

void spinlock_release(spinlock_t* lock) {
    atomic_store_explicit(&lock->locked, 0, memory_order_release);
}
```

### **信号量实现**

信号量也可以用原子操作实现：

```c
// 二值信号量实现
typedef struct {
    atomic_int count;
} semaphore_t;

void semaphore_init(semaphore_t* sem, int initial_count) {
    atomic_store(&sem->count, initial_count);
}

void semaphore_wait(semaphore_t* sem) {
    int expected;
    do {
        expected = atomic_load(&sem->count);
        if (expected <= 0) {
            // 等待信号
            continue;
        }
    } while (!atomic_compare_exchange_weak(&sem->count, &expected, expected - 1));
}

void semaphore_signal(semaphore_t* sem) {
    atomic_fetch_add(&sem->count, 1);
}
```

---

## 🔄 **多核系统中的内存排序**

### **缓存一致性与内存排序**

缓存一致性协议确保所有核心看到一致的内存视图，但它们不保证内存排序。仍然需要内存屏障来建立排序约束。

```
Cache Coherency vs. Memory Ordering:
┌─────────────────────────────────────────────────────────────────┐
│  Cache Coherency:                                              │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ Ensures all cores see the same value for a memory location │ │
│  │ Does NOT guarantee the order of operations                  │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
│                                                                 │
│  Memory Ordering:                                               │ │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐ │
│  │ Establishes the order of operations                         │ │
│  │ Requires explicit memory barriers                           │ │
│  └─────────────┴─────────────┴─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### **不同体系结构中的内存排序**

不同处理器体系结构提供不同的内存排序保证：

#### **x86/x64 体系结构**

- **TSO（总存储排序）**：允许存储-加载重排
- **强内存模型**：默认防止大多数重排
- **显式屏障**：`mfence`、`sfence`、`lfence` 指令

#### **ARM 体系结构**

- **弱内存模型**：默认允许大多数重排
- **显式屏障**：`dmb`、`dsb`、`isb` 指令
- **加载获取/存储释放**：内置排序语义

#### **PowerPC 体系结构**

- **弱内存模型**：默认允许大多数重排
- **显式屏障**：`sync`、`lwsync`、`ptsync` 指令
- **加载保留/存储条件**：原子操作

---

## ⚡ **性能影响**

### **内存屏障性能成本**

内存屏障具有随体系结构而变的性能成本：

1. **流水线停顿**：屏障可能引起流水线停顿
2. **内存访问串行化**：某些屏障使内存访问串行化
3. **缓存效应**：屏障可能影响缓存行为
4. **编译限制**：屏障限制编译器优化

### **优化策略**

多种策略可以最小化内存屏障的性能影响：

#### **屏障放置优化**

```c
// 最小化屏障使用
void optimized_synchronization() {
    // 执行不需要排序的工作
    int local_result = compute_something();
    
    // 尽可能晚地放置屏障
    atomic_thread_fence(memory_order_release);
    
    // 只有最终结果需要排序
    atomic_store(&shared_result, local_result);
}
```

#### **批量操作**

```c
// 批量操作以减少屏障开销
void batched_operations() {
    // 收集多个更新
    int updates[10];
    for (int i = 0; i < 10; i++) {
        updates[i] = compute_update(i);
    }
    
    // 所有更新用单个屏障
    atomic_thread_fence(memory_order_release);
    
    // 原子地应用所有更新
    for (int i = 0; i < 10; i++) {
        atomic_store(&shared_data[i], updates[i]);
    }
}
```

---

## 🎯 **最佳实践与指南**

### **一般内存排序指南**

1. **使用最弱的可行排序**：选择提供所需保证的最弱内存排序
2. **了解目标体系结构**：不同体系结构具有不同的默认行为
3. **彻底测试**：内存排序错误可能是微妙且依赖体系结构的
4. **记录假设**：清晰地记录内存排序要求

### **应避免的常见陷阱**

1. **假设顺序一致性**：不要假设操作按程序顺序执行
2. **忽略编译器重排**：编译器优化可能重排操作
3. **混用内存模型**：在程序内保持内存排序一致
4. **过度同步**：不要使用比所需更强的排序

### **调试内存排序问题**

内存排序问题可能难以调试：

1. **使用内存排序工具**：像 ThreadSanitizer 这样的工具可以检测某些问题
2. **压力测试**：在各种系统负载下运行并发测试
3. **体系结构特定测试**：在不同处理器体系结构上测试
4. **形式化验证**：对关键并发代码使用形式化方法

---

## 📚 **进一步阅读与资源**

- **Memory Barriers: a Hardware View for Software Hackers**，作者 Paul E. McKenney
- **The Art of Multiprocessor Programming**，作者 Herlihy 和 Shavit
- **C++ Concurrency in Action**，作者 Anthony Williams
- **ARM Architecture Reference Manual**
- **Intel 64 and IA-32 Architectures Software Developer's Manual**

---

*本内存排序综合指南为理解现代多核系统如何处理并发内存访问提供了基础。这里涵盖的概念对于处理并发编程和理解多线程应用行为的嵌入式软件工程师至关重要。*
