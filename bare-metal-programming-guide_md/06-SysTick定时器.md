---
tags:
  - embedded
  - 裸机编程
  - 单片机组
created: 2026-08-27
source: bare-metal-programming-guide/README_zh-CN.md
---

# 06 · SysTick 定时器

> 前置知识：[[02-GPIO与引脚]]、[[05-LED闪烁]]。
> 本节对应的完整工程位于 `steps/step-2-systick/`。

## 为什么需要 SysTick

为了实现精确的时间控制，我们应该使能 ARM 的 SysTick 中断。SysTick 是一个 24 位的硬件计数器，是 ARM 核的一部分，因为在 ARM 的数据手册中有它的文档。从芯片数据手册中可以看到，SysTick 有 4 个寄存器：

- CTRL，使能/禁能 SysTick
- LOAD，初始计数值
- VAL，当前计数值，每个时钟周期递减
- CALIB，校准寄存器

每次 VAL 减到 0，就会产生一个 SysTick 中断，SysTick 中断在向量表中的索引为 15，我们需要设置它。在启动时，Nucleo-F429ZI 的时钟是 16MHz，我们可以配置 SysTick 计数器使其每毫秒产生一个中断。

## 定义 SysTick 外设

首先，定义 SysTick 外设，我们知道有 4 个寄存器，并且从芯片数据手册中可以知道 SysTick 地址为 `0xe000e010`，编写代码：

```c
struct systick {
  volatile uint32_t CTRL, LOAD, VAL, CALIB;
};
#define SYSTICK ((struct systick *) 0xe000e010)
```

## 初始化 SysTick

接下来定义一个 API 函数配置 SysTick，我们需要在 `SYSTICK->CTRL` 寄存器使能 SysTick，同时也需要在 `RCC->APB2ENR` 中使能它的时钟，这一点在芯片数据手册 7.4.14 节描述：

```c
#define BIT(x) (1UL << (x))
static inline void systick_init(uint32_t ticks) {
  if ((ticks - 1) > 0xffffff) return;  // Systick timer is 24 bit
  SYSTICK->LOAD = ticks - 1;
  SYSTICK->VAL = 0;
  SYSTICK->CTRL = BIT(0) | BIT(1) | BIT(2);  // Enable systick
  RCC->APB2ENR |= BIT(14);                   // Enable SYSCFG
}
```

默认情况下，Nucleo-F429ZI 运行频率为 16MHz，这表示如果我们调用 `systick_init(16000000 / 1000)`，就会每毫秒产生一个 SysTick 中断。

## 中断处理函数与 volatile

还需要定义一个中断处理函数，现在只是在中断处理函数中递增一个 32 位数：

```c
static volatile uint32_t s_ticks; // volatile is important!!
void SysTick_Handler(void) {
  s_ticks++;
}
```

注意 `s_ticks` 要用 `volatile` 说明符，在中断处理函数中的任何变量都应该标记为 `volatile`，这样可以避免编译器通过在寄存器中缓存变量值的优化操作。`volatile` 关键字可以是生成的代码总是从存储空间载入变量值。

现在把 SysTick 中断处理函数加到向量表中：

```c
__attribute__((section(".vectors"))) void (*tab[16 + 91])(void) = {
    0, _reset, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, SysTick_Handler};
```

这样我们就有精确的毫秒时钟了！

## 通用定时器工具函数

再创建一个任意周期的定时器工具函数：

```c
// t: expiration time, prd: period, now: current time. Return true if expired
bool timer_expired(uint32_t *t, uint64_t prd, uint64_t now) {
  if (now + prd < *t) *t = 0;                    // Time wrapped? Reset timer
  if (*t == 0) *t = now + prd;                   // First poll? Set expiration
  if (*t > now) return false;                    // Not expired yet, return
  *t = (now - *t) > prd ? now + prd : *t + prd;  // Next expiration time
  return true;                                   // Expired, return true
}
```

使用这个精确的定时器函数更新闪烁 LED 的主循环，例如，每 500ms 闪烁一次：

```c
  uint32_t timer = 0, period = 500;       // Declare timer and 500ms period
  for (;;) {
    if (timer_expired(&timer, period, s_ticks)) {
      static bool on;       // This block is executed
      gpio_write(led, on);  // Every `period` milliseconds
      on = !on;             // Toggle LED state
    }
    // Here we could perform other activities!
  }
```

通过使用 `SysTick` 和 `timer_expired()` 工具函数，使主循环成为非阻塞的。这意味着在主循环中我们还可以执行许多动作，例如，创建不同周期的定时器，它们都能及时被触发。

## 源码整合：step-2-systick

