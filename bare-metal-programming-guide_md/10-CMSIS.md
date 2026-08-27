---
tags:
  - embedded
  - 裸机编程
  - 单片机组
created: 2026-08-27
source: bare-metal-programming-guide/README_zh-CN.md
---

# 10 · CMSIS

> 前置知识：[[02-GPIO与引脚]]、[[08-printf重定向]]。本节对应的完整工程位于 `steps/step-5-cmsis/`。

## CMSIS 是什么

在前面的部分，我们仅使用数据手册、编辑器和 GCC 编译器开发了固件程序，使用数据手册创建了外设结构定义。

现在我们已经知道 MCU 是怎么工作的，是时候介绍一下 CMSIS 头文件了。它是什么？它是由 MCU 厂商创建和提供的带有全部定义的头文件。它包含 MCU 相关的全部，所以很庞大。

CMSIS 代表通用微控制器软件接口标准（Common Microcontroller Software Interface Standard），因此它是 MCU 制造商指定外设 API 的共同基础。因为 CMSIS 是一种 ARM 标准，并且 CMSIS 头文件由 MCU 厂商提供，所以是权威的来源。因此，使用供应商头文件是首选方法，而不是手动编写定义。

在这一节，我们将使用供应商 CMSIS 头文件替换 `hal.h` 中的 API 函数，并保持固件其它部分不变。

## 下载 CMSIS 头文件

