---
tags: [嵌入式, 内存映射, 寄存器, 硬件]
source: Data_Struct_Implementation/memoryMap
created: 2026-08-27
---

# 内存映射 IO 寄存器实现（Memory Map IO Register Implementation）

### 问题描述

*假设有如下 GPIO 内存映射，请实现一种数据结构，能以一种简洁易懂的方式对该映射中的某些寄存器进行置位/清除。这是一台**小端（little endian）**机器。*

![[_assets/GPIO_memory_map.png]]

*请将 GPIO 15 置为 1。*

*（GPIO&lt;pin_num&gt; 通过向 ODR 寄存器的 bit&lt;pin_num&gt; 写 1 来置位）*。

### 实现

```c
// 寄存器值使用 Volatile。硬件可能随时改变其值。
typedef struct {
    volatile uint32_t MODER,  // 小端，LSB 在最低地址
    volatile uint32_t OTYPER,
    volatile uint32_t OSPEEDR,
    volatile uint32_t PUPDR,
    volatile uint32_t IDR,
    volatile uint32_t ODR,
    volatile uint32_t BSRR,
    volatile uint32_t LCKR,
    volatile uint32_t AFR[2],
    volatile uint32_t BRR,
    volatile uint32_t ASCR,  
} GPIO_REG;

// 寄存器端口映射
#define GPIO_A ((GPIO_REG*) 0x48000000)
#define GPIO_B ((GPIO_REG*) 0x48000400)
#define GPIO_C ((GPIO_REG*) 0x48000800)

// 将 GPIOA 的 pin15 置为 1
GPIO_A->ODR |= 1UL << 15;

// 宏：设置任意 GPIO 端口的任意引脚
#define SET_GPIO(pin_num, GPIO_PORT) ((GPIO_PORT)->ODR = (1UL<<pin_num))
```

### 追问

*如果原内存映射中以下寄存器是 16 位寄存器而非 32 位，你会如何改动数据结构？*

```
uint16_t OTYPER
uint16_t IDR
uint16_t ODR
```

### 实现

```c
#define _IO volatile

typedef struct {
    _IO uint32_t MODER,
    _IO uint16_t OTYPER,
        uint16_t rev0,  // 2 字节填充
    _IO uint32_t OSPEEDR,
    _IO uint32_t PUPDR,
    _IO uint16_t IDR,
        uint16_t rev1,  // 2 字节填充
    _IO uint16_t ODR,
        uint16_t rev2,  // 2 字节填充
    _IO uint32_t BSRR,
    _IO uint32_t LCKR,
    _IO uint32_t AFR[2],
    _IO uint32_t BRR,
    _IO uint32_t ASCR,  
} GPIO_REG;
```

## 分析

- 用结构体把内存映射中的寄存器按地址顺序“复刻”出来，每个字段都是一个 `volatile`（硬件可随时修改）寄存器。
- 通过基址 + 偏移访问寄存器，如 `GPIO_A->ODR`。这种方式把寄存器访问变成直观的结构体成员访问。
- 基址用宏 `GPIO_A` 等定义（如 Cortex-M4 的 GPIO 起始地址 0x48000000）。
- 16 位寄存器与相邻 32 位寄存器混排时，需要用**显式填充字段**（如 `rev0`）补齐 2 字节，以满足对齐且保持寄存器在内存映射中的正确偏移。

## 参考

[Yifeng zhu - Embedded Systems with ARM Cortex-M Microcontrollers in Assembly Language and C: Third Edition](https://www.youtube.com/watch?v=aT5XMOrid7Y&list=PLRJhV4hUhIymmp5CCeIFPyxbknsdcXCc8&index=5&ab_channel=EmbeddedSystemswithARMCortex-MMicrocontrollersinAssemblyLanguageandC)

## 相关文档
- [[sizeof]] —— 结构体对齐与填充
- [[endianess]] —— 小端/大端
