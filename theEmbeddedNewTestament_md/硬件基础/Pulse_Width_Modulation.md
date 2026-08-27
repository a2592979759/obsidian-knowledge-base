---
tags:
  - 嵌入式
  - PWM
  - 定时器
source: "Hardware_Fundamentals/Pulse_Width_Modulation.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入探索
>
> 把这些硬件概念整理成带参考答案的排名面试题，并配有交互式深度探索指南。
>
> 👉 **[浏览外设与硬件问题 →](https://embeddedinterviewlab.com/questions/domain/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=hardware_fundamentals)** &nbsp;·&nbsp; **[阅读深度指南 →](https://embeddedinterviewlab.com/topics/timers-pwm?utm_source=github&utm_medium=referral&utm_campaign=kb_topic&utm_content=hardware_fundamentals)**

---

# ⏱️ 脉宽调制 (Pulse Width Modulation, PWM)

## 快速参考：关键事实

- **脉宽调制（PWM）** 通过在开/关状态之间快速切换来控制功率输出
- **占空比** 是信号保持高电平的时间百分比，控制平均功率输出
- **频率** 决定切换速率，并影响效率、噪声和分辨率
- **分辨率** 是离散占空比电平的数量，与频率成反比
- **定时器硬件** 使用比较寄存器和输出比较模式生成 PWM
- **应用** 包括电机控制、LED 调光、电源和音频生成
- **滤波** 可将 PWM 转换为模拟信号，有效创建 DAC
- **权衡** 存在于频率（噪声）、分辨率（精度）和效率之间

> **精通嵌入式系统的 PWM**  
> PWM 生成、频率控制、占空比与实际应用

## 📋 目录

- [🎯 概述](#-overview)
- [🔧 PWM 基础](#-pwm-fundamentals)
- [⚙️ PWM 配置](#️-pwm-configuration)
- [🎛️ 占空比控制](#️-duty-cycle-control)
- [🔄 频率控制](#-frequency-control)
- [📊 PWM 应用](#-pwm-applications)
- [⚡ 高级 PWM 技术](#-advanced-pwm-techniques)
- [🎯 常见应用](#-common-applications)
- [⚠️ 常见陷阱](#️-common-pitfalls)
- [✅ 最佳实践](#-best-practices)
- [🎯 面试问题](#-interview-questions)
- [📚 其他资源](#-additional-resources)

---

## 🎯 概述

### 概念：占空比、频率和分辨率相互耦合

PWM 是定时器比较硬件。你对周期（ARR）和时钟预分频的选择决定了频率和占空比分辨率。将这些匹配到执行器需求和 EMC 约束。

### 最小示例
```c
// 若时钟允许，将 PWM 设置为 20 kHz、10 位分辨率
void pwm_init(void){ /* 设置 PSC/ARR；配置 CCx 模式；使能输出 */ }
void pwm_set_duty(uint16_t duty){ /* 写 CCRx，钳位到 ARR */ }
```

### 试一下
1. 扫频以将开关噪声移出敏感频段。
2. 评估电机/LED 上线性度 vs 死区/驱动器延迟。

### 要点
- 频率 vs 分辨率权衡由定时器时钟决定。
- 对半桥使用互补输出和死区。
- 滤波后的 PWM（RC）行为类似 DAC；设计该滤波器。

### **面试官意图（他们在考察什么）**
- 你能根据定时器设置计算 PWM 频率/分辨率吗？
- 你理解权衡：噪声 vs 分辨率 vs 效率吗？
- 你能解释实际的执行器约束（死区、驱动器限制）吗？

---

## 🔍 可视化理解

### **PWM 信号特性**
```
PWM 信号参数
┌─────────────────────────────────────────────────────────────┐
│                    PWM 信号分析                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              高频 PWM                               │   │
│  │  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │   │
│  │  │<->│ 周期 │<->│ 周期 │<->│ 周期 │<->│       │   │
│  │  │<->│ 占空比 │<->│ 占空比 │<->│ 占空比 │<->│       │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              低频 PWM                               │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────┐ │   │
│  │  │             │    │             │    │         │ │   │
│  │  │             │    │             │    │         │ │   │
│  │  └─────────────┘    └─────────────┘    └─────────┘ │   │
│  │  │<---------->│ 周期 │<---------->│ 周期 │<->│ │   │
│  │  │<---------->│ 占空比 │<---------->│ 占空比 │<->│ │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **占空比与功率控制**
```
占空比 vs 功率输出
┌─────────────────────────────────────────────────────────────┐
│                    占空比控制                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   25% 占空比│ │   50% 占空比│ │   75% 占空比│         │
│  │   ┌─┐ ┌─┐  │ │  ┌───┐ ┌───┐│ │ ┌─────┐ ┌─┐│         │
│  │   │ │ │ │  │ │  │   │ │   │ │ │ │     │ │ ││         │
│  │   │ │ │ │  │ │  │   │ │   │ │ │ │     │ │ ││         │
│  │   └─┘ └─┘  │ │  └───┘ └───┘│ │ └─────┘ └─┘│         │
│  │   │<->│     │ │  │<--->│<--->│ │ │<----->│<->│         │
│  │   │   │     │ │  │     │     │ │ │       │   │         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │
│  │   低        │ │   中        │ │   高        │         │
│  │   功率      │ │   功率      │ │   功率      │         │
│  │ （25% 平均）│ │ （50% 平均）│ │ （75% 平均）│         │
│  └─────────────┘ └─────────────┘ └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

### **PWM 滤波与 DAC 效应**
```
PWM 到模拟转换
┌─────────────────────────────────────────────────────────────┐
│                    PWM 滤波过程                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              原始 PWM 信号                          │   │
│  │  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │       │   │
│  │  └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘       │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              RC 低通滤波器                          │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │   │
│  │  │     R       │ │     C       │ │             │   │   │
│  │  │             │ │             │ │             │   │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              滤波后的模拟输出                        │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  │                                             │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  │  │<---------->│ 平均值 │<---------->│       │   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### **🧠 概念基础**

#### **PWM 原理**
脉宽调制代表了数字控制模拟系统的一项基本技术。通过在高低状态之间快速切换数字信号，PWM 创建了一个有效的模拟输出，其平均值与占空比成正比。

**关键特性：**
- **数字控制**：PWM 使用数字信号控制模拟功率电平
- **效率**：切换操作最小化控制元件中的功耗
- **灵活性**：占空比和频率可独立控制
- **可扩展性**：同一技术适用于从毫瓦到千瓦

#### **为什么 PWM 很重要**
PWM 对现代嵌入式系统至关重要：

- **功率控制**：实现电机速度、LED 亮度和功率转换的精确控制
- **效率**：切换操作比线性控制方法更高效
- **数字集成**：PWM 可由微控制器定时器直接生成
- **噪声控制**：频率选择可将开关噪声移出敏感频段
- **成本效益**：PWM 控制通常比模拟替代方案更便宜

#### **PWM 设计挑战**
设计有效的 PWM 系统需要平衡多个相互竞争的需求：

- **频率选择**：更高频率提供更平滑输出，但增加开关损耗
- **分辨率 vs 频率**：定时器约束在精度和切换速率之间产生权衡
- **噪声管理**：必须选择开关频率以最小化干扰
- **滤波器设计**：RC 滤波器可将 PWM 转换为模拟，但引入延迟和纹波

## 🔧 PWM 基础

### **PWM 参数**
```c
// PWM 参数结构
typedef struct {
    uint32_t frequency;      // PWM 频率（Hz）
    uint32_t period;         // 定时器周期值
    uint32_t prescaler;      // 定时器预分频值
    uint16_t duty_cycle;     // 占空比（0-100）
    uint16_t resolution;     // PWM 分辨率（位）
} PWM_Config_t;

// 计算 PWM 参数
void pwm_calculate_parameters(PWM_Config_t* config, uint32_t clock_freq, uint32_t target_freq) {
    // 计算目标频率的预分频和周期
    uint32_t psc = 1;
    uint32_t arr = clock_freq / target_freq;
    
    // 若周期过大则调整预分频
    while (arr > 65535 && psc < 65535) {
        psc++;
        arr = (clock_freq / psc) / target_freq;
    }
    
    config->frequency = target_freq;
    config->period = arr - 1;
    config->prescaler = psc - 1;
    config->resolution = 16; // 假设 16 位定时器
}
```

### **占空比计算**
```c
// 将占空比百分比转换为定时器比较值
uint16_t duty_cycle_to_compare(uint16_t duty_cycle, uint16_t period) {
    return (uint16_t)((duty_cycle * period) / 100);
}

// 将定时器比较值转换为占空比百分比
uint16_t compare_to_duty_cycle(uint16_t compare_value, uint16_t period) {
    return (uint16_t)((compare_value * 100) / period);
}

// 设置 PWM 占空比
void pwm_set_duty_cycle(TIM_HandleTypeDef* htim, uint32_t channel, uint16_t duty_cycle) {
    uint16_t compare_value = duty_cycle_to_compare(duty_cycle, htim->Init.Period);
    __HAL_TIM_SET_COMPARE(htim, channel, compare_value);
}
```

---

## ⚙️ PWM 配置

### **基本 PWM 配置**
```c
// 配置定时器用于 PWM 输出
void pwm_timer_config(TIM_HandleTypeDef* htim, PWM_Config_t* config) {
    TIM_OC_InitTypeDef sConfigOC = {0};
    
    // 配置定时器基础
    htim->Instance = TIM2;
    htim->Init.Prescaler = config->prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = config->period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置 PWM 通道
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, TIM_CHANNEL_1);
    
    // 启动 PWM
    HAL_TIM_PWM_Start(htim, TIM_CHANNEL_1);
}
```

### **多通道 PWM 配置**
```c
// 配置多个 PWM 通道
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channels[4];
    uint8_t channel_count;
    PWM_Config_t config;
} MultiChannelPWM_t;

void multi_channel_pwm_init(MultiChannelPWM_t* mpwm, TIM_HandleTypeDef* htim,
                           uint32_t* channels, uint8_t count, uint32_t frequency) {
    mpwm->htim = htim;
    mpwm->channel_count = count;
    
    for (int i = 0; i < count; i++) {
        mpwm->channels[i] = channels[i];
    }
    
    // 计算 PWM 参数
    pwm_calculate_parameters(&mpwm->config, SystemCoreClock, frequency);
    
    // 配置定时器基础
    htim->Instance = TIM2;
    htim->Init.Prescaler = mpwm->config.prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = mpwm->config.period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置每个通道
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    for (int i = 0; i < count; i++) {
        HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, channels[i]);
        HAL_TIM_PWM_Start(htim, channels[i]);
    }
}
```

### **带中断的 PWM**
```c
// 配置带更新中断的 PWM
void pwm_with_interrupt_config(TIM_HandleTypeDef* htim, PWM_Config_t* config) {
    // 配置定时器基础
    htim->Instance = TIM2;
    htim->Init.Prescaler = config->prescaler;
    htim->Init.CounterMode = TIM_COUNTERMODE_UP;
    htim->Init.Period = config->period;
    htim->Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    htim->Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
    
    HAL_TIM_Base_Init(htim);
    
    // 配置 PWM 通道
    TIM_OC_InitTypeDef sConfigOC = {0};
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
    sConfigOC.Pulse = 0;
    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
    
    HAL_TIM_PWM_ConfigChannel(htim, &sConfigOC, TIM_CHANNEL_1);
    
    // 使能更新中断
    __HAL_TIM_ENABLE_IT(htim, TIM_IT_UPDATE);
    HAL_NVIC_SetPriority(TIM2_IRQn, 0, 0);
    HAL_NVIC_EnableIRQ(TIM2_IRQn);
    
    // 启动 PWM
    HAL_TIM_PWM_Start(htim, TIM_CHANNEL_1);
}

// PWM 中断处理程序
void TIM2_IRQHandler(void) {
    if (__HAL_TIM_GET_FLAG(&htim2, TIM_FLAG_UPDATE) != RESET) {
        if (__HAL_TIM_GET_IT_SOURCE(&htim2, TIM_IT_UPDATE) != RESET) {
            __HAL_TIM_CLEAR_IT(&htim2, TIM_IT_UPDATE);
            
            // 处理 PWM 更新
            pwm_update_callback();
        }
    }
}
```

---

## 🎛️ 占空比控制

### **平滑占空比转换**
```c
// 平滑占空比转换
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t current_duty;
    uint16_t target_duty;
    uint16_t step_size;
    uint32_t transition_time;
    uint32_t last_update_time;
} PWM_Transition_t;

void pwm_transition_init(PWM_Transition_t* transition, TIM_HandleTypeDef* htim,
                        uint32_t channel, uint16_t step_size, uint32_t transition_time_ms) {
    transition->htim = htim;
    transition->channel = channel;
    transition->current_duty = 0;
    transition->target_duty = 0;
    transition->step_size = step_size;
    transition->transition_time = transition_time_ms;
    transition->last_update_time = 0;
}

void pwm_transition_set_target(PWM_Transition_t* transition, uint16_t target_duty) {
    transition->target_duty = target_duty;
}

void pwm_transition_update(PWM_Transition_t* transition) {
    uint32_t current_time = HAL_GetTick();
    
    if (current_time - transition->last_update_time >= transition->transition_time) {
        if (transition->current_duty < transition->target_duty) {
            transition->current_duty += transition->step_size;
            if (transition->current_duty > transition->target_duty) {
                transition->current_duty = transition->target_duty;
            }
        } else if (transition->current_duty > transition->target_duty) {
            transition->current_duty -= transition->step_size;
            if (transition->current_duty < transition->target_duty) {
                transition->current_duty = transition->target_duty;
            }
        }
        
        pwm_set_duty_cycle(transition->htim, transition->channel, transition->current_duty);
        transition->last_update_time = current_time;
    }
}
```

### **占空比斜坡**
```c
// 占空比斜坡
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t start_duty;
    uint16_t end_duty;
    uint16_t current_duty;
    uint16_t step_size;
    uint32_t ramp_time;
    uint32_t step_time;
    uint32_t last_step_time;
    uint8_t ramping;
} PWM_Ramp_t;

void pwm_ramp_init(PWM_Ramp_t* ramp, TIM_HandleTypeDef* htim, uint32_t channel,
                   uint16_t step_size, uint32_t ramp_time_ms) {
    ramp->htim = htim;
    ramp->channel = channel;
    ramp->step_size = step_size;
    ramp->ramp_time = ramp_time_ms;
    ramp->ramping = 0;
    ramp->current_duty = 0;
}

void pwm_ramp_start(PWM_Ramp_t* ramp, uint16_t start_duty, uint16_t end_duty) {
    ramp->start_duty = start_duty;
    ramp->end_duty = end_duty;
    ramp->current_duty = start_duty;
    ramp->ramping = 1;
    ramp->last_step_time = HAL_GetTick();
    
    // 计算步进时间
    uint16_t total_steps = abs(end_duty - start_duty) / ramp->step_size;
    ramp->step_time = ramp->ramp_time / total_steps;
    
    // 设置初始占空比
    pwm_set_duty_cycle(ramp->htim, ramp->channel, ramp->current_duty);
}

void pwm_ramp_update(PWM_Ramp_t* ramp) {
    if (!ramp->ramping) return;
    
    uint32_t current_time = HAL_GetTick();
    
    if (current_time - ramp->last_step_time >= ramp->step_time) {
        if (ramp->current_duty < ramp->end_duty) {
            ramp->current_duty += ramp->step_size;
            if (ramp->current_duty > ramp->end_duty) {
                ramp->current_duty = ramp->end_duty;
                ramp->ramping = 0;
            }
        } else if (ramp->current_duty > ramp->end_duty) {
            ramp->current_duty -= ramp->step_size;
            if (ramp->current_duty < ramp->end_duty) {
                ramp->current_duty = ramp->end_duty;
                ramp->ramping = 0;
            }
        }
        
        pwm_set_duty_cycle(ramp->htim, ramp->channel, ramp->current_duty);
        ramp->last_step_time = current_time;
    }
}
```

---

## 🔄 频率控制

### **动态频率控制**
```c
// 动态频率控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t current_frequency;
    uint32_t target_frequency;
    uint32_t min_frequency;
    uint32_t max_frequency;
} PWM_Frequency_Control_t;

void pwm_frequency_control_init(PWM_Frequency_Control_t* freq_ctrl, TIM_HandleTypeDef* htim,
                               uint32_t min_freq, uint32_t max_freq) {
    freq_ctrl->htim = htim;
    freq_ctrl->min_frequency = min_freq;
    freq_ctrl->max_frequency = max_freq;
    freq_ctrl->current_frequency = min_freq;
    freq_ctrl->target_frequency = min_freq;
}

void pwm_set_frequency(PWM_Frequency_Control_t* freq_ctrl, uint32_t frequency) {
    if (frequency < freq_ctrl->min_frequency) {
        frequency = freq_ctrl->min_frequency;
    } else if (frequency > freq_ctrl->max_frequency) {
        frequency = freq_ctrl->max_frequency;
    }
    
    freq_ctrl->target_frequency = frequency;
    
    // 计算新的定时器参数
    PWM_Config_t config;
    pwm_calculate_parameters(&config, SystemCoreClock, frequency);
    
    // 更新定时器配置
    freq_ctrl->htim->Init.Prescaler = config.prescaler;
    freq_ctrl->htim->Init.Period = config.period;
    
    HAL_TIM_Base_Init(freq_ctrl->htim);
    freq_ctrl->current_frequency = frequency;
}
```

### **频率扫描**
```c
// 频率扫描
typedef struct {
    PWM_Frequency_Control_t* freq_ctrl;
    uint32_t start_frequency;
    uint32_t end_frequency;
    uint32_t sweep_time;
    uint32_t current_time;
    uint8_t sweeping;
} PWM_Frequency_Sweep_t;

void pwm_frequency_sweep_init(PWM_Frequency_Sweep_t* sweep, PWM_Frequency_Control_t* freq_ctrl,
                             uint32_t sweep_time_ms) {
    sweep->freq_ctrl = freq_ctrl;
    sweep->sweep_time = sweep_time_ms;
    sweep->sweeping = 0;
}

void pwm_frequency_sweep_start(PWM_Frequency_Sweep_t* sweep, uint32_t start_freq, uint32_t end_freq) {
    sweep->start_frequency = start_freq;
    sweep->end_frequency = end_freq;
    sweep->current_time = 0;
    sweep->sweeping = 1;
    
    // 设置初始频率
    pwm_set_frequency(sweep->freq_ctrl, start_freq);
}

void pwm_frequency_sweep_update(PWM_Frequency_Sweep_t* sweep) {
    if (!sweep->sweeping) return;
    
    sweep->current_time += 10; // 每 10ms 更新
    
    // 计算当前频率
    float progress = (float)sweep->current_time / sweep->sweep_time;
    if (progress > 1.0f) progress = 1.0f;
    
    uint32_t current_freq = sweep->start_frequency + 
                           (uint32_t)(progress * (sweep->end_frequency - sweep->start_frequency));
    
    pwm_set_frequency(sweep->freq_ctrl, current_freq);
    
    if (progress >= 1.0f) {
        sweep->sweeping = 0;
    }
}
```

---

## 📊 PWM 应用

### **LED 调光**
```c
// LED 调光控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t brightness;
    uint8_t fade_direction;
    uint16_t fade_speed;
} LED_Dimmer_t;

void led_dimmer_init(LED_Dimmer_t* dimmer, TIM_HandleTypeDef* htim, uint32_t channel) {
    dimmer->htim = htim;
    dimmer->channel = channel;
    dimmer->brightness = 0;
    dimmer->fade_direction = 0; // 0 = 淡入，1 = 淡出
    dimmer->fade_speed = 1;
}

void led_dimmer_set_brightness(LED_Dimmer_t* dimmer, uint16_t brightness) {
    dimmer->brightness = brightness;
    pwm_set_duty_cycle(dimmer->htim, dimmer->channel, brightness);
}

void led_dimmer_fade(LED_Dimmer_t* dimmer) {
    if (dimmer->fade_direction == 0) {
        // 淡入
        dimmer->brightness += dimmer->fade_speed;
        if (dimmer->brightness >= 100) {
            dimmer->brightness = 100;
            dimmer->fade_direction = 1;
        }
    } else {
        // 淡出
        dimmer->brightness -= dimmer->fade_speed;
        if (dimmer->brightness <= 0) {
            dimmer->brightness = 0;
            dimmer->fade_direction = 0;
        }
    }
    
    pwm_set_duty_cycle(dimmer->htim, dimmer->channel, dimmer->brightness);
}
```

### **电机速度控制**
```c
// 电机速度控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t speed;
    uint16_t max_speed;
    uint16_t min_speed;
    uint8_t direction;
} Motor_Controller_t;

void motor_controller_init(Motor_Controller_t* motor, TIM_HandleTypeDef* htim, uint32_t channel,
                          uint16_t min_speed, uint16_t max_speed) {
    motor->htim = htim;
    motor->channel = channel;
    motor->min_speed = min_speed;
    motor->max_speed = max_speed;
    motor->speed = 0;
    motor->direction = 0;
}

void motor_set_speed(Motor_Controller_t* motor, uint16_t speed) {
    if (speed > motor->max_speed) {
        speed = motor->max_speed;
    } else if (speed < motor->min_speed) {
        speed = motor->min_speed;
    }
    
    motor->speed = speed;
    pwm_set_duty_cycle(motor->htim, motor->channel, speed);
}

void motor_set_direction(Motor_Controller_t* motor, uint8_t direction) {
    motor->direction = direction;
    // 如需方向控制可添加额外的 GPIO 控制
}
```

### **音频生成**
```c
// 音频音调生成
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t frequency;
    uint16_t volume;
    uint8_t playing;
} Audio_Generator_t;

void audio_generator_init(Audio_Generator_t* audio, TIM_HandleTypeDef* htim, uint32_t channel) {
    audio->htim = htim;
    audio->channel = channel;
    audio->frequency = 440; // A4 音符
    audio->volume = 50;
    audio->playing = 0;
}

void audio_generator_play_tone(Audio_Generator_t* audio, uint16_t frequency, uint16_t volume) {
    audio->frequency = frequency;
    audio->volume = volume;
    audio->playing = 1;
    
    // 设置频率
    PWM_Config_t config;
    pwm_calculate_parameters(&config, SystemCoreClock, frequency);
    
    audio->htim->Init.Prescaler = config.prescaler;
    audio->htim->Init.Period = config.period;
    HAL_TIM_Base_Init(audio->htim);
    
    // 设置音量（占空比）
    pwm_set_duty_cycle(audio->htim, audio->channel, volume);
}

void audio_generator_stop(Audio_Generator_t* audio) {
    audio->playing = 0;
    pwm_set_duty_cycle(audio->htim, audio->channel, 0);
}
```

---

## ⚡ 高级 PWM 技术

### **相移 PWM**
```c
// 用于多相系统的相移 PWM
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channels[3];
    uint16_t duty_cycle;
    uint16_t phase_shift;
} Phase_Shifted_PWM_t;

void phase_shifted_pwm_init(Phase_Shifted_PWM_t* pspwm, TIM_HandleTypeDef* htim,
                           uint32_t* channels, uint16_t phase_shift) {
    pspwm->htim = htim;
    pspwm->phase_shift = phase_shift;
    
    for (int i = 0; i < 3; i++) {
        pspwm->channels[i] = channels[i];
    }
}

void phase_shifted_pwm_set_duty(Phase_Shifted_PWM_t* pspwm, uint16_t duty_cycle) {
    pspwm->duty_cycle = duty_cycle;
    
    // 为每个通道设置带相移的占空比
    for (int i = 0; i < 3; i++) {
        uint16_t phase_adjusted_duty = (duty_cycle + (i * pspwm->phase_shift)) % 100;
        pwm_set_duty_cycle(pspwm->htim, pspwm->channels[i], phase_adjusted_duty);
    }
}
```

### **死区插入**
```c
// 用于 H 桥控制的死区插入
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel1;
    uint32_t channel2;
    uint16_t dead_time;
} Dead_Time_PWM_t;

void dead_time_pwm_init(Dead_Time_PWM_t* dtpwm, TIM_HandleTypeDef* htim,
                       uint32_t channel1, uint32_t channel2, uint16_t dead_time) {
    dtpwm->htim = htim;
    dtpwm->channel1 = channel1;
    dtpwm->channel2 = channel2;
    dtpwm->dead_time = dead_time;
    
    // 配置死区
    __HAL_TIM_SET_AUTORELOAD(htim, htim->Init.Period);
    __HAL_TIM_SET_COMPARE(htim, channel1, 0);
    __HAL_TIM_SET_COMPARE(htim, channel2, dead_time);
}
```

---

## 🎯 常见应用

### **电源控制**
```c
// 降压转换器控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t duty_cycle;
    float output_voltage;
    float target_voltage;
    float input_voltage;
} Buck_Converter_t;

void buck_converter_init(Buck_Converter_t* buck, TIM_HandleTypeDef* htim, uint32_t channel,
                        float input_voltage) {
    buck->htim = htim;
    buck->channel = channel;
    buck->input_voltage = input_voltage;
    buck->output_voltage = 0;
    buck->target_voltage = 0;
    buck->duty_cycle = 0;
}

void buck_converter_set_voltage(Buck_Converter_t* buck, float target_voltage) {
    buck->target_voltage = target_voltage;
    
    // 计算占空比：Vout = Vin * 占空比
    float duty_cycle = (target_voltage / buck->input_voltage) * 100.0f;
    
    if (duty_cycle > 100.0f) duty_cycle = 100.0f;
    if (duty_cycle < 0.0f) duty_cycle = 0.0f;
    
    buck->duty_cycle = (uint16_t)duty_cycle;
    pwm_set_duty_cycle(buck->htim, buck->channel, buck->duty_cycle);
}
```

### **舵机控制**
```c
// 舵机电机控制
typedef struct {
    TIM_HandleTypeDef* htim;
    uint32_t channel;
    uint16_t position;
    uint16_t min_position;
    uint16_t max_position;
} Servo_Controller_t;

void servo_controller_init(Servo_Controller_t* servo, TIM_HandleTypeDef* htim, uint32_t channel) {
    servo->htim = htim;
    servo->channel = channel;
    servo->min_position = 5;   // 0.5ms 脉冲
    servo->max_position = 25;  // 2.5ms 脉冲
    servo->position = 15;      // 中心位置
}

void servo_set_position(Servo_Controller_t* servo, uint16_t position) {
    if (position < servo->min_position) {
        position = servo->min_position;
    } else if (position > servo->max_position) {
        position = servo->max_position;
    }
    
    servo->position = position;
    pwm_set_duty_cycle(servo->htim, servo->channel, position);
}
```

---

## ⚠️ 常见陷阱

### **1. 频率计算错误**
```c
// ❌ 错误：频率计算错误
void pwm_frequency_wrong(TIM_HandleTypeDef* htim, uint32_t frequency) {
    htim->Init.Period = 1000 / frequency; // 计算错误
}

// ✅ 正确：正确的频率计算
void pwm_frequency_correct(TIM_HandleTypeDef* htim, uint32_t frequency) {
    uint32_t clock_freq = SystemCoreClock;
    uint32_t period = (clock_freq / frequency) - 1;
    htim->Init.Period = period;
}
```

### **2. 缺少死区**
```c
// ❌ 错误：H 桥无死区
void h_bridge_control_wrong(TIM_HandleTypeDef* htim) {
    // 无死区的直接 PWM 控制
}

// ✅ 正确：带死区
void h_bridge_control_correct(TIM_HandleTypeDef* htim) {
    // 配置死区以实现安全切换
    __HAL_TIM_SET_COMPARE(htim, TIM_CHANNEL_1, 0);
    __HAL_TIM_SET_COMPARE(htim, TIM_CHANNEL_2, DEAD_TIME_VALUE);
}
```

### **3. 分辨率不足**
```c
// ❌ 错误：低分辨率 PWM
void pwm_low_resolution_wrong(TIM_HandleTypeDef* htim) {
    htim->Init.Period = 100; // 仅 100 个级别
}

// ✅ 正确：高分辨率 PWM
void pwm_high_resolution_correct(TIM_HandleTypeDef* htim) {
    htim->Init.Period = 4095; // 4096 个级别（12 位）
}
```

---

## ✅ 最佳实践

### **1. 使用适当的频率**
```c
// 为不同应用使用适当频率
void configure_pwm_frequency(PWM_Config_t* config, uint8_t application_type) {
    switch (application_type) {
        case PWM_LED_DIMMING:
            config->frequency = 1000; // LED 调光用 1kHz
            break;
        case PWM_MOTOR_CONTROL:
            config->frequency = 20000; // 电机控制用 20kHz
            break;
        case PWM_AUDIO:
            config->frequency = 44100; // 音频用 44.1kHz
            break;
        case PWM_POWER_SUPPLY:
            config->frequency = 100000; // 电源用 100kHz
            break;
    }
}
```

### **2. 实现平滑转换**
```c
// 始终使用平滑转换以获得更好性能
void smooth_pwm_transition(TIM_HandleTypeDef* htim, uint32_t channel,
                          uint16_t start_duty, uint16_t end_duty, uint16_t steps) {
    uint16_t step_size = (end_duty - start_duty) / steps;
    
    for (int i = 0; i < steps; i++) {
        uint16_t current_duty = start_duty + (i * step_size);
        pwm_set_duty_cycle(htim, channel, current_duty);
        HAL_Delay(10); // 平滑转换的小延时
    }
    
    pwm_set_duty_cycle(htim, channel, end_duty);
}
```

### **3. 监控 PWM 性能**
```c
// 监控 PWM 性能和效率
typedef struct {
    uint32_t frequency;
    uint16_t duty_cycle;
    float efficiency;
    uint32_t switching_losses;
} PWM_Performance_t;

void pwm_performance_monitor(PWM_Performance_t* perf, TIM_HandleTypeDef* htim) {
    // 计算开关损耗
    perf->switching_losses = perf->frequency * perf->duty_cycle / 100;
    
    // 计算效率（简化）
    perf->efficiency = 100.0f - (perf->switching_losses / 1000.0f);
}
```

---

## 🎯 面试问题

### **基本问题**
1. **什么是 PWM，它如何工作？**
   - 脉宽调制，在开/关状态之间快速切换

2. **什么是占空比，如何计算？**
   - 信号保持高电平的时间百分比，（导通时间 / 总时间）* 100

3. **频率和周期之间有什么关系？**
   - 频率 = 1 / 周期

### **高级问题**
1. **你如何实现相移 PWM？**
   - 使用多个具有不同相位延迟的通道

2. **什么是死区，为什么重要？**
   - 切换之间的延迟，防止 H 桥中的直通

3. **你如何为不同应用优化 PWM？**
   - 选择适当的频率、分辨率和切换策略

### **实践问题**
1. **使用 PWM 设计电机速度控制器**
   - 定时器配置、占空比控制、反馈回路

2. **实现带平滑转换的 LED 调光**
   - PWM 生成、占空比斜坡、时序控制

3. **创建音频音调生成器**
   - 频率控制、音量控制、波形生成

---

## 📚 其他资源

### **文档**
- [STM32 定时器参考手册](https://www.st.com/resource/en/reference_manual/dm00031020-stm32f405-415-stm32f407-417-stm32f427-437-and-stm32f429-439-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf)
- [ARM Cortex-M 定时器编程](https://developer.arm.com/documentation/dui0552/a/the-cortex-m3-processor/peripherals/general-purpose-timers)

### **工具**
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) - PWM 配置
- [PWM 计算器](https://www.st.com/resource/en/user_manual/dm00104712-stm32cubemx-user-manual-stmicroelectronics.pdf)

### **相关主题**
- **[[GPIO_Configuration]]** - GPIO 模式、配置、电气特性
- **[[Timer_Counter_Programming]]** - 输入捕获、输出比较、频率测量
- **[[Analog_IO]]** - ADC 采样技术、DAC 输出生成

---

**下一个主题：** [[Timer_Counter_Programming]] → [[External_Interrupts]]