STM32 F4 系列的 CMSIS 头文件在这个[仓库](https://github.com/STMicroelectronics/cmsis_device_f4)，从那里将以下文件拷到我们的固件文件夹 [step-5-cmsis](step-5-cmsis)：

- [stm32f429xx.h](https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f4/master/Include/stm32f429xx.h)
- [system_stm32f4xx.h](https://raw.githubusercontent.com/STMicroelectronics/cmsis_device_f4/master/Include/system_stm32f4xx.h)

Those two files depend on a standard ARM CMSIS includes, download them too:
- [core_cm4.h](https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF4/master/Drivers/CMSIS/Core/Include/core_cm4.h)
- [cmsis_gcc.h](https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF4/master/Drivers/CMSIS/Core/Include/cmsis_gcc.h)
- [cmsis_version.h](https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF4/master/Drivers/CMSIS/Core/Include/cmsis_version.h)
- [cmsis_compiler.h](https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF4/master/Drivers/CMSIS/Core/Include/cmsis_compiler.h)
- [mpu_armv7.h](https://raw.githubusercontent.com/STMicroelectronics/STM32CubeF4/master/Drivers/CMSIS/Core/Include/mpu_armv7.h)

然后移除 `hal.h` 中所有外设 API 和定义，只留下标准 C 包含、供应商 CMSIS 包含，引脚定义等：

```c
#pragma once

#include <inttypes.h>
#include <stdbool.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/stat.h>

#include "stm32f429xx.h"

#define FREQ 16000000  // CPU frequency, 16 Mhz
#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)

static inline void spin(volatile uint32_t count) {
  while (count--) asm("nop");
}

static inline bool timer_expired(uint32_t *t, uint32_t prd, uint32_t now) {
  ...
}
```

如果我们执行 `make clean build` 重新编译固件，GCC 会报错：缺少 `systick_init()`, `GPIO_MODE_OUTPUT`, `uart_init()`, 和 `UART3`。我们使用 STM32 CMSIS 文件重新添加它们。

## 替换为 CMSIS 类型

从 `systick_init()` 开始，`core_cm4.h` 头文件中定义了 `SysTick_Type` 结构体，与我们的 `struct systick` 和 `SysTick` 外设相关宏定义作用相同。还有，`stm32f429xx.h` 头文件中有一个 `RCC_TypeDef` 结构体与我们的 `RCC` 宏定义一样，所以我们的 `systick_init()` 函数几乎不用修改，只需要用 `SYSTICK` 替换 `SysTick`：

```c
static inline void systick_init(uint32_t ticks) {
  if ((ticks - 1) > 0xffffff) return;  // Systick timer is 24 bit
  SysTick->LOAD = ticks - 1;
  SysTick->VAL = 0;
  SysTick->CTRL = BIT(0) | BIT(1) | BIT(2);  // Enable systick
  RCC->APB2ENR |= BIT(14);                   // Enable SYSCFG
}
```

接下来是 `gpio_set_mode()` 函数。`stm32f429xx.h` 头文件中有一个 `GPIO_TypeDef` 结构体，与我们的 `struct gpio` 相同，使用它改写：

```c
#define GPIO(bank) ((GPIO_TypeDef *) (GPIOA_BASE + 0x400 * (bank)))
enum { GPIO_MODE_INPUT, GPIO_MODE_OUTPUT, GPIO_MODE_AF, GPIO_MODE_ANALOG };

static inline void gpio_set_mode(uint16_t pin, uint8_t mode) {
  GPIO_TypeDef *gpio = GPIO(PINBANK(pin));  // GPIO bank
  int n = PINNO(pin);                      // Pin number
  RCC->AHB1ENR |= BIT(PINBANK(pin));       // Enable GPIO clock
  gpio->MODER &= ~(3U << (n * 2));         // Clear existing setting
  gpio->MODER |= (mode & 3) << (n * 2);    // Set new mode
}
```

`gpio_set_af()` 和 `gpio_write()` 也是一样简单替换就行。

然后是串口，CMSIS 中有 `USART_TypeDef` 和 `USART1`、`USART2`、`USART3` 的定义，使用它们：

```c
#define UART1 USART1
#define UART2 USART2
#define UART3 USART3
```

在 `uart_init()` 以及其它串口函数中将 `struct uart` 替换为 `USART_TypeDef`，其余部分保持不变。

## 重新组织 include 目录

做完这些，重新编译和烧写固件。LED 又闪烁起来，串口也有输出了。恭喜！

我们已经使用供应商 CMSIS 头文件重写了固件代码，现在重新组织下代码，把所有标准文件放到 `include` 目录下，然后更新 Makefile 文件让 GCC 编译器知道：

```make
...
  -g3 -Os -ffunction-sections -fdata-sections -I. -Iinclude \
```

现在得到了一个可以在未来的工程中重用的工程模板。

## 源码整合：step-5-cmsis

### hal.h

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#pragma once

#include <inttypes.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>

#include "stm32f429xx.h"

#define FREQ 16000000  // CPU frequency, 16 Mhz
#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)

static inline void spin(volatile uint32_t count) {
  while (count--) asm("nop");
}

#define GPIO(bank) ((GPIO_TypeDef *) (GPIOA_BASE + 0x400U * (bank)))
enum { GPIO_MODE_INPUT, GPIO_MODE_OUTPUT, GPIO_MODE_AF, GPIO_MODE_ANALOG };

static inline void gpio_set_mode(uint16_t pin, uint8_t mode) {
  GPIO_TypeDef *gpio = GPIO(PINBANK(pin));  // GPIO bank
  int n = PINNO(pin);                       // Pin number
  RCC->AHB1ENR |= BIT(PINBANK(pin));        // Enable GPIO clock
  gpio->MODER &= ~(3U << (n * 2));          // Clear existing setting
  gpio->MODER |= (mode & 3U) << (n * 2);    // Set new mode
}

static inline void gpio_set_af(uint16_t pin, uint8_t af_num) {
  GPIO_TypeDef *gpio = GPIO(PINBANK(pin));  // GPIO bank
  int n = PINNO(pin);                       // Pin number
  gpio->AFR[n >> 3] &= ~(15UL << ((n & 7) * 4));
  gpio->AFR[n >> 3] |= ((uint32_t) af_num) << ((n & 7) * 4);
}

static inline void gpio_write(uint16_t pin, bool val) {
  GPIO_TypeDef *gpio = GPIO(PINBANK(pin));
  gpio->BSRR = (1U << PINNO(pin)) << (val ? 0 : 16);
}

#define UART1 USART1
#define UART2 USART2
#define UART3 USART3

#ifndef UART_DEBUG
#define UART_DEBUG USART3
#endif

static inline void uart_init(USART_TypeDef *uart, unsigned long baud) {
  // https://www.st.com/resource/en/datasheet/stm32f429zi.pdf
  uint8_t af = 7;           // Alternate function
  uint16_t rx = 0, tx = 0;  // pins

  if (uart == UART1) RCC->APB2ENR |= BIT(4);
  if (uart == UART2) RCC->APB1ENR |= BIT(17);
  if (uart == UART3) RCC->APB1ENR |= BIT(18);

  if (uart == UART1) tx = PIN('A', 9), rx = PIN('A', 10);
  if (uart == UART2) tx = PIN('A', 2), rx = PIN('A', 3);
  if (uart == UART3) tx = PIN('D', 8), rx = PIN('D', 9);

  gpio_set_mode(tx, GPIO_MODE_AF);
  gpio_set_af(tx, af);
  gpio_set_mode(rx, GPIO_MODE_AF);
  gpio_set_af(rx, af);
  uart->CR1 = 0;                           // Disable this UART
  uart->BRR = FREQ / baud;                 // FREQ is a UART bus frequency
  uart->CR1 |= BIT(13) | BIT(2) | BIT(3);  // Set UE, RE, TE
}

static inline void uart_write_byte(USART_TypeDef *uart, uint8_t byte) {
  uart->DR = byte;
  while ((uart->SR & BIT(7)) == 0) spin(1);
}

static inline void uart_write_buf(USART_TypeDef *uart, char *buf, size_t len) {
  while (len-- > 0) uart_write_byte(uart, *(uint8_t *) buf++);
}

static inline int uart_read_ready(USART_TypeDef *uart) {
  return uart->SR & BIT(5);  // If RXNE bit is set, data is ready
}

static inline uint8_t uart_read_byte(USART_TypeDef *uart) {
  return (uint8_t) (uart->DR & 255);
}

static inline bool timer_expired(volatile uint32_t *t, uint32_t prd,
                                 uint32_t now) {
  if (now + prd < *t) *t = 0;                    // Time wrapped? Reset timer
  if (*t == 0) *t = now + prd;                   // Firt poll? Set expiration
  if (*t > now) return false;                    // Not expired yet, return
  *t = (now - *t) > prd ? now + prd : *t + prd;  // Next expiration time
  return true;                                   // Expired, return true
}
```

### main.c

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#include "hal.h"

static volatile uint32_t s_ticks;
void SysTick_Handler(void) {
  s_ticks++;
}

uint32_t SystemCoreClock = FREQ;
void SystemInit(void) {
  RCC->APB2ENR |= RCC_APB2ENR_SYSCFGEN;    // Enable SYSCFG
  SysTick_Config(SystemCoreClock / 1000);  // Tick every 1 ms
}

int main(void) {
  uint16_t led = PIN('B', 7);                 // Blue LED
  gpio_set_mode(led, GPIO_MODE_OUTPUT);       // Set blue LED to output mode
  uart_init(UART_DEBUG, 115200);              // Initialise UART
  volatile uint32_t timer = 0, period = 500;  // Declare timers
  for (;;) {
    if (timer_expired(&timer, period, s_ticks)) {
      static bool on;       // This block is executed
      gpio_write(led, on);  // Every `period` milliseconds
      on = !on;             // Toggle LED state
      printf("LED: %d, tick: %lu\r\n", on, s_ticks);  // Write message
    }
    // Here we could perform other activities!
  }
  return 0;
}
```

### syscalls.c

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#include <sys/stat.h>

#include "hal.h"

int _fstat(int fd, struct stat *st) {
  if (fd < 0) return -1;
  st->st_mode = S_IFCHR;
  return 0;
}

void *_sbrk(int incr) {
  extern char _end;
  static unsigned char *heap = NULL;
  unsigned char *prev_heap;
  if (heap == NULL) heap = (unsigned char *) &_end;
  prev_heap = heap;
  heap += incr;
  return prev_heap;
}

int _open(const char *path) {
  (void) path;
  return -1;
}

int _close(int fd) {
  (void) fd;
  return -1;
}

int _isatty(int fd) {
  (void) fd;
  return 1;
}

int _lseek(int fd, int ptr, int dir) {
  (void) fd, (void) ptr, (void) dir;
  return 0;
}

void _exit(int status) {
  (void) status;
  for (;;) asm volatile("BKPT #0");
}

void _kill(int pid, int sig) {
  (void) pid, (void) sig;
}

int _getpid(void) {
  return -1;
}

int _write(int fd, char *ptr, int len) {
  (void) fd, (void) ptr, (void) len;
  if (fd == 1) uart_write_buf(UART_DEBUG, ptr, (size_t) len);
  return -1;
}

int _read(int fd, char *ptr, int len) {
  (void) fd, (void) ptr, (void) len;
  return -1;
}

int _link(const char *a, const char *b) {
  (void) a, (void) b;
  return -1;
}

int _unlink(const char *a) {
  (void) a;
  return -1;
}

int _stat(const char *path, struct stat *st) {
  (void) path, (void) st;
  return -1;
}

int mkdir(const char *path, mode_t mode) {
  (void) path, (void) mode;
  return -1;
}

void _init(void) {
}
```

### link.ld

注意这里 `ENTRY(Reset_Handler)`，且 `.vectors` 使用 `KEEP(*(.isr_vector))`（对应 ST 启动文件中的向量表区段）。

```ld
ENTRY(Reset_Handler);
MEMORY {
  flash(rx)  : ORIGIN = 0x08000000, LENGTH = 2048k
  sram(rwx) : ORIGIN = 0x20000000, LENGTH = 192k  /* remaining 64k in a separate address space */
}
_estack     = ORIGIN(sram) + LENGTH(sram);    /* stack points to end of SRAM */

SECTIONS {
  .vectors  : { KEEP(*(.isr_vector)) }   > flash
  .text     : { *(.text*) }              > flash
  .rodata   : { *(.rodata*) }            > flash

  .data : {
    _sdata = .;
    *(.first_data)
    *(.data SORT(.data.*))
    _edata = .;
  } > sram AT > flash
  _sidata = LOADADDR(.data);

  .bss : { _sbss = .; *(.bss SORT(.bss.*) COMMON) _ebss = .; } > sram

  . = ALIGN(8);
  _end = .;
}
```

### Makefile

新增了 `-Icmsis_core/CMSIS/Core/Include -Icmsis_f4/Include` 头文件路径、ST 启动文件，以及 `cmsis_core`/`cmsis_f4` 的 git clone 目标。

```make
CFLAGS  ?=  -W -Wall -Wextra -Werror -Wundef -Wshadow -Wdouble-promotion \
            -Wformat-truncation -fno-common -Wconversion \
            -g3 -Os -ffunction-sections -fdata-sections \
            -I. -Iinclude -Icmsis_core/CMSIS/Core/Include -Icmsis_f4/Include \
            -mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 $(EXTRA_CFLAGS)
LDFLAGS ?= -Tlink.ld -nostartfiles -nostdlib --specs nano.specs -lc -lgcc -Wl,--gc-sections -Wl,-Map=$@.map
SOURCES = main.c syscalls.c
SOURCES += cmsis_f4/Source/Templates/gcc/startup_stm32f429xx.s # ST startup file. Compiler-dependent!

ifeq ($(OS),Windows_NT)
  RM = cmd /C del /Q /F
else
  RM = rm -rf
endif

build: firmware.bin

firmware.elf: cmsis_core cmsis_f4 hal.h link.ld Makefile $(SOURCES) 
	arm-none-eabi-gcc $(SOURCES) $(CFLAGS) $(CFLAGS_EXTRA) $(LDFLAGS) -o $@

firmware.bin: firmware.elf
	arm-none-eabi-objcopy -O binary $< $@

flash: firmware.bin
	st-flash --reset write $< 0x8000000

cmsis_core:
	git clone -q -c advice.detachedHead=false --depth 1 -b 5.9.0 https://github.com/ARM-software/CMSIS_5 $@

cmsis_f4:
	git clone -q -c advice.detachedHead=false --depth 1 -b v2.6.8 https://github.com/STMicroelectronics/cmsis_device_f4 $@

clean:
	$(RM) firmware.* cmsis_*
```

> `step-5-cmsis` 使用 ST 提供的 `startup_stm32f429xx.s` 启动文件，因此不再需要手写 `startup.c`。此时 `main.c` 通过 `SystemInit()`（由启动文件自动调用）初始化时钟。

## 下一步

此时系统仍运行在默认的 16MHz。接下来配置 PLL 把时钟拉高到 180MHz，见 [[11-时钟配置]]。
