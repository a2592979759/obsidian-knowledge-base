---
tags:
  - 嵌入式
  - 定时器
  - 计时器
source: "Hardware_Fundamentals/Timer_Counter_Programming.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入探索
>
> 把这些硬件概念整理成带参考答案的排名面试题，并配有交互式深度探索指南。
>
> 👉 **[浏览外设与硬件问题 →](https://embeddedinterviewlab.com/questions/domain/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=hardware_fundamentals)** &nbsp;·&nbsp; **[阅读深度指南 →](https://embeddedinterviewlab.com/topics/timers-pwm?utm_source=github&utm_medium=referral&utm_campaign=kb_topic&utm_content=hardware_fundamentals)**

---

# ⏱️ 定时器/计数器编程 (Timer/Counter Programming)

> **精通嵌入式系统的定时器与计数器操作**  
> 输入捕获、输出比较、频率测量与时序应用

---

## 📋 **目录**

- [概述](#overview)
- [快速参考：关键事实](#quick-reference-key-facts)
- [可视化理解](#visual-understanding)
- [概念基础](#conceptual-foundation)
- [核心概念](#core-concepts)
- [实践考量](#practical-considerations)
- [其他资源](#additional-resources)

---

## 🎯 **概述**

定时器和计数器是嵌入式系统中用于精确时序、频率测量、PWM 生成和事件计数的必备外设。理解定时器编程对实时应用至关重要。

---

## 🚀 **快速参考：关键事实**

- **定时器模式**：向上计数、向下计数、中心对齐
- **输入捕获**：边沿检测、频率测量、脉宽测量
- **输出比较**：PWM 生成、时序控制、波形生成
- **预分频**：分频时钟频率以实现所需时序分辨率
- **自动重载**：自动重载计数器值以连续运行
- **中断**：用于精确时序事件的定时器中断
- **DMA 集成**：无需 CPU 干预的高速数据传输

### **面试官意图（他们在考察什么）**
- 你能从时钟和预分频计算定时器频率/周期吗？
- 你理解捕获 vs 比较的用例吗？
- 你能推理抖动、ISR 成本和 DMA 权衡吗？

---

## 🔍 **可视化理解**

### **定时器框图**
```
时钟源 → 预分频 → 计数器 → 比较/输出 → 中断/DMA
     ↓            ↓         ↓           ↓              ↓
系统时钟    除以计数值   向上/向下/  生成           触发
(84 MHz)    PSC+1       中心         PWM/事件      CPU 事件
```

### **定时器模式可视化**
```
向上计数：    0 → 1 → 2 → ... → ARR → 0 → 1 → ...
向下计数：    ARR → ARR-1 → ... → 1 → 0 → ARR → ...
中心对齐：    0 → 1 → ... → ARR → ARR-1 → ... → 1 → 0 → ...
```

### **输入捕获 vs 输出比较**
```
输入捕获：    外部事件 → 捕获定时器值 → 计算时序
                     ↓                ↓                ↓
                GPIO 边沿        存储计数值      频率/脉冲
                                在寄存器中       宽度

输出比较：    定时器计数 → 与 CCR 比较 → 生成输出事件
                     ↓            ↓              ↓
                递增          匹配值        PWM/中断
                计数器        (CCR 寄存器)   生成
```

---

## 🧠 **概念基础**

### **定时器作为时间基础**
定时器是嵌入式系统中所有与时间相关操作的基本构建块。它们提供：
- **精确时序**：基于硬件、抖动最小化的时序
- **事件调度**：周期任务的可预测执行
- **测量能力**：准确的频率和时间间隔测量
- **波形生成**：创建精确的时序模式和 PWM 信号

### **为什么定时器编程很重要**
定时器编程至关重要，因为：
- **实时需求**：许多嵌入式应用需要精确时序
- **系统同步**：协调多个系统事件和外设
- **功耗效率**：定时器支持睡眠模式和唤醒时序
- **性能优化**：硬件定时器将时序任务从 CPU 卸载

### **定时器设计挑战**
设计定时器系统需要平衡几个相互竞争的问题：
- **分辨率 vs 范围**：更高的分辨率（更小的预分频）降低最大周期
- **精度 vs 复杂度**：更精确的时序需要仔细配置
- **硬件 vs 软件**：硬件定时器 vs 软件时序循环
- **中断频率**：平衡时序精度与系统开销

---

## 🎯 **核心概念**

### **概念：定时器配置与频率计算**

**为什么重要**：正确的定时器配置确保准确时序并防止溢出错误。理解时钟频率、预分频和周期之间的关系对于实现所需的时序分辨率和范围至关重要。

**最小示例**
```c
// 基本定时器配置结构
typedef struct {
    uint32_t prescaler;      // 定时器预分频值
    uint32_t period;         // 定时器周期 (ARR 值)
    uint32_t clock_freq;     // 定时器时钟频率
    uint32_t mode;           // 定时器模式 (UP, DOWN, CENTER)
} timer_config_t;

// 计算定时器频率
uint32_t calculate_timer_frequency(uint32_t clock_freq, uint32_t prescaler, uint32_t period) {
    return clock_freq / ((prescaler + 1) * (period + 1));
}

// 计算目标频率的定时器周期
uint32_t calculate_timer_period(uint32_t clock_freq, uint32_t prescaler, uint32_t target_freq) {
    return (clock_freq / ((prescaler + 1) * target_freq)) - 1;
}
```

**试一下**：为使用 84MHz 时钟的 1kHz 定时器计算所需的预分频和周期值。

**要点**
- 预分频分频输入时钟频率
- 周期决定定时器溢出频率
- 更高的预分频值提供更长的时序周期
- 始终验证计算以防止溢出

### **概念：用于精确时序测量的输入捕获**

**为什么重要**：输入捕获能够精确测量外部事件，使其对嵌入式系统中的频率测量、脉宽分析和事件时序至关重要。

**最小示例**
```c
// 输入捕获配置
typedef struct {
    uint32_t channel;        // 定时器通道 (1-4)
    uint32_t edge;           // 上升/下降沿
    uint32_t filter;         // 输入滤波值
    bool interrupt_enable;   // 使能捕获中断
} input_capture_config_t;

// 配置输入捕获
void configure_input_capture(TIM_HandleTypeDef* htim, input_capture_config_t* config) {
    // 配置通道为输入
    TIM_IC_InitTypeDef sConfigIC = {0};
    sConfigIC.ICPolarity = config->edge;
    sConfigIC.ICSelection = TIM_ICSELECTION_DIRECTTI;
    sConfigIC.ICPrescaler = TIM_ICPSC_DIV1;
    sConfigIC.ICFilter = config->filter;
    
    HAL_TIM_IC_ConfigChannel(htim, &sConfigIC, config->channel);
    
    // 如请求则使能中断
    if (config->interrupt_enable) {
        __HAL_TIM_ENABLE_IT(htim, TIM_IT_CC1 << (config->channel - 1));
    }
}
```

**试一下**：实现输入捕获来测量外部信号的频率。

**要点**
- 输入捕获在外部事件发生时存储定时器值
- 边沿选择影响测量精度
- 输入滤波降低噪声敏感性
- 中断支持实时事件处理

### **概念：用于精确事件生成的输出比较**

**为什么重要**：输出比较能够精确生成时序事件、PWM 信号和周期输出。它对电机控制、音频生成和时序同步至关重要。

**最小示例**
```c
// 输出比较配置
typedef struct {
    uint32_t channel;        // 定时器通道 (1-4)
    uint32_t compare_value;  // 比较值 (CCR)
    uint32_t mode;           // 输出模式 (Toggle, PWM, Force)
    bool interrupt_enable;   // 使能比较中断
} output_compare_config_t;

// 配置输出比较
void configure_output_compare(TIM_HandleTypeDef* htim, output_compare_config_t* config) {
    // 配置通道为输出
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = config->mode;
    sConfigOC.Pulse = config->compare_value;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_OC_ConfigChannel(htim, &sConfigOC, config->channel);
    
    // 如请求则使能中断
    if (config->interrupt_enable) {
        __HAL_TIM_ENABLE_IT(htim, TIM_IT_CC1 << (config->channel - 1));
    }
}
```

**试一下**：使用输出比较生成一个 50% 占空比的 1kHz 方波。

**要点**
- 输出比较在定时器匹配比较值时生成事件
- 比较值决定输出事件的时序
- 多个通道支持复杂时序模式
- 中断提供精确的事件通知

### **概念：定时器中断与 DMA 集成**

**为什么重要**：定时器中断和 DMA 集成能够高效处理时序事件和高速数据传输，减少 CPU 开销并提升系统性能。

**最小示例**
```c
// 定时器中断配置
void configure_timer_interrupt(TIM_HandleTypeDef* htim, uint32_t priority) {
    // 使能定时器更新中断
    __HAL_TIM_ENABLE_IT(htim, TIM_IT_UPDATE);
    
    // 配置 NVIC 优先级
    HAL_NVIC_SetPriority(TIM2_IRQn, priority, 0);
    HAL_NVIC_EnableIRQ(TIM2_IRQn);
}

// 定时器中断处理程序
void TIM2_IRQHandler(void) {
    if (__HAL_TIM_GET_FLAG(&htim2, TIM_FLAG_UPDATE) != RESET) {
        if (__HAL_TIM_GET_IT_SOURCE(&htim2, TIM_IT_UPDATE) != RESET) {
            __HAL_TIM_CLEAR_IT(&htim2, TIM_IT_UPDATE);
            
            // 处理定时器事件
            handle_timer_event();
        }
    }
}
```

**试一下**：实现一个每 500ms 翻转 LED 的定时器中断。

**要点**
- 定时器中断为事件处理提供精确时序
- 保持中断处理程序简短高效
- 使用 DMA 进行高速数据传输
- 正确的优先级配置防止时序冲突

---

## 🧪 **引导实验**

### **实验 1：基本定时器配置与中断**
**目标**：配置定时器生成周期中断，并测量时序精度。

**步骤**：
1. 配置定时器以 1kHz 运行
2. 使能定时器中断
3. 在中断处理程序中翻转 GPIO 引脚
4. 用示波器测量时序精度

**预期结果**：理解定时器配置与中断时序。

### **实验 2：用于频率测量的输入捕获**
**目标**：使用输入捕获测量外部信号的频率。

**步骤**：
1. 配置定时器通道用于输入捕获
2. 用函数发生器生成测试信号
3. 实现频率计算算法
4. 比较测量值与实际频率

**预期结果**：获得输入捕获与频率测量的实践经验。

### **实验 3：用于 PWM 生成的输出比较**
**目标**：使用输出比较生成可变占空比的 PWM 信号。

**步骤**：
1. 配置定时器用于 PWM 模式
2. 实现占空比控制
3. 生成不同的 PWM 频率
4. 用示波器测量 PWM 特性

**预期结果**：理解 PWM 生成与输出比较操作。

---

## ✅ **自我检查**

### **基本理解**
- 定时器和计数器模式有什么区别？
- 你如何从时钟、预分频和周期计算定时器频率？
- 输入捕获和输出比较的主要应用有哪些？

### **实际应用**
- 你会如何配置一个 100Hz、1ms 分辨率的定时器？
- 选择定时器预分频值时有哪些重要考量？
- 你如何使用定时器实现精确时序测量？

### **高级概念**
- 你如何在长时测量中处理定时器溢出？
- 硬件和软件时序之间有什么权衡？
- 你如何为复杂应用同步多个定时器？

---

## 🔗 **交叉链接**

- **[[GPIO_Configuration]]** - GPIO 模式、配置、电气特性
- **[[Pulse_Width_Modulation]]** - PWM 生成、频率控制、占空比
- **[[External_Interrupts]]** - 边沿/电平触发中断、去抖动
- **[[Interrupts_Exceptions]]** - 中断处理与 ISR 设计
- **[[Hardware_Abstraction_Layer]]** - 定时器抽象与可移植性

---

## 🎯 **实践考量**

### **系统级设计决策**
- **定时器选择**：基于分辨率和范围需求选择适当的定时器
- **中断频率**：平衡时序精度与系统开销
- **资源分配**：考虑多个应用间的定时器共享

### **性能与优化**
- **预分频选择**：优化以实现所需时序分辨率和范围
- **中断效率**：最小化 ISR 执行时间
- **DMA 使用**：为高速定时器应用使用 DMA

### **调试与测试**
- **时序验证**：使用示波器或逻辑分析仪验证时序
- **中断调试**：监控中断时序和频率
- **性能剖析**：测量定时器精度和抖动

---

## 📚 **其他资源**

### **文档**
- [STM32 定时器参考手册](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)
- [ARM Cortex-M 定时器编程](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/general-purpose-timers)

### **工具**
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) - 定时器配置
- [定时器计算器](https://www.st.com/resource/en/user_manual/dm00104712-stm32cubemx-user-manual-stmicroelectronics.pdf)

### **书籍**
- 《Embedded Systems: Introduction to ARM Cortex-M Microcontrollers》 Jonathan Valvano 著
- 《Making Embedded Systems》 Elecia White 著

---

**下一个主题：** [[Watchdog_Timers]] → [[Interrupts_Exceptions]]
