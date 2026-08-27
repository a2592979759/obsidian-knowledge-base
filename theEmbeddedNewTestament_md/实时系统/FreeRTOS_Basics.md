---
tags:
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Real_Time_Systems/FreeRTOS_Basics.md
created: 2026-08-27
---

# FreeRTOS 基础

> **通过概念而非仅仅代码来理解 FreeRTOS 基础。了解 RTOS 为何重要，以及如何思考实时系统。**

## 📋 **目录**
- [概念 → 为什么重要 → 最小示例 → 动手试试 → 要点](#concept--why-it-matters--minimal-example--try-it--takeaways)
- [核心概念](#core-concepts)
- [FreeRTOS 架构](#freertos-architecture)
- [任务管理](#task-management)
- [同步](#synchronization)
- [配置](#configuration)
- [引导实验](#guided-labs)
- [自测](#check-yourself)
- [交叉链接](#cross-links)

---

## **概念 → 为什么重要 → 最小示例 → 动手试试 → 要点**

**概念**：FreeRTOS 是一个实时操作系统，它通过给每个任务分配一段 CPU 时间来管理多个任务，确保关键操作在需要的时候发生。

**为什么重要**：没有 RTOS，你就得手动管理程序各部分之间的时序、优先级和资源共享。随着复杂度增加，这会变得无法维护，关键操作可能会错过截止时间。

**最小示例**：一个包含两个任务的简单系统——一个每 100ms 闪烁一次 LED，另一个每 500ms 读取一次传感器。FreeRTOS 确保两者可靠运行且互不干扰。

**动手试试**：从单个闪烁 LED 的任务开始，然后添加第二个读取传感器的任务。观察 FreeRTOS 如何自动管理两者。

**要点**：FreeRTOS 提供可预测的时序和资源管理，让你专注于系统应该做什么，而不是如何协调多个操作。

---

## 📋 **速查表：关键要点**

### **FreeRTOS 基础**
- **实时**：保证可预测的时序，而不一定快
- **抢占式**：更高优先级的任务可以打断更低优先级的任务
- **可移植**：可在多种微控制器架构上运行
- **开源**：MIT 许可，无版税或许可费
- **可扩展**：可针对最小或复杂需求进行配置

### **核心组件**
- **调度器(Scheduler)**：根据优先级管理哪个任务何时运行
- **任务管理器(Task Manager)**：处理任务创建、删除和状态转换
- **内存管理器(Memory Manager)**：管理栈分配和内存池
- **定时服务(Timing Services)**：提供延时、超时和周期执行
- **通信(Communication)**：队列、信号量和互斥锁用于任务协调

### **任务状态**
- **已创建(Created)**：任务存在但尚未被调度
- **就绪(Ready)**：任务已准备好运行，等待 CPU 时间
- **运行(Running)**：任务当前正在 CPU 上执行
- **阻塞(Blocked)**：任务在等待某物（延时、数据、资源）
- **已删除(Deleted)**：任务已从系统中移除

### **关键配置选项**
- **configUSE_PREEMPTION**：启用/禁用抢占式调度
- **configTICK_RATE_HZ**：系统节拍频率（通常为 1000Hz）
- **configMAX_PRIORITIES**：最大任务优先级数量
- **configMINIMAL_STACK_SIZE**：任务的最小栈大小
- **configUSE_MUTEXES**：启用互斥锁以保护资源

---

## 🧠 **核心概念**

### **什么是实时？**

实时并不意味着"快"——它意味着**可预测**。实时系统保证操作在其指定的时间限制内完成。

```
┌─────────────────────────────────────────────────────────────┐
│                    Real-Time vs Non-Real-Time              │
├─────────────────────────────────────────────────────────────┤
│                Real-Time System                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │
│  │ Task A  │  │ Task B  │  │ Task A  │  │ Task B  │      │
│  │ 100ms   │  │ 500ms   │  │ 100ms   │  │ 500ms   │      │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │
│  ↑           ↑           ↑           ↑                    │
│  0ms        100ms       200ms       300ms                 │
│                                                           │
│                Non-Real-Time System                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │
│  │ Task A  │  │ Task B  │  │ Task A  │  │ Task B  │      │
│  │ 100ms   │  │ 500ms   │  │ 150ms   │  │ 600ms   │      │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │
│  ↑           ↑           ↑           ↑                    │
│  0ms        100ms       250ms       350ms                 │
│                                                           │
│  ❌ Timing varies - unpredictable!                        │
└─────────────────────────────────────────────────────────────┘
```

### **为什么使用 RTOS 而不是裸机？**

**裸机方式：**
```
┌─────────────────────────────────────────────────────────────┐
│                    Bare Metal Programming                  │
├─────────────────────────────────────────────────────────────┤
│  while(1) {                                               │
│    // Check if it's time to blink LED                     │
│    if (timer_elapsed(100ms)) {                            │
│      toggle_led();                                         │
│      reset_timer();                                        │
│    }                                                       │
│                                                            │
│    // Check if it's time to read sensor                   │
│    if (timer_elapsed(500ms)) {                            │
│      sensor_value = read_sensor();                         │
│      reset_timer();                                        │
│    }                                                       │
│                                                            │
│    // What if we need to add more tasks?                  │
│    // What if priorities change?                          │
│    // What if timing requirements change?                  │
│  }                                                         │
│                                                            │
│  ❌ Becomes unmanageable quickly!                          │
└─────────────────────────────────────────────────────────────┘
```

**FreeRTOS 方式：**
```
┌─────────────────────────────────────────────────────────────┐
│                    FreeRTOS Approach                       │
├─────────────────────────────────────────────────────────────┤
│  // Task 1: Blink LED every 100ms                         │
│  void vBlinkTask(void *pvParameters) {                    │
│    while(1) {                                             │
│      toggle_led();                                         │
│      vTaskDelay(pdMS_TO_TICKS(100));                      │
│    }                                                       │
│  }                                                         │
│                                                            │
│  // Task 2: Read sensor every 500ms                       │
│  void vSensorTask(void *pvParameters) {                    │
│    while(1) {                                             │
│      sensor_value = read_sensor();                         │
│      vTaskDelay(pdMS_TO_TICKS(500));                      │
│    }                                                       │
│  }                                                         │
│                                                            │
│  // FreeRTOS handles the rest automatically!               │
│  ✅ Clean, maintainable, scalable                          │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏗️ **FreeRTOS 架构**

### **系统概述**

FreeRTOS 位于你的应用与硬件之间，管理资源和时序：

```
┌─────────────────────────────────────────────────────────────┐
│                    FreeRTOS Architecture                    │
├─────────────────────────────────────────────────────────────┤
│                Application Layer                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Task 1    │  │   Task 2    │  │   Task 3    │        │
│  │ (Blink LED) │  │ (Read Sens) │  │ (Send Data) │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
├─────────────────────────────────────────────────────────────┤
│                FreeRTOS Kernel                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Scheduler  │  │  Memory     │  │  Timing     │        │
│  │             │  │  Manager    │  │  Services   │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Queues    │  │ Semaphores  │  │   Mutexes   │        │
│  │             │  │             │  │             │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
├─────────────────────────────────────────────────────────────┤
│                Hardware Abstraction Layer                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   Port      │  │   Memory    │  │  Interrupt  │        │
│  │  Layer      │  │   Model     │  │   Handler   │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
├─────────────────────────────────────────────────────────────┤
│                Hardware (MCU)                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │     CPU     │  │    RAM      │  │  Peripherals│        │
│  │             │  │             │  │             │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### **关键组件**

**调度器(Scheduler)**：决定哪个任务何时运行的"大脑"
**内存管理器(Memory Manager)**：处理栈分配和内存池
**定时服务(Timing Services)**：提供延时、超时和周期执行
**通信(Communication)**：队列、信号量和互斥锁用于任务协调

---

## 📋 **任务管理**

### **什么是任务？**

任务就像一个独立运行的程序。把它看作一个有特定工作的工人。

```
┌─────────────────────────────────────────────────────────────┐
│                    Task Lifecycle                          │
├─────────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                │
│  │ Created │───▶│  Ready  │───▶│ Running │                │
│  └─────────┘    └─────────┘    └────┬────┘                │
│                                      │                     │
│                                      ▼                     │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                │
│  │Deleted  │◄───│ Blocked │◄───│         │                │
│  └─────────┘    └─────────┘    └─────────┘                │
│                                                           │
│  • Created: Task exists but not yet scheduled             │
│  • Ready: Task is ready to run, waiting for CPU           │
│  • Running: Task is currently executing                   │
│  • Blocked: Task is waiting for something (delay, data)   │
│  • Deleted: Task has been removed from system             │
└─────────────────────────────────────────────────────────────┘
```

### **任务优先级**

任务有优先级——更高优先级的任务先获得 CPU 时间：

```
┌─────────────────────────────────────────────────────────────┐
│                    Task Priority System                    │
├─────────────────────────────────────────────────────────────┤
│                                                           │
│  Priority 5: Emergency Stop (highest)                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ██████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                           │
│  Priority 4: Safety Monitoring                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ██████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                           │
│  Priority 3: Control Loop                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ██████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                           │
│  Priority 2: Data Logging                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ██████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                           │
│  Priority 1: Status Updates (lowest)                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ██████████████████████████████████████████████████ │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                           │
│  ⚠️  Higher priority tasks can interrupt lower ones!      │
└─────────────────────────────────────────────────────────────┘
```

### **创建你的第一个任务**

这是创建简单任务的最小代码：

```c
#include "FreeRTOS.h"
#include "task.h"

// Task function - this is what the task will do
void vBlinkTask(void *pvParameters) {
    while (1) {
        // Toggle LED (your hardware-specific code here)
        toggle_led();
        
        // Wait for 500ms - FreeRTOS handles the timing
        vTaskDelay(pdMS_TO_TICKS(500));
    }
}

// In your main function
int main(void) {
    // Create the task
    xTaskCreate(
        vBlinkTask,        // Function to run
        "BlinkTask",       // Task name (for debugging)
        128,               // Stack size in words
        NULL,              // Parameters (none in this case)
        1,                 // Priority (1 = lowest)
        NULL               // Task handle (not needed here)
    );
    
    // Start the FreeRTOS scheduler
    vTaskStartScheduler();
    
    // Should never reach here
    while (1);
}
```

**关键点：**
- `vTaskDelay()` 不会阻塞 CPU——它让其他任务运行
- 栈大小(128) 对简单任务应该足够
- 优先级 1 是最低优先级
- 任务在 `while(1)` 循环中永远运行

---

## 🔗 **同步**

### **为什么任务需要同步？**

当多个任务共享资源（比如某个传感器或通信总线）时，它们需要协调：

```
┌─────────────────────────────────────────────────────────────┐
│                Problem: Resource Conflict                   │
├─────────────────────────────────────────────────────────────┤
│                                                           │
│  Task A: "I want to read the temperature sensor"          │
│  Task B: "I want to read the temperature sensor"          │
│                                                           │
│  ❌ Both try to read at the same time → corrupted data!   │
│                                                           │
│  Solution: Use a mutex (mutual exclusion)                 │
│                                                           │
│  Task A: "I'll take the mutex first"                      │
│  Task B: "I'll wait for the mutex"                        │
│                                                           │
│  ✅ Only one task can access the sensor at a time         │
└─────────────────────────────────────────────────────────────┘
```

### **使用互斥锁进行基本同步**

```c
#include "FreeRTOS.h"
#include "task.h"
#include "semphr.h"

// Mutex to protect the sensor
SemaphoreHandle_t xSensorMutex;

// Task that reads sensor
void vSensorTask(void *pvParameters) {
    while (1) {
        // Wait for mutex (wait forever if needed)
        if (xSemaphoreTake(xSensorMutex, portMAX_DELAY) == pdTRUE) {
            // We have the mutex - safe to read sensor
            float temperature = read_temperature_sensor();
            
            // Process temperature data
            process_temperature(temperature);
            
            // Give back the mutex so other tasks can use it
            xSemaphoreGive(xSensorMutex);
        }
        
        // Wait before next reading
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

// Initialize mutex
void vInitializeSystem(void) {
    // Create the mutex
    xSensorMutex = xSemaphoreCreateMutex();
    
    // Create the task
    xTaskCreate(vSensorTask, "Sensor", 128, NULL, 2, NULL);
}
```

---

## ⚙️ **配置**

### **必备配置选项**

FreeRTOS 高度可配置。以下是你需要理解的关键设置：

```c
// FreeRTOSConfig.h - Key configuration options

// Enable preemptive scheduling (tasks can interrupt each other)
#define configUSE_PREEMPTION                    1

// System tick frequency (how often FreeRTOS checks for task switches)
#define configTICK_RATE_HZ                      1000

// Maximum number of task priorities
#define configMAX_PRIORITIES                    32

// Minimum stack size for tasks
#define configMINIMAL_STACK_SIZE                128

// Enable mutex support
#define configUSE_MUTEXES                       1

// Enable queue support
#define configUSE_QUEUES                        1

// Enable semaphore support
#define configUSE_COUNTING_SEMAPHORES           1

// Check for stack overflow (important for debugging)
#define configCHECK_FOR_STACK_OVERFLOW          2
```

**这些选项的含义：**
- **抢占(Preemption)**：更高优先级的任务可以打断更低优先级的任务
- **节拍率(Tick Rate)**：FreeRTOS 做调度决策的频率
- **优先级(Priorities)**：你可以使用的优先级级别数量
- **栈大小(Stack Size)**：每个任务获得的最小内存
- **特性(Features)**：启用了哪些 FreeRTOS 特性

---

## 🧪 **引导实验**

### **实验 1：单任务系统**
**目标**：理解基本的任务创建和时序。

**设置**：创建一个以特定频率闪烁 LED 的单一任务。

**步骤**：
1. 创建一个翻转 LED 的任务函数
2. 使用 `vTaskDelay()` 来控制时序
3. 观察一致的时序行为
4. 使用示波器测量实际时序（如果可用）

**预期结果**：理解 FreeRTOS 提供可预测的时序。

### **实验 2：多任务协调**
**目标**：学习任务如何协同工作。

**设置**：创建两个任务——一个每 200ms 闪烁 LED A，另一个每 300ms 闪烁 LED B。

**步骤**：
1. 创建两个独立的任务函数
2. 给它们不同的优先级
3. 观察两者如何独立运行
4. 注意两者的时序保持一致

**预期结果**：理解多个任务可以在互不干扰的情况下并发运行。

### **实验 3：资源共享**
**目标**：学习同步和资源保护。

**设置**：创建两个需要共享资源（如 UART 或传感器）的任务。

**步骤**：
1. 创建一个互斥锁来保护共享资源
2. 让两个任务都尝试使用该资源
3. 观察互斥锁如何防止冲突
4. 测量对时序的影响

**预期结果**：理解为什么需要同步以及如何实现它。

---

## ✅ **自测**

### **理解检查**
- [ ] 你能解释为什么实时并不等于"快"吗？
- [ ] 你理解裸机方法与 RTOS 方法的区别吗？
- [ ] 你能解释什么是任务以及它与函数有何不同吗？
- [ ] 你理解为什么任务需要优先级吗？
- [ ] 你能解释什么是互斥锁以及何时使用它吗？

### **应用检查**
- [ ] 你能创建一个周期性运行的简单任务吗？
- [ ] 你知道如何正确设置任务优先级吗？
- [ ] 你能实现任务之间的基本同步吗？
- [ ] 你理解如何为你的需求配置 FreeRTOS 吗？
- [ ] 你能调试基本的任务调度问题吗？

### **分析检查**
- [ ] 你能分析何时使用 FreeRTOS 而不是裸机吗？
- [ ] 你理解不同配置选项之间的权衡吗？
- [ ] 你能识别潜在优先级反转问题吗？
- [ ] 你知道如何衡量和优化任务性能吗？
- [ ] 你能设计一个多任务系统架构吗？

---

## 🔗 **交叉链接**

### **相关主题**
- **[[Task_Creation_Management]]**：深入了解任务管理
- **[[Scheduling_Algorithms]]**：理解 FreeRTOS 如何决定何时运行什么
- **[[Interrupt_Handling]]**：FreeRTOS 如何与硬件中断协同工作
- **[[Memory_Management]]**：理解 RTOS 中的内存分配

### **进一步阅读**
- **FreeRTOS 用户手册**：官方文档和 API 参考
- **实时系统设计**：理解实时原理
- **嵌入式系统编程**：实用的嵌入式开发
- **RTOS 性能分析**：衡量和优化 RTOS 性能

### **行业标准**
- **POSIX 实时扩展**：标准实时编程接口
- **OSEK/VDX**：汽车实时操作系统标准
- **ARINC 653**：航空电子实时操作系统标准
- **IEC 61508**：电气/电子系统的功能安全