### main.c

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#include <inttypes.h>
#include <stdbool.h>

#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)

struct systick {
  volatile uint32_t CTRL, LOAD, VAL, CALIB;
};
#define SYSTICK ((struct systick *) 0xe000e010)  // 2.2.2

struct rcc {
  volatile uint32_t CR, PLLCFGR, CFGR, CIR, AHB1RSTR, AHB2RSTR, AHB3RSTR,
      RESERVED0, APB1RSTR, APB2RSTR, RESERVED1[2], AHB1ENR, AHB2ENR, AHB3ENR,
      RESERVED2, APB1ENR, APB2ENR, RESERVED3[2], AHB1LPENR, AHB2LPENR,
      AHB3LPENR, RESERVED4, APB1LPENR, APB2LPENR, RESERVED5[2], BDCR, CSR,
      RESERVED6[2], SSCGR, PLLI2SCFGR;
};
#define RCC ((struct rcc *) 0x40023800)

static inline void systick_init(uint32_t ticks) {
  if ((ticks - 1) > 0xffffff) return;  // Systick timer is 24 bit
  SYSTICK->LOAD = ticks - 1;
  SYSTICK->VAL = 0;
  SYSTICK->CTRL = BIT(0) | BIT(1) | BIT(2);  // Enable systick
  RCC->APB2ENR |= BIT(14);                   // Enable SYSCFG
}

struct gpio {
  volatile uint32_t MODER, OTYPER, OSPEEDR, PUPDR, IDR, ODR, BSRR, LCKR, AFR[2];
};
#define GPIO(bank) ((struct gpio *) (0x40020000 + 0x400 * (bank)))

// Enum values are per datasheet: 0, 1, 2, 3
enum { GPIO_MODE_INPUT, GPIO_MODE_OUTPUT, GPIO_MODE_AF, GPIO_MODE_ANALOG };

static inline void gpio_set_mode(uint16_t pin, uint8_t mode) {
  struct gpio *gpio = GPIO(PINBANK(pin));  // GPIO bank
  int n = PINNO(pin);                      // Pin number
  gpio->MODER &= ~(3U << (n * 2));         // Clear existing setting
  gpio->MODER |= (mode & 3U) << (n * 2);   // Set new mode
}

static inline void gpio_write(uint16_t pin, bool val) {
  struct gpio *gpio = GPIO(PINBANK(pin));
  gpio->BSRR = (1U << PINNO(pin)) << (val ? 0 : 16);
}

static inline void spin(volatile uint32_t count) {
  while (count--) asm("nop");
}

static volatile uint32_t s_ticks;
void SysTick_Handler(void) {
  s_ticks++;
}

// t: expiration time, prd: period, now: current time. Return true if expired
bool timer_expired(uint32_t *t, uint32_t prd, uint32_t now) {
  if (now + prd < *t) *t = 0;                    // Time wrapped? Reset timer
  if (*t == 0) *t = now + prd;                   // Firt poll? Set expiration
  if (*t > now) return false;                    // Not expired yet, return
  *t = (now - *t) > prd ? now + prd : *t + prd;  // Next expiration time
  return true;                                   // Expired, return true
}

int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  RCC->AHB1ENR |= BIT(PINBANK(led));     // Enable GPIO clock for LED
  systick_init(16000000 / 1000);         // Tick every 1 ms
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  uint32_t timer = 0, period = 500;      // Declare timer and 500ms period
  for (;;) {
    if (timer_expired(&timer, period, s_ticks)) {
      static bool on;       // This block is executed
      gpio_write(led, on);  // Every `period` milliseconds
      on = !on;             // Toggle LED state
    }
    // Here we could perform other activities!
  }
  return 0;
}

// Startup code
__attribute__((naked, noreturn)) void _reset(void) {
  // Initialise memory
  extern long _sbss, _ebss, _sdata, _edata, _sidata;
  for (long *dst = &_sbss; dst < &_ebss; dst++) *dst = 0;
  for (long *dst = &_sdata, *src = &_sidata; dst < &_edata;) *dst++ = *src++;

  // Call main()
  main();
  for (;;) (void) 0;  // Infinite loop
}

extern void _estack(void);  // Defined in link.ld

// 16 standard and 91 STM32-specific handlers
__attribute__((section(".vectors"))) void (*const tab[16 + 91])(void) = {
    _estack, _reset, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, SysTick_Handler};
```

> `step-2-systick` 的 `Makefile`、`link.ld` 与 `step-0-minimal` 相同（`Makefile` 的 `SOURCES = main.c`）。

## 下一步

LED 已经能精确计时闪烁，但还无法输出诊断信息。接下来添加串口调试输出，见 [[07-UART串口]]。
