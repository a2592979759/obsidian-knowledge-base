---
tags:
  - embedded
  - 裸机编程
  - 单片机组
created: 2026-08-27
source: bare-metal-programming-guide/README_zh-CN.md
---

# 05 · LED 闪烁

> 前置知识：[[02-GPIO与引脚]]、[[04-Makefile构建]]。
> 本节对应的完整工程位于 `steps/step-1-blinky/`。

## 板载 LED

现在我们已经搭建好了完整的构建、烧写的基础设施，是时候让固件做点儿有用的事情了。什么是有用的事情？当然是闪烁 LED 了！Nucleo-F429ZI 开发板有 3 颗 LED，在开发板数据手册的 6.5 节，我们可以看到板载 LED 连接的引脚：

- PB0: green LED
- PB7: blue LED
- PB14: red LED

我们以 **PB7（蓝色 LED）** 为目标。

## 引脚与 GPIO 工具

再次修改 `main.c` 文件，添加上引脚定义，然后把蓝色 LED 引脚设为输出模式，开始无限循环。首先，把我们之前讨论过的 GPIO 定义和模式设置拷贝过来，注意，现在又新加了一个 `BIT(position)` 工具宏：

```c
#include <inttypes.h>
#include <stdbool.h>

#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)

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
  gpio->MODER |= (mode & 3) << (n * 2);    // Set new mode
}
```

## 使能 GPIO 时钟（RCC）

某些微控制器在上电时会把所有外设都自动使能，然而，STM32 微控制器在上电时外设是默认关闭的，以降低功耗。为了使能 GPIO 外设，我们需要通过 RCC 单元使能外设时钟。在芯片数据手册 7.3.10 节，可以找到 AHB1ENR 寄存器与此相关，还是先定义整个 RCC 单元：

```c
struct rcc {
  volatile uint32_t CR, PLLCFGR, CFGR, CIR, AHB1RSTR, AHB2RSTR, AHB3RSTR,
      RESERVED0, APB1RSTR, APB2RSTR, RESERVED1[2], AHB1ENR, AHB2ENR, AHB3ENR,
      RESERVED2, APB1ENR, APB2ENR, RESERVED3[2], AHB1LPENR, AHB2LPENR,
      AHB3LPENR, RESERVED4, APB1LPENR, APB2LPENR, RESERVED5[2], BDCR, CSR,
      RESERVED6[2], SSCGR, PLLI2SCFGR;
};
#define RCC ((struct rcc *) 0x40023800)
```

在 AHB1ENR 寄存器文档中可以看到 0-8 位控制 GPIOA - GPIOI 的时钟。要把蓝色 LED（PB7）设为输出，需要先使能 GPIOB 的时钟：

```c
int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  RCC->AHB1ENR |= BIT(PINBANK(led));     // Enable GPIO clock for LED
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  for (;;) asm volatile("nop");          // Infinite loop
  return 0;
}
```

## 开关引脚（BSRR）

接下来需要做的就是找到如何开关 GPIO 引脚，然后在主循环中点亮 LED，延时，熄灭 LED，延时。在芯片数据手册 8.4.7 节，可以看到 BSRR 寄存器与设置电压高低有关，低 16 位设置 ODR 寄存器输出高，高 16 位设置 ODR 寄存器输出低。为此定义一个 API 函数：

```c
static inline void gpio_write(uint16_t pin, bool val) {
  struct gpio *gpio = GPIO(PINBANK(pin));
  gpio->BSRR = (1U << PINNO(pin)) << (val ? 0 : 16);
}
```

## 延时

下一步我们需要实现一个延时函数，目前还不需要精确延时，所以定义一个 `spin()` 函数，执行 NOP 指令给定的次数：

```c
static inline void spin(volatile uint32_t count) {
  while (count--) asm("nop");
}
```

最后，修改主循环来让 LED 闪烁起来：

```c
  for (;;) {
    gpio_write(pin, true);
    spin(999999);
    gpio_write(pin, false);
    spin(999999);
  }
```

执行 `make flash` 来看蓝色 LED 闪烁吧！

## 源码整合：step-1-blinky

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

struct rcc {
  volatile uint32_t CR, PLLCFGR, CFGR, CIR, AHB1RSTR, AHB2RSTR, AHB3RSTR,
      RESERVED0, APB1RSTR, APB2RSTR, RESERVED1[2], AHB1ENR, AHB2ENR, AHB3ENR,
      RESERVED2, APB1ENR, APB2ENR, RESERVED3[2], AHB1LPENR, AHB2LPENR,
      AHB3LPENR, RESERVED4, APB1LPENR, APB2LPENR, RESERVED5[2], BDCR, CSR,
      RESERVED6[2], SSCGR, PLLI2SCFGR;
};
#define RCC ((struct rcc *) 0x40023800)

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

int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  RCC->AHB1ENR |= BIT(PINBANK(led));     // Enable GPIO clock for LED
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  for (;;) {
    gpio_write(led, true);
    spin(999999);
    gpio_write(led, false);
    spin(999999);
  }
  return 0;
}

// Startup code
__attribute__((naked, noreturn)) void _reset(void) {          //naked=don't generate prologue/epilogue  //noreturn=it will never exit
  // memset .bss to zero, and copy .data section to RAM region
  extern long _sbss, _ebss, _sdata, _edata, _sidata;          //linker symbols declared in the .ld file
  for (long *dst = &_sbss; dst < &_ebss; dst++) *dst = 0;
  for (long *dst = &_sdata, *src = &_sidata; dst < &_edata;) *dst++ = *src++;

  main();             // Call main()
  for (;;) (void) 0;  // Infinite loop in the case if main() returns
}

extern void _estack(void);  // Defined in link.ld

// 16 standard and 91 STM32-specific handlers
__attribute__((section(".vectors"))) void (*const tab[16 + 91])(void) = { // defines the interrupt vector table and dumps it into .vectors via the linker script
    _estack, _reset};
```

> `step-1-blinky` 的 `Makefile`、`link.ld` 与 `step-0-minimal` 完全相同，见 [[04-Makefile构建]]、[[03-启动代码与链接]]。

## 下一步

这里的 `spin()` 延时是不精确的。接下来用 SysTick 实现精确计时，见 [[06-SysTick定时器]]。
