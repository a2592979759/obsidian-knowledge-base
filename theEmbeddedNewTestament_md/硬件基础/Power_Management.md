---
tags:
  - 嵌入式
  - 电源
  - 功耗
source: "Hardware_Fundamentals/Power_Management.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入探索
>
> 把这些硬件概念整理成带参考答案的排名面试题，并配有交互式深度探索指南。
>
> 👉 **[浏览外设与硬件问题 →](https://embeddedinterviewlab.com/questions/domain/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=hardware_fundamentals)** &nbsp;·&nbsp; **[浏览外设指南 →](https://embeddedinterviewlab.com/categories/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=hardware_fundamentals)**

---

# 🔋 电源管理 (Power Management)

## 快速参考：关键事实

- **电源管理** 对电池供电的嵌入式系统和节能应用至关重要
- **电源模式** 包括活动、空闲、睡眠和深度睡眠状态，具有不同的电流消耗曲线
- **睡眠模式** 通过禁用未使用的外设和降低时钟频率来降低功耗
- **唤醒源** 包括外部中断、定时器、看门狗定时器和外设事件
- **功耗优化** 技术包括时钟门控、外设禁用和动态频率调整
- **电池管理** 涉及监控电压、电流和充电状态以实现最佳运行
- **功耗预算** 需要在不同系统状态间测量和分配功耗
- **能效** 以每次操作的微焦耳来衡量，而不仅仅是电流消耗

> **为电池供电和节能嵌入式系统优化功耗**  
> 学习实现睡眠模式、唤醒源与功耗优化技术

---

## 📋 **目录**

- [概述](#overview)
- [电源模式](#power-modes)
- [睡眠模式](#sleep-modes)
- [唤醒源](#wake-up-sources)
- [功耗优化](#power-optimization)
- [时钟管理](#clock-management)
- [外设电源管理](#peripheral-power-management)
- [电池管理](#battery-management)
- [功耗监控](#power-monitoring)
- [最佳实践](#best-practices)
- [常见陷阱](#common-pitfalls)
- [示例](#examples)
- [面试问题](#interview-questions)

---

## 🎯 **概述**

电源管理对电池供电的嵌入式系统和节能应用至关重要。有效的电源管理延长电池寿命、减少热量产生，并使便携式和 IoT 设备成为可能。

### 概念：按状态和按唤醒进行功耗预算

功耗由你的状态机掌控：定义具有已知电流和唤醒源的活动/空闲/睡眠状态。测量，而非猜测。

### 最小示例
```c
typedef enum { RUN, IDLE, SLEEP } pm_state_t;
void enter_idle(void){ /* 降低时钟，门控外设 */ }
void enter_sleep(void){ /* 无tick空闲，停止时钟，使能唤醒 */ }
```

### 要点
- 当延迟预算允许时使用无tick空闲；验证唤醒源。
- 门控未使用的时钟/外设；禁用泄漏的上拉。
- 量化每次事件的能量（每次传感器读取的微库仑）以比较设计。

---

## 🔍 可视化理解

### **电源状态转换**
```
电源状态机
┌─────────────────────────────────────────────────────────────┐
│                    电源状态转换                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │   活动      │───▶│   空闲      │───▶│   睡眠      │   │
│  │（全功率）   │    │（降低       │    │（最小       │   │
│  │             │    │ 功耗）      │    │ 功耗）      │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   │
│         ▲                   ▲                   ▲         │
│         │                   │                   │         │
│         └───────────────────┴───────────────────┘         │
│                    唤醒事件                                │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   定时器    │ │   外部      │ │   外设      │         │
│  │   中断      │ │   中断      │ │   事件      │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **功耗曲线**
```
功耗 vs. 时间
功耗 (mW)
   ^
   │    ┌─────────────────────────────────────────┐
   │    │              活动模式                   │
   │    │        （全功率运行）                   │
   │    └─────────────────────────────────────────┘
   │
   │    ┌─────────────────────────────────────────┐
   │    │               空闲模式                  │
   │    │         （降低功耗状态）                │
   │    └─────────────────────────────────────────┘
   │
   │    ┌─────────────────────────────────────────┐
   │    │               睡眠模式                  │
   │    │         （最小功耗状态）                │
   │    └─────────────────────────────────────────┘
   │
   +───────────────────────────────────────────────> 时间
   │<->│  唤醒  │<->│  活动   │<->│  睡眠   │
```

### **时钟门控与功耗降低**
```
用于功耗优化的时钟门控
┌─────────────────────────────────────────────────────────────┐
│                    时钟门控控制                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │  模块 1     │ │  模块 2     │ │  模块 3     │         │
│  │ 时钟门控    │ │ 时钟门控    │ │ 时钟门控    │         │
│  │    [开]     │ │    [关]     │ │    [开]     │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   活动      │ │   未激活    │ │   活动      │         │
│  │ （消耗      │ │ （无功耗    │ │ （消耗      │         │
│  │  功耗）     │ │  消耗）     │ │  功耗）     │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基础**

#### **电源管理挑战**
嵌入式系统中的电源管理涉及平衡性能需求与能量约束。与市电供电系统不同，电池供电设备必须仔细管理每微焦耳的能量以最大化运行寿命。

**关键特性：**
- **能量预算**：有限的能量存储需要在系统状态间仔细分配
- **动态调整**：系统必须使功耗适应当前需求
- **唤醒延迟**：在省电与响应时间之间权衡
- **状态管理**：复杂状态机管理电源模式之间的转换

#### **为什么电源管理很重要**
有效的电源管理对现代嵌入式系统至关重要：

- **电池寿命**：正确的电源管理可将电池寿命延长 10 倍或更多
- **热管理**：降低功耗减少热量产生
- **成本降低**：更低的功耗需求使电源更小更便宜
- **环境影响**：节能系统减少环境足迹

#### **功耗-性能权衡**
电源管理涉及必须仔细考虑的基本权衡：

- **活动 vs 睡眠**：更高的性能需要更多功耗，睡眠模式省电但增加延迟
- **频率 vs 效率**：更高的时钟频率提升性能但增加功耗
- **外设管理**：启用更多外设提升功能但增加功耗
- **唤醒策略**：快速的唤醒源消耗更多功耗但提供更好的响应性

## 🧪 引导实验
1) 电源状态测量
- 使用万用表或电源分析仪测量不同电源状态下的电流消耗。

2) 唤醒源测试
- 测试不同的唤醒源，并测量唤醒时间和功耗。

## ✅ 自我检查
- 你如何计算系统的总功耗预算？
- 何时应使用深度睡眠模式 vs 浅睡眠模式？

## 🔗 交叉链接
- `Hardware_Fundamentals/Clock_Management.md` 用于时钟门控
- `Hardware_Fundamentals/Watchdog_Timers.md` 用于唤醒源

### **关键概念**
- **睡眠模式** - 用于节能的不同电源状态
- **唤醒源** - 使系统脱离睡眠的事件
- **功耗优化** - 最小化功耗的技术
- **电池管理** - 监控与优化电池使用

---

## 🔄 **电源模式**

### **1. 活动模式**
完全系统运行，所有外设启用。

```c
// 活动模式配置
typedef struct {
    uint32_t cpu_frequency;      // CPU 频率（Hz）
    bool peripherals_enabled;    // 所有外设启用
    uint32_t power_consumption;  // 功耗（mW）
} active_mode_config_t;

// 活动模式功耗
void configure_active_mode(active_mode_config_t *config) {
    // 设置 CPU 频率
    set_cpu_frequency(config->cpu_frequency);
    
    // 启用所有必要外设
    if (config->peripherals_enabled) {
        enable_all_peripherals();
    }
    
    // 监控功耗
    monitor_power_consumption();
}
```

### **2. 睡眠模式**
降低功耗，部分外设禁用。

```c
// 睡眠模式配置
typedef struct {
    sleep_mode_t mode;           // 睡眠模式类型
    uint32_t wake_up_time;       // 唤醒时间（ms）
    wake_up_source_t sources;    // 唤醒源
    bool peripherals_disabled;   // 禁用未使用的外设
} sleep_mode_config_t;

// 睡眠模式类型
typedef enum {
    SLEEP_MODE_LIGHT,    // 浅睡眠 - CPU 停止，外设活动
    SLEEP_MODE_DEEP,     // 深度睡眠 - CPU 和大多数外设停止
    SLEEP_MODE_STANDBY,  // 待机 - 仅备份域活动
    SLEEP_MODE_HIBERNATE // 休眠 - 仅 RTC 活动
} sleep_mode_t;
```

### **3. 电源状态转换**

```c
// 电源状态转换
typedef enum {
    POWER_STATE_ACTIVE,
    POWER_STATE_SLEEP,
    POWER_STATE_DEEP_SLEEP,
    POWER_STATE_STANDBY
} power_state_t;

// 电源状态管理
typedef struct {
    power_state_t current_state;
    power_state_t target_state;
    uint32_t transition_time;
    bool transition_in_progress;
} power_state_manager_t;

// 转换到电源状态
void transition_to_power_state(power_state_t target_state) {
    power_state_manager_t *pm = get_power_state_manager();
    
    if (pm->current_state != target_state) {
        // 准备转换
        prepare_power_transition(target_state);
        
        // 执行转换
        execute_power_transition(target_state);
        
        // 更新状态
        pm->current_state = target_state;
    }
}
```

---

## 😴 **睡眠模式**

### **1. 浅睡眠模式**
CPU 停止，但外设和内存保持活动。

```c
// 浅睡眠模式实现
void enter_light_sleep(void) {
    // 保存当前状态
    save_system_state();
    
    // 禁用 CPU
    __WFI(); // 等待中断
    
    // 唤醒后恢复状态
    restore_system_state();
}

// 浅睡眠配置
void configure_light_sleep(void) {
    // 配置唤醒源
    configure_wake_up_sources();
    
    // 设置睡眠模式
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    // 使能睡眠模式
    __enable_irq();
}
```

### **2. 深度睡眠模式**
CPU 和大多数外设停止，仅必要功能活动。

```c
// 深度睡眠模式实现
void enter_deep_sleep(void) {
    // 保存关键数据
    save_critical_data();
    
    // 禁用未使用的外设
    disable_unused_peripherals();
    
    // 配置深度睡眠
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    // 进入深度睡眠
    __WFI();
    
    // 唤醒后恢复
    restore_critical_data();
    enable_peripherals();
}

// 深度睡眠配置
void configure_deep_sleep(void) {
    // 配置唤醒源
    configure_deep_sleep_wake_up();
    
    // 设置深度睡眠模式
    PWR->CR |= PWR_CR_LPDS;
    
    // 配置电压调整
    PWR->CR |= PWR_CR_VOS;
}
```

### **3. 待机模式**
仅备份域和 RTC 活动，所有其他功能停止。

```c
// 待机模式实现
void enter_standby_mode(void) {
    // 将关键数据保存到备份寄存器
    save_to_backup_registers();
    
    // 配置待机模式
    PWR->CR |= PWR_CR_CWUF;
    PWR->CR |= PWR_CR_PDDS;
    
    // 进入待机
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    __WFI();
    
    // 唤醒后系统将复位
}

// 待机模式配置
void configure_standby_mode(void) {
    // 配置 RTC 作为唤醒源
    configure_rtc_wake_up();
    
    // 使能备份域
    RCC->APB1ENR |= RCC_APB1ENR_PWREN;
    PWR->CR |= PWR_CR_DBP;
}
```

---

## 🔔 **唤醒源**

### **1. 外部中断**

```c
// 外部中断唤醒配置
typedef struct {
    uint8_t pin;
    edge_type_t edge;
    bool enabled;
} external_wake_up_config_t;

// 配置外部唤醒
void configure_external_wake_up(external_wake_up_config_t *config) {
    // 将 GPIO 配置为输入
    configure_gpio_input(config->pin);
    
    // 配置中断
    configure_external_interrupt(config->pin, config->edge);
    
    // 使能唤醒能力
    EXTI->IMR |= (1 << config->pin);
    
    // 在 NVIC 中使能
    NVIC_EnableIRQ(EXTI0_IRQn + config->pin);
}

// 外部唤醒处理程序
void external_wake_up_handler(void) {
    // 清除唤醒标志
    PWR->CR |= PWR_CR_CWUF;
    
    // 处理唤醒事件
    process_wake_up_event();
}
```

### **2. 定时器唤醒**

```c
// 定时器唤醒配置
typedef struct {
    uint32_t wake_up_time_ms;
    timer_type_t timer_type;
    bool enabled;
} timer_wake_up_config_t;

// 配置定时器唤醒
void configure_timer_wake_up(timer_wake_up_config_t *config) {
    // 配置定时器
    configure_timer(config->timer_type, config->wake_up_time_ms);
    
    // 使能定时器中断
    enable_timer_interrupt(config->timer_type);
    
    // 配置为唤醒源
    configure_timer_wake_up_source(config->timer_type);
}

// 定时器唤醒处理程序
void timer_wake_up_handler(void) {
    // 清除定时器中断
    clear_timer_interrupt();
    
    // 处理定时器唤醒
    process_timer_wake_up();
}
```

### **3. RTC 唤醒**

```c
// RTC 唤醒配置
typedef struct {
    uint32_t wake_up_time;
    rtc_wake_up_source_t source;
    bool enabled;
} rtc_wake_up_config_t;

// 配置 RTC 唤醒
void configure_rtc_wake_up(rtc_wake_up_config_t *config) {
    // 配置 RTC
    configure_rtc();
    
    // 设置唤醒时间
    set_rtc_wake_up_time(config->wake_up_time);
    
    // 使能 RTC 唤醒
    RTC->CR |= RTC_CR_WUTE;
    
    // 使能 RTC 中断
    NVIC_EnableIRQ(RTC_IRQn);
}

// RTC 唤醒处理程序
void rtc_wake_up_handler(void) {
    // 清除 RTC 唤醒标志
    RTC->ISR &= ~RTC_ISR_WUTF;
    
    // 处理 RTC 唤醒
    process_rtc_wake_up();
}
```

---

## ⚡ **功耗优化**

### **1. CPU 功耗优化**

```c
// CPU 功耗优化
typedef struct {
    uint32_t frequency;
    voltage_scale_t voltage;
    bool dynamic_scaling;
} cpu_power_config_t;

// 配置 CPU 功耗
void configure_cpu_power(cpu_power_config_t *config) {
    // 设置电压调整
    set_voltage_scaling(config->voltage);
    
    // 设置 CPU 频率
    set_cpu_frequency(config->frequency);
    
    // 使能动态频率调整
    if (config->dynamic_scaling) {
        enable_dynamic_frequency_scaling();
    }
}

// 动态频率调整
void enable_dynamic_frequency_scaling(void) {
    // 监控 CPU 负载
    uint32_t cpu_load = get_cpu_load();
    
    if (cpu_load < 30) {
        // 低负载降低频率
        set_cpu_frequency(CPU_FREQ_LOW);
    } else if (cpu_load > 80) {
        // 高负载提高频率
        set_cpu_frequency(CPU_FREQ_HIGH);
    }
}
```

### **2. 外设功耗优化**

```c
// 外设电源管理
typedef struct {
    peripheral_type_t peripheral;
    bool enabled;
    uint32_t power_consumption;
} peripheral_power_config_t;

// 禁用未使用的外设
void disable_unused_peripherals(void) {
    // 禁用未使用的 UART
    if (!uart1_used) {
        disable_peripheral(UART1);
    }
    
    // 禁用未使用的定时器
    if (!timer1_used) {
        disable_peripheral(TIM1);
    }
    
    // 禁用未使用的 ADC
    if (!adc1_used) {
        disable_peripheral(ADC1);
    }
}

// 仅在需要时启用外设
void enable_peripheral_on_demand(peripheral_type_t peripheral) {
    // 启用外设
    enable_peripheral(peripheral);
    
    // 使用外设
    use_peripheral(peripheral);
    
    // 使用后禁用外设
    disable_peripheral(peripheral);
}
```

### **3. 内存功耗优化**

```c
// 内存功耗优化
typedef struct {
    bool flash_power_down;
    bool sram_retention;
    bool cache_enabled;
} memory_power_config_t;

// 配置内存功耗
void configure_memory_power(memory_power_config_t *config) {
    if (config->flash_power_down) {
        // 不使用 Flash 时断电
        power_down_flash();
    }
    
    if (config->sram_retention) {
        // 在睡眠模式下使能 SRAM 保持
        enable_sram_retention();
    }
    
    if (config->cache_enabled) {
        // 使能缓存以获得更好性能
        enable_cache();
    }
}
```

---

## ⏰ **时钟管理**

### **1. 时钟配置**

```c
// 时钟配置
typedef struct {
    uint32_t system_clock;
    uint32_t peripheral_clock;
    bool pll_enabled;
    clock_source_t source;
} clock_config_t;

// 配置系统时钟
void configure_system_clock(clock_config_t *config) {
    // 如使能则配置 PLL
    if (config->pll_enabled) {
        configure_pll(config->system_clock);
    }
    
    // 设置系统时钟
    set_system_clock(config->system_clock);
    
    // 配置外设时钟
    configure_peripheral_clocks(config->peripheral_clock);
}

// 动态时钟调整
void dynamic_clock_scaling(void) {
    uint32_t current_load = get_system_load();
    
    if (current_load < 20) {
        // 低负载 - 降低时钟频率
        set_system_clock(SYSTEM_CLOCK_LOW);
    } else if (current_load > 80) {
        // 高负载 - 提高时钟频率
        set_system_clock(SYSTEM_CLOCK_HIGH);
    }
}
```

### **2. 时钟门控**

```c
// 门控时钟以省电
void enable_clock_gating(void) {
    // 门控未使用的外设时钟
    RCC->AHB1ENR &= ~(RCC_AHB1ENR_GPIOAEN | RCC_AHB1ENR_GPIOBEN);
    RCC->APB1ENR &= ~(RCC_APB1ENR_USART2EN | RCC_APB1ENR_TIM3EN);
    RCC->APB2ENR &= ~(RCC_APB2ENR_USART1EN | RCC_APB2ENR_TIM1EN);
}

// 仅在需要时使能时钟
void enable_peripheral_clock(peripheral_type_t peripheral) {
    switch (peripheral) {
        case PERIPHERAL_UART1:
            RCC->APB2ENR |= RCC_APB2ENR_USART1EN;
            break;
        case PERIPHERAL_TIM1:
            RCC->APB2ENR |= RCC_APB2ENR_TIM1EN;
            break;
        case PERIPHERAL_ADC1:
            RCC->APB2ENR |= RCC_APB2ENR_ADC1EN;
            break;
    }
}
```

---

## 🔋 **电池管理**

### **1. 电池监控**

```c
// 电池监控
typedef struct {
    uint32_t voltage;
    uint32_t capacity;
    uint32_t remaining;
    battery_status_t status;
} battery_info_t;

// 电池状态
typedef enum {
    BATTERY_STATUS_UNKNOWN,
    BATTERY_STATUS_CHARGING,
    BATTERY_STATUS_DISCHARGING,
    BATTERY_STATUS_FULL,
    BATTERY_STATUS_LOW,
    BATTERY_STATUS_CRITICAL
} battery_status_t;

// 监控电池
void monitor_battery(void) {
    battery_info_t battery;
    
    // 读取电池电压
    battery.voltage = read_battery_voltage();
    
    // 计算剩余容量
    battery.remaining = calculate_battery_capacity(battery.voltage);
    
    // 更新电池状态
    battery.status = get_battery_status(battery.voltage);
    
    // 处理低电量
    if (battery.status == BATTERY_STATUS_CRITICAL) {
        handle_critical_battery();
    }
}
```

### **2. 电池优化**

```c
// 电池优化策略
void optimize_for_battery_life(void) {
    // 降低 CPU 频率
    set_cpu_frequency(CPU_FREQ_LOW);
    
    // 禁用未使用的外设
    disable_unused_peripherals();
    
    // 使能睡眠模式
    enable_sleep_modes();
    
    // 优化通信
    optimize_communication_power();
    
    // 降低传感器采样率
    reduce_sensor_sampling_rate();
}

// 关键电量处理
void handle_critical_battery(void) {
    // 保存关键数据
    save_critical_data();
    
    // 进入深度睡眠模式
    enter_deep_sleep();
    
    // 仅配置关键事件的唤醒
    configure_critical_wake_up_sources();
}
```

---

## 📊 **功耗监控**

### **1. 功耗监控**

```c
// 功耗监控
typedef struct {
    uint32_t current_consumption;
    uint32_t average_consumption;
    uint32_t peak_consumption;
    uint32_t total_energy;
} power_consumption_t;

// 监控功耗
void monitor_power_consumption(void) {
    power_consumption_t power;
    
    // 读取当前消耗
    power.current_consumption = read_current_consumption();
    
    // 更新平均消耗
    update_average_consumption(power.current_consumption);
    
    // 检查峰值消耗
    if (power.current_consumption > power.peak_consumption) {
        power.peak_consumption = power.current_consumption;
    }
    
    // 计算总能量
    power.total_energy += power.current_consumption;
    
    // 记录功耗
    log_power_consumption(&power);
}
```

### **2. 功耗剖析**

```c
// 功耗剖析
typedef struct {
    uint32_t timestamp;
    power_state_t state;
    uint32_t consumption;
    uint32_t duration;
} power_profile_entry_t;

// 功耗剖析
void profile_power_consumption(void) {
    static power_profile_entry_t profile[MAX_PROFILE_ENTRIES];
    static uint8_t profile_index = 0;
    
    // 记录功耗剖析条目
    profile[profile_index].timestamp = get_system_tick();
    profile[profile_index].state = get_current_power_state();
    profile[profile_index].consumption = read_current_consumption();
    profile[profile_index].duration = calculate_duration();
    
    // 增加索引
    profile_index = (profile_index + 1) % MAX_PROFILE_ENTRIES;
    
    // 分析功耗剖析
    analyze_power_profile(profile, profile_index);
}
```

---

## 🎯 **最佳实践**

### **1. 电源管理指南**

```c
// 电源管理清单
/*
    □ 使用适当的睡眠模式
    □ 禁用未使用的外设
    □ 优化时钟频率
    □ 实现动态电源调整
    □ 监控功耗
    □ 处理电池管理
    □ 使用高效的唤醒源
    □ 优化通信协议
    □ 实现功耗感知调度
    □ 测试功耗
*/

// 好的电源管理示例
void good_power_management(void) {
    // 配置电源管理
    configure_power_management();
    
    // 带功耗优化的主循环
    while (1) {
        // 处理任务
        process_tasks();
        
        // 检查系统是否可以睡眠
        if (can_enter_sleep_mode()) {
            // 进入睡眠模式
            enter_sleep_mode();
        }
        
        // 监控功耗
        monitor_power_consumption();
    }
}
```

### **2. 睡眠模式指南**

```c
// 睡眠模式清单
/*
    □ 选择适当的睡眠模式
    □ 配置唤醒源
    □ 保存关键数据
    □ 禁用未使用的外设
    □ 处理唤醒事件
    □ 恢复系统状态
    □ 监控睡眠持续时间
    □ 测试睡眠模式功能
    □ 记录睡眠行为
    □ 考虑安全需求
*/

// 好的睡眠模式实现
void good_sleep_mode(void) {
    // 保存系统状态
    save_system_state();
    
    // 配置唤醒源
    configure_wake_up_sources();
    
    // 进入睡眠模式
    enter_sleep_mode();
    
    // 唤醒后恢复系统状态
    restore_system_state();
    
    // 处理唤醒事件
    process_wake_up_events();
}
```

---

## ⚠️ **常见陷阱**

### **1. 睡眠模式使用不当**

```c
// 错误：不使用睡眠模式
void bad_no_sleep(void) {
    while (1) {
        // 处理任务
        process_tasks();
        
        // 始终活动 - 浪费功耗
        delay_ms(100);
    }
}

// 正确：使用睡眠模式
void good_sleep_usage(void) {
    while (1) {
        // 处理任务
        process_tasks();
        
        // 空闲时进入睡眠模式
        if (is_system_idle()) {
            enter_sleep_mode();
        }
    }
}
```

### **2. 未使用的外设**

```c
// 错误：不禁用未使用的外设
void bad_unused_peripherals(void) {
    // 启用所有外设
    enable_all_peripherals();
    
    // 仅使用部分外设
    use_some_peripherals();
    
    // 保留未使用的外设启用
}

// 正确：禁用未使用的外设
void good_peripheral_management(void) {
    // 仅启用需要的外设
    enable_needed_peripherals();
    
    // 使用外设
    use_peripherals();
    
    // 不需要时禁用
    disable_unused_peripherals();
}
```

### **3. 时钟管理不当**

```c
// 错误：固定高频率
void bad_fixed_clock(void) {
    // 始终使用高频率
    set_cpu_frequency(CPU_FREQ_HIGH);
    
    // 处理任务
    process_tasks();
}

// 正确：动态时钟调整
void good_clock_management(void) {
    // 根据负载调整时钟
    uint32_t load = get_cpu_load();
    
    if (load < 30) {
        set_cpu_frequency(CPU_FREQ_LOW);
    } else if (load > 80) {
        set_cpu_frequency(CPU_FREQ_HIGH);
    }
    
    // 处理任务
    process_tasks();
}
```

---

## 💡 **示例**

### **1. 简单电源管理**

```c
// 简单电源管理实现
void simple_power_management(void) {
    // 配置电源管理
    configure_power_management();
    
    while (1) {
        // 处理应用任务
        process_application_tasks();
        
        // 检查系统是否可以睡眠
        if (is_system_idle()) {
            // 进入浅睡眠
            enter_light_sleep();
        }
        
        // 监控电池
        monitor_battery();
    }
}

// 系统空闲检查
bool is_system_idle(void) {
    // 检查是否没有待处理任务
    if (task_queue_empty() && !communication_pending()) {
        return true;
    }
    
    return false;
}
```

### **2. 高级电源管理**

```c
// 带多个模式的高级电源管理
typedef enum {
    POWER_MODE_ACTIVE,
    POWER_MODE_LIGHT_SLEEP,
    POWER_MODE_DEEP_SLEEP,
    POWER_MODE_STANDBY
} power_mode_t;

void advanced_power_management(void) {
    power_mode_t current_mode = POWER_MODE_ACTIVE;
    
    while (1) {
        // 确定最佳电源模式
        power_mode_t optimal_mode = determine_optimal_power_mode();
        
        // 转换到最佳模式
        if (optimal_mode != current_mode) {
            transition_to_power_mode(optimal_mode);
            current_mode = optimal_mode;
        }
        
        // 根据模式处理任务
        switch (current_mode) {
            case POWER_MODE_ACTIVE:
                process_active_tasks();
                break;
            case POWER_MODE_LIGHT_SLEEP:
                process_light_sleep_tasks();
                break;
            case POWER_MODE_DEEP_SLEEP:
                process_deep_sleep_tasks();
                break;
            case POWER_MODE_STANDBY:
                process_standby_tasks();
                break;
        }
    }
}

// 确定最佳电源模式
power_mode_t determine_optimal_power_mode(void) {
    uint32_t battery_level = get_battery_level();
    uint32_t system_load = get_system_load();
    
    if (battery_level < 20) {
        return POWER_MODE_DEEP_SLEEP;
    } else if (system_load < 10) {
        return POWER_MODE_LIGHT_SLEEP;
    } else {
        return POWER_MODE_ACTIVE;
    }
}
```

### **3. 电池优化系统**

```c
// 电池优化系统
void battery_optimized_system(void) {
    // 配置电池运行
    configure_battery_optimization();
    
    while (1) {
        // 监控电池电量
        uint32_t battery_level = get_battery_level();
        
        if (battery_level < 10) {
            // 关键电量 - 进入深度睡眠
            enter_critical_battery_mode();
        } else if (battery_level < 30) {
            // 低电量 - 降低功耗
            enter_low_battery_mode();
        } else {
            // 正常电量 - 标准运行
            enter_normal_mode();
        }
        
        // 根据电池电量处理任务
        process_tasks_based_on_battery(battery_level);
    }
}

// 关键电量模式
void enter_critical_battery_mode(void) {
    // 禁用非必要外设
    disable_non_essential_peripherals();
    
    // 降低 CPU 频率
    set_cpu_frequency(CPU_FREQ_MIN);
    
    // 以最小唤醒源进入深度睡眠
    configure_minimal_wake_up_sources();
    enter_deep_sleep();
}
```

---

## 🎯 **面试问题**

### **基本问题**
1. **嵌入式系统有哪些不同的睡眠模式？**
   - 浅睡眠：CPU 停止，外设活动
   - 深度睡眠：CPU 和大多数外设停止
   - 待机：仅备份域活动
   - 休眠：仅 RTC 活动

2. **你如何在嵌入式系统中实现电源管理？**
   - 使用适当的睡眠模式
   - 禁用未使用的外设
   - 优化时钟频率
   - 监控功耗

3. **常见的唤醒源有哪些？**
   - 外部中断
   - 定时器中断
   - RTC 闹钟
   - 通信事件

### **高级问题**
4. **你如何为电池供电设备优化功耗？**
   - 实现动态电源调整
   - 使用高效的睡眠模式
   - 优化通信协议
   - 监控和管理电池使用

5. **电源管理有哪些权衡？**
   - 性能 vs 功耗
   - 响应时间 vs 睡眠时长
   - 功能性 vs 电池寿命
   - 成本 vs 电源效率

6. **你如何在实时系统中处理电源管理？**
   - 确保满足时序需求
   - 使用适当的唤醒源
   - 平衡省电与响应性
   - 彻底测试电源管理

### **实践问题**
7. **为 IoT 设备设计电源管理系统。**
   ```c
   void iot_power_management(void) {
       // 为 IoT 运行配置
       configure_iot_power_management();
       
       while (1) {
           // 处理 IoT 任务
           process_iot_tasks();
           
           // 检查通信需求
           if (communication_needed()) {
               enable_communication();
               send_data();
               disable_communication();
           }
           
           // 进入睡眠模式
           enter_sleep_mode();
       }
   }
   ```

8. **实现电池监控系统。**
   ```c
   void battery_monitoring_system(void) {
       // 配置电池监控
       configure_battery_monitoring();
       
       while (1) {
           // 监控电池
           uint32_t battery_level = get_battery_level();
           
           // 处理不同电池电量
           if (battery_level < 10) {
               handle_critical_battery();
           } else if (battery_level < 30) {
               handle_low_battery();
           }
           
           // 记录电池状态
           log_battery_status(battery_level);
           
           // 按监控间隔睡眠
           delay_ms(BATTERY_MONITOR_INTERVAL);
       }
   }
   ```

---

## 🔗 **相关主题**

- **[[External_Interrupts]]** - 边沿/电平触发中断、去抖动
- **[[Watchdog_Timers]]** - 系统监控与恢复机制
- **[[Interrupts_Exceptions]]** - 中断处理、ISR 设计、中断延迟
- **[[Clock_Management]]** - 系统时钟配置、PLL 设置

---

## 📚 **资源**

### **书籍**
- 《Making Embedded Systems》 Elecia White 著
- 《Programming Embedded Systems》 Michael Barr 著
- 《Real-Time Systems》 Jane W. S. Liu 著

### **在线资源**
- [ARM Cortex-M 电源管理](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/power-management)
- [STM32 电源管理](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)

---

**下一个主题：** [[Clock_Management]] → [[Reset_Management]]
