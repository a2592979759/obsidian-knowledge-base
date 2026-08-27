---
tags:
  - embedded
  - 裸机编程
  - 单片机组
created: 2026-08-27
source: bare-metal-programming-guide/README_zh-CN.md
---

# 08 · printf 重定向

> 前置知识：[[07-UART串口]]。本节对应的完整工程位于 `steps/step-4-printf/`。

## 目标：格式化输出

在这一节，我们将 `uart_write_buf()` 调用替换为 `printf()`，它使我们能够进行格式化输出，这样可以更好的输出诊断信息，实现了"打印样式的调试"。

我们使用的 GNU ARM 工具链除了包含 GCC 编译器和一些工具外，还包含了一个被称为 [newlib](https://sourceware.org/newlib) 的 C 库，由红帽为嵌入式系统开发。

如果我们的固件调用了一个标准 C 库函数，比如 `strcmp()`，newlib 就会被 GCC 链接器加到我们的固件中。

newlib 实现了一些标准 C 函数，特别是文件输入输出操作，并且被实现的很随潮流：这些函数最终调用一组被称为 "syscalls" 的底层输入输出函数。

例如：

- `fopen()` 最终调用 `_open()`
- `fread()` 最终调用 `_read()`
- `fwrite()`、`fprintf()`、`printf()` 最终调用 `_write()`
- `malloc` 最终调用 `_sbrk()`，等等

因此，通过修改 `_write()` 系统调用，我们可以重定向 `printf()` 到任何我们希望的地方，这个机制被称为 "IO retargeting"。

> 注意：STM32 Cube 也使用 ARM GCC 工具链，这就是为什么 Cube 工程都包含 `syscalls.c` 文件。其它工具链，比如 TI 的 CCS、Keil 的 CC，可能使用不同的 C 库，重定向机制会有一点区别。我们用 newlib，所以修改 `_write()` 可以打印到串口 3。

## 重新组织源码结构

在与重定向之前，我们先重新组织下源码结构：

- 把所有 API 定义放到 `hal.h` 文件中
- 把启动代码放到 `startup.c` 文件中
- 为 newlib 的系统调用创建一个空文件 `syscalls.c`
- 修改 Makefile，把 `syscalls.c` 和 `startup.c` 加到 build 中

将所有 API 定义移动到 `hal.h` 后，`main.c` 文件变得相当紧凑。注意我们还没提到底层寄存器，高级 API 函数很容易理解：

```c
#include "hal.h"

static volatile uint32_t s_ticks;
void SysTick_Handler(void) {
  s_ticks++;
}

int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  systick_init(16000000 / 1000);         // Tick every 1 ms
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  uart_init(UART3, 115200);              // Initialise UART
  uint32_t timer = 0, period = 500;      // Declare timer and 500ms period
  for (;;) {
    if (timer_expired(&timer, period, s_ticks)) {
      static bool on;                      // This block is executed
      gpio_write(led, on);                 // Every `period` milliseconds
      on = !on;                            // Toggle LED state
      uart_write_buf(UART3, "hi\r\n", 4);  // Write message
    }
    // Here we could perform other activities!
  }
  return 0;
}
```

## 修改 _write()

现在我们把 `printf()` 重定向到串口 3，在空的 `syscalls.c` 文件中拷入一下内容：

```c
#include "hal.h"

int _write(int fd, char *ptr, int len) {
  (void) fd, (void) ptr, (void) len;
  if (fd == 1) uart_write_buf(UART3, ptr, (size_t) len);
  return -1;
}
```

这段代码：如果我们写入的文件描述符是 1（这是一个标准输出描述符），则将缓冲区写入串口 3，否则忽视。这就是重定向的本质！

## 补齐其余 syscalls

重新编译，会得到一些链接器错误：

```sh
../../arm-none-eabi/lib/thumb/v7e-m+fp/hard/libc_nano.a(lib_a-sbrkr.o): in function `_sbrk_r':
sbrkr.c:(.text._sbrk_r+0xc): undefined reference to `_sbrk'
closer.c:(.text._close_r+0xc): undefined reference to `_close'
lseekr.c:(.text._lseek_r+0x10): undefined reference to `_lseek'
readr.c:(.text._read_r+0x10): undefined reference to `_read'
fstatr.c:(.text._fstat_r+0xe): undefined reference to `_fstat'
isattyr.c:(.text._isatty_r+0xc): undefined reference to `_isatty'
```

这是因为我们使用了 newlib 的标准输入输出函数，那么就需要把 newlib 中其它的系统调用也实现。除了 `_sbrk` 之外我们可以加入一些简单的什么都不做的桩函数。而 `_sbrk` 会在 `printf()` 函数中使用 `malloc()` 的时候被调用到，所以它需要被实现：

```c
int _fstat(int fd, struct stat *st) {
  (void) fd, (void) st;
  return -1;
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

int _close(int fd) {
  (void) fd;
  return -1;
}

int _isatty(int fd) {
  (void) fd;
  return 1;
}

int _read(int fd, char *ptr, int len) {
  (void) fd, (void) ptr, (void) len;
  return -1;
}

int _lseek(int fd, int ptr, int dir) {
  (void) fd, (void) ptr, (void) dir;
  return 0;
}
```

再重新编译，应该就不会报错了。

## 替换为 printf

最后一步，将 `main()` 中 `uart_write_buf()` 替换为 `printf()`，并打印一些有用的信息，比如 LED 状态和当前 `s_ticks` 的值：

```c
printf("LED: %d, tick: %lu\r\n", on, s_ticks);  // Write message
```

再重新编译，串口输出应该像这样：

```sh
LED: 1, tick: 250
LED: 0, tick: 500
LED: 1, tick: 750
LED: 0, tick: 1000
```

可喜可贺！我们学习了 IO 重定向是如何工作的，并且可以用打印输出来调试固件了。

## 源码整合：step-4-printf

### hal.h

所有的外设 API 定义都集中到这里。

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#pragma once

#include <inttypes.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>

#define FREQ 16000000  // CPU frequency, 16 Mhz
#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)

static inline void spin(volatile uint32_t count) {
  while (count--) (void) 0;
}

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
  RCC->AHB1ENR |= BIT(PINBANK(pin));       // Enable GPIO clock
  gpio->MODER &= ~(3U << (n * 2));         // Clear existing setting
  gpio->MODER |= (mode & 3U) << (n * 2);   // Set new mode
}

static inline void gpio_set_af(uint16_t pin, uint8_t af_num) {
  struct gpio *gpio = GPIO(PINBANK(pin));  // GPIO bank
  int n = PINNO(pin);                      // Pin number
  gpio->AFR[n >> 3] &= ~(15UL << ((n & 7) * 4));
  gpio->AFR[n >> 3] |= ((uint32_t) af_num) << ((n & 7) * 4);
}

static inline void gpio_write(uint16_t pin, bool val) {
  struct gpio *gpio = GPIO(PINBANK(pin));
  gpio->BSRR = (1U << PINNO(pin)) << (val ? 0 : 16);
}

struct uart {
  volatile uint32_t SR, DR, BRR, CR1, CR2, CR3, GTPR;
};
#define UART1 ((struct uart *) 0x40011000)
#define UART2 ((struct uart *) 0x40004400)
#define UART3 ((struct uart *) 0x40004800)

static inline void uart_init(struct uart *uart, unsigned long baud) {
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

static inline void uart_write_byte(struct uart *uart, uint8_t byte) {
  uart->DR = byte;
  while ((uart->SR & BIT(7)) == 0) spin(1);
}

static inline void uart_write_buf(struct uart *uart, char *buf, size_t len) {
  while (len-- > 0) uart_write_byte(uart, *(uint8_t *) buf++);
}

static inline int uart_read_ready(struct uart *uart) {
  return uart->SR & BIT(5);  // If RXNE bit is set, data is ready
}

static inline uint8_t uart_read_byte(struct uart *uart) {
  return (uint8_t) (uart->DR & 255);
}

// t: expiration time, prd: period, now: current time. Return true if expired
static inline bool timer_expired(uint32_t *t, uint32_t prd, uint32_t now) {
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

int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  systick_init(FREQ / 1000);             // Tick every 1 ms
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  uart_init(UART3, 115200);              // Initialise UART
  uint32_t timer = 0, period = 500;      // Declare timer and 500ms period
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

### startup.c

启动代码从 `main.c` 独立出来。

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

// Startup code
__attribute__((naked, noreturn)) void _reset(void) {
  // Initialise memory
  extern long _sbss, _ebss, _sdata, _edata, _sidata;
  for (long *dst = &_sbss; dst < &_ebss; dst++) *dst = 0;
  for (long *dst = &_sdata, *src = &_sidata; dst < &_edata;) *dst++ = *src++;

  // Call main()
  extern void main(void);
  main();
  for (;;) (void) 0;  // Infinite loop
}

extern void SysTick_Handler(void);  // Defined in main.c
extern void _estack(void);          // Defined in link.ld

// 16 standard and 91 STM32-specific handlers
__attribute__((section(".vectors"))) void (*const tab[16 + 91])(void) = {
    _estack, _reset, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, SysTick_Handler};
```

### syscalls.c

newlib 所需的全部 syscalls（`_write` 是重定向的关键）。

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
  if (fd == 1) uart_write_buf(UART3, ptr, (size_t) len);
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
```

### Makefile

`SOURCES` 新增了 `startup.c` 和 `syscalls.c`。

```make
CFLAGS  ?=  -W -Wall -Wextra -Werror -Wundef -Wshadow -Wdouble-promotion \
            -Wformat-truncation -fno-common -Wconversion \
            -g3 -Os -ffunction-sections -fdata-sections -I. \
            -mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 $(EXTRA_CFLAGS)
LDFLAGS ?= -Tlink.ld -nostartfiles -nostdlib --specs nano.specs -lc -lgcc -Wl,--gc-sections -Wl,-Map=$@.map
SOURCES = main.c startup.c syscalls.c

ifeq ($(OS),Windows_NT)
  RM = cmd /C del /Q /F
else
  RM = rm -f
endif

build: firmware.bin

firmware.elf: $(SOURCES)
	arm-none-eabi-gcc $(SOURCES) $(CFLAGS) $(LDFLAGS) -o $@

firmware.bin: firmware.elf
	arm-none-eabi-objcopy -O binary $< $@

flash: firmware.bin
	st-flash --reset write $< 0x8000000

clean:
	$(RM) firmware.*
```

> `step-4-printf` 的 `link.ld` 与 `step-0-minimal` 相同。

## 下一步

现在有了 printf 调试，但如果固件卡住或启动代码有问题呢？我们需要调试器。见 [[09-调试]]。
