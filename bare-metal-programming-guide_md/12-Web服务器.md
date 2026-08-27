---
tags:
  - embedded
  - 裸机编程
  - 单片机组
created: 2026-08-27
source: bare-metal-programming-guide/README_zh-CN.md
---

# 12 · Web 服务器

> 前置知识：[[10-CMSIS]]、[[11-时钟配置]]。本节对应的完整工程位于 `steps/step-7-webserver/`（以 `nucleo-f429zi` 板卡为主）。

## 以太网硬件：MAC 与 PHY

Nucleo-F429ZI 带有板载以太网。以太网硬件需要两个组件：PHY（向铜缆、光缆等介质发送和接收电信号）和 MAC（驱动 PHY 控制器）。在我们的 Nucleo 开发板上，MAC 控制器是 MCU 内置的，PHY 是外部的（具体来说，是 Microchip 的 LAN8720a）。

MAC 和 PHY 可以用多个接口通信，我们将使用 RMII。为此，一些引脚必须配置为使用其替代功能（AF）。要实现 Web 服务器，我们需要 3 个软件组件：

- 网络驱动程序，用于向 MAC 控制器发送/接收以太网帧
- 一个网络堆栈，用于解析帧并理解 TCP/IP
- 理解 HTTP 的网络库

我们将使用[猫鼬网络库](https://github.com/cesanta/mongoose)，它在单个文件中实现所有这些。这是一个双重许可的库（GPLv2/商业），旨在使网络嵌入式开发快速简便。

## 引入 Mongoose 与示例

先拷贝 [mongoose.c](https://raw.githubusercontent.com/cesanta/mongoose/master/mongoose.c) 和 [mongoose.h](https://raw.githubusercontent.com/cesanta/mongoose/master/mongoose.h) 到我们的工程中，现在我们手上有网络驱动、网络协议栈和 HTTP 库了。Mongoose 还提供了很多示例，其中之一是[设备仪表盘示例](https://github.com/cesanta/mongoose/tree/master/examples/device-dashboard)。这个示例实现了很多事情，像登录、通过 WebSocket 实时传输数据、嵌入式文件系统、MQTT 通信等等，我们就使用这个例子，再拷贝 2 个文件：

- [net.c](https://raw.githubusercontent.com/cesanta/mongoose/master/examples/device-dashboard/net.c)，实现了仪表盘功能
- [packed_fs.c](https://raw.githubusercontent.com/cesanta/mongoose/master/examples/device-dashboard/packed_fs.c)，包含了 HTML/CSS/JS GUI 文件

我们需要告诉 Mongoose 开启哪些功能，可以通过设置预处理常数等编译器标记实现，也可以在 `mongoose_custom.h` 文件中设置。我们用第二种方法，创建 `mongoose_custom.h` 文件并写入以下内容：

```c
#pragma once
#define MG_ARCH MG_ARCH_NEWLIB
#define MG_ENABLE_MIP 1
#define MG_ENABLE_PACKED_FS 1
#define MG_IO_SIZE 512
#define MG_ENABLE_CUSTOM_MILLIS 1
```

## 初始化以太网与 Mongoose

现在向 `main.c` 添加一些网络代码，`#include "mongoose.c"` 初始化以太网 RMII 引脚，并在 RCC 中使能以太网：

```c
  uint16_t pins[] = {PIN('A', 1),  PIN('A', 2),  PIN('A', 7),
                     PIN('B', 13), PIN('C', 1),  PIN('C', 4),
                     PIN('C', 5),  PIN('G', 11), PIN('G', 13)};
  for (size_t i = 0; i < sizeof(pins) / sizeof(pins[0]); i++) {
    gpio_init(pins[i], GPIO_MODE_AF, GPIO_OTYPE_PUSH_PULL, GPIO_SPEED_INSANE,
              GPIO_PULL_NONE, 11);
  }
  nvic_enable_irq(61);                          // Setup Ethernet IRQ handler
  RCC->APB2ENR |= BIT(14);                      // Enable SYSCFG
  SYSCFG->PMC |= BIT(23);                       // Use RMII. Goes first!
  RCC->AHB1ENR |= BIT(25) | BIT(26) | BIT(27);  // Enable Ethernet clocks
  RCC->AHB1RSTR |= BIT(25);                     // ETHMAC force reset
  RCC->AHB1RSTR &= ~BIT(25);                    // ETHMAC release reset
```

## 弱符号与向量表

Mongoose 的驱动程序使用以太网中断，因此我们需要更新 `startup.c` 并将 `ETH_IRQHandler` 添加到向量表中。让我们以不需要任何修改就能添加中断处理函数的方式重新组织 `startup.c` 中的向量表定义，方法是使用"弱符号"概念。

函数可以标记为"弱"，它的工作方式与普通函数类似。当源代码定义具有相同名称的函数时，差异就来了。通常，两个同名的函数会构建失败。但是，如果一个函数被标记为弱函数，则可以构建成功并且链接器会选择非弱函数。这提供了设置样板中的函数为"默认函数"的能力，然后可以在代码中的其他位置简单地创建一个同名函数来覆盖它。

我们接下来用这种方法填充向量表，创建一个 `DefaultIRQHandler()` 并标记为 weak，然后给每一个中断处理函数声明一个处理函数名并使它成为 `DefaultIRQHandler()` 的别名：

```c
void __attribute__((weak)) DefaultIRQHandler(void) {
  for (;;) (void) 0;
}
#define WEAK_ALIAS __attribute__((weak, alias("DefaultIRQHandler")))

WEAK_ALIAS void NMI_Handler(void);
WEAK_ALIAS void HardFault_Handler(void);
WEAK_ALIAS void MemManage_Handler(void);
...
__attribute__((section(".vectors"))) void (*tab[16 + 91])(void) = {
    0, _reset, NMI_Handler, HardFault_Handler, MemManage_Handler,
    ...
```

现在，我们可以在代码中定义任何中断处理函数，它会替代默认的那个。这就是我们的例子中所发生的：Mongoose 的 STM32 驱动中定义了一个 `ETH_IRQHandler()`，它会替代默认的中断处理函数。

## MIP 网络栈初始化

下一步是初始化 Mongoose 库：创建时间管理器、配置网络驱动、启动监听 HTTP 连接：

```c
  struct mg_mgr mgr;        // Initialise Mongoose event manager
  mg_mgr_init(&mgr);        // and attach it to the MIP interface
  mg_log_set(MG_LL_DEBUG);  // Set log level

  struct mip_driver_stm32 driver_data = {.mdc_cr = 4};  // See driver_stm32.h
  struct mip_if mif = {
      .mac = {2, 0, 1, 2, 3, 5},
      .use_dhcp = true,
      .driver = &mip_driver_stm32,
      .driver_data = &driver_data,
  };
  mip_init(&mgr, &mif);
  extern void device_dashboard_fn(struct mg_connection *, int, void *, void *);
  mg_http_listen(&mgr, "http://0.0.0.0", device_dashboard_fn, &mgr);
  MG_INFO(("Init done, starting main loop"));
```

剩下的就是把 `mg_mgr_poll()` 调用加到主循环。

## 构建、烧写与连接

现在把 `mongoose.c`、`net.c` 和 `packed_fs.c` 文件加到 Makefile，重新构建，烧写到板子上。连接一个串口控制台到调试输出，可以观察到板子通过 DHCP 获取了 IP 地址：

```
847 3 mongoose.c:6784:arp_cache_add     ARP cache: added 0xc0a80001 @ 90:5c:44:55:19:8b
84e 2 mongoose.c:6817:onstatechange     READY, IP: 192.168.0.24
854 2 mongoose.c:6818:onstatechange            GW: 192.168.0.1
859 2 mongoose.c:6819:onstatechange            Lease: 86363 sec
LED: 1, tick: 2262
LED: 0, tick: 2512
```

打开一个浏览器，输入上面的 IP 地址，就可以看到一个仪表盘。在[full description](https://github.com/cesanta/mongoose/tree/master/examples/device-dashboard)获取更多细节。

## 源码整合：step-7-webserver/nucleo-f429zi

> `mongoose.c`、`mongoose.h`、`net.c`、`packed_fs.c` 体积很大，此处不再贴出完整内容，可到仓库对应目录查看。核心自定义文件如下。

### main.c

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#include "hal.h"
#include "mongoose.h"

static volatile uint32_t s_ticks;
void SysTick_Handler(void) { s_ticks++; }     // IRQ handler
uint64_t mg_millis(void) { return s_ticks; }  // For Mongoose

int main(void) {
  uint16_t led = PIN('B', 7);            // Blue LED
  clock_init();                          // Run at 180Mhz
  systick_init(SYS_FREQUENCY / 1000);    // Tick every 1 ms
  gpio_set_mode(led, GPIO_MODE_OUTPUT);  // Set blue LED to output mode
  uart_init(UART3, 115200);              // Initialise UART
  uint32_t timer = 0, period = 500;      // Declare timer and 500ms period

  // Initialise Ethernet. Enable MAC GPIO pins, see
  // https://www.farnell.com/datasheets/2014265.pdf section 6.10
  uint16_t pins[] = {PIN('A', 1),  PIN('A', 2),  PIN('A', 7),
                     PIN('B', 13), PIN('C', 1),  PIN('C', 4),
                     PIN('C', 5),  PIN('G', 11), PIN('G', 13)};
  for (size_t i = 0; i < sizeof(pins) / sizeof(pins[0]); i++) {
    gpio_init(pins[i], GPIO_MODE_AF, 0, GPIO_SPEED_INSANE, GPIO_PULL_NONE, 11);
  }
  NVIC_EnableIRQ(ETH_IRQn);                     // Setup Ethernet IRQ handler
  RCC->APB2ENR |= BIT(14);                      // Enable SYSCFG
  SYSCFG->PMC |= BIT(23);                       // Use RMII. Goes first!
  RCC->AHB1ENR |= BIT(25) | BIT(26) | BIT(27);  // Enable Ethernet clocks
  RCC->AHB1RSTR |= BIT(25);                     // ETHMAC force reset
  RCC->AHB1RSTR &= ~BIT(25);                    // ETHMAC release reset

  struct mg_mgr mgr;        // Initialise Mongoose event manager
  mg_mgr_init(&mgr);        // and attach it to the MIP interface
  mg_log_set(MG_LL_DEBUG);  // Set log level

  // Initialise Mongoose network stack
  // Specify MAC address, and use 0 for IP, mask, GW - i.e. use DHCP
  // For static configuration, specify IP/mask/GW in network byte order
  struct mip_driver_stm32 driver_data = {.mdc_cr = 4};  // See driver_stm32.h
  struct mip_if mif = {
      .mac = {2, 0, 1, 2, 3, 5},
      .use_dhcp = true,
      .driver = &mip_driver_stm32,
      .driver_data = &driver_data,
  };
  mip_init(&mgr, &mif);
  extern void device_dashboard_fn(struct mg_connection *, int, void *, void *);
  mg_http_listen(&mgr, "http://0.0.0.0", device_dashboard_fn, &mgr);
  MG_INFO(("Init done, starting main loop"));

  for (;;) {
    if (timer_expired(&timer, period, s_ticks)) {
      static bool on;       // This block is executed
      gpio_write(led, on);  // Every `period` milliseconds
      on = !on;             // Toggle LED state
      printf("LED: %d, tick: %lu\r\n", on, s_ticks);  // Write message
    }
    mg_mgr_poll(&mgr, 0);  // Handle networking
  }
  return 0;
}
```

### mongoose_custom.h

```c
// Copyright (c) 2022 Cesanta Software Limited
// All rights reserved

#pragma once
#define MG_ARCH MG_ARCH_NEWLIB
#define MG_ENABLE_MIP 1
#define MG_ENABLE_PACKED_FS 1
#define MG_IO_SIZE 512
#define MG_ENABLE_CUSTOM_MILLIS 1
```

### hal.h

承继自 [[11-时钟配置]] 的 `hal.h`，并新增 `clock_init()`（配置 180MHz，见 [[11-时钟配置]]），同时保留 `data` 相关的 `systick_init`、`gpio_init`、`uart_init` 等。

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

#define BIT(x) (1UL << (x))
#define PIN(bank, num) ((((bank) - 'A') << 8) | (num))
#define PINNO(pin) (pin & 255)
#define PINBANK(pin) (pin >> 8)
#define SETBITS(R, CLEARMASK, SETMASK) (R) = ((R) & ~(CLEARMASK)) | (SETMASK)

// 6.3.3: APB1 clock <= 45MHz; APB2 clock <= 90MHz
// 3.5.1, Table 11: configure flash latency (WS) in accordance to clock freq
// 33.4: The AHB clock must be at least 25 MHz when Ethernet is used
enum { APB1_PRE = 5 /* AHB clock / 4 */, APB2_PRE = 4 /* AHB clock / 2 */ };
enum { PLL_HSI = 16, PLL_M = 8, PLL_N = 180, PLL_P = 2 };  // Run at 180 Mhz
#define FLASH_LATENCY 5
#define SYS_FREQUENCY ((PLL_HSI * PLL_N / PLL_M / PLL_P) * 1000000)
#define APB2_FREQUENCY (SYS_FREQUENCY / (BIT(APB2_PRE - 3)))
#define APB1_FREQUENCY (SYS_FREQUENCY / (BIT(APB1_PRE - 3)))

static inline void spin(volatile uint32_t count) {
  while (count--) asm("nop");
}

static inline void systick_init(uint32_t ticks) {
  if ((ticks - 1) > 0xffffff) return;  // Systick timer is 24 bit
  SysTick->LOAD = ticks - 1;
  SysTick->VAL = 0;
  SysTick->CTRL = BIT(0) | BIT(1) | BIT(2);  // Enable systick
  RCC->APB2ENR |= BIT(14);                   // Enable SYSCFG
}

#define GPIO(bank) ((GPIO_TypeDef *) (GPIOA_BASE + 0x400U * (bank)))
enum { GPIO_MODE_INPUT, GPIO_MODE_OUTPUT, GPIO_MODE_AF, GPIO_MODE_ANALOG };
enum { GPIO_OTYPE_PUSH_PULL, GPIO_OTYPE_OPEN_DRAIN };
enum { GPIO_SPEED_LOW, GPIO_SPEED_MEDIUM, GPIO_SPEED_HIGH, GPIO_SPEED_INSANE };
enum { GPIO_PULL_NONE, GPIO_PULL_UP, GPIO_PULL_DOWN };

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

static inline void gpio_init(uint16_t pin, uint8_t mode, uint8_t type,
                             uint8_t speed, uint8_t pull, uint8_t af) {
  GPIO_TypeDef *gpio = GPIO(PINBANK(pin));  // GPIO bank
  uint8_t n = (uint8_t) (PINNO(pin));
  RCC->AHB1ENR |= BIT(PINBANK(pin));  // Enable GPIO clock
  SETBITS(gpio->OTYPER, 1UL << n, ((uint32_t) type) << n);
  SETBITS(gpio->OSPEEDR, 3UL << (n * 2), ((uint32_t) speed) << (n * 2));
  SETBITS(gpio->PUPDR, 3UL << (n * 2), ((uint32_t) pull) << (n * 2));
  SETBITS(gpio->AFR[n >> 3], 15UL << ((n & 7) * 4),
          ((uint32_t) af) << ((n & 7) * 4));
  SETBITS(gpio->MODER, 3UL << (n * 2), ((uint32_t) mode) << (n * 2));
}

#define UART1 USART1
#define UART2 USART2
#define UART3 USART3

static inline void uart_init(USART_TypeDef *uart, unsigned long baud) {
  // https://www.st.com/resource/en/datasheet/stm32f429zi.pdf
  uint8_t af = 7;           // Alternate function
  uint16_t rx = 0, tx = 0;  // pins
  uint32_t freq = 0;        // Bus frequency. UART1 is on APB2, rest on APB1

  if (uart == UART1) freq = APB2_FREQUENCY, RCC->APB2ENR |= BIT(4);
  if (uart == UART2) freq = APB1_FREQUENCY, RCC->APB1ENR |= BIT(17);
  if (uart == UART3) freq = APB1_FREQUENCY, RCC->APB1ENR |= BIT(18);

  if (uart == UART1) tx = PIN('A', 9), rx = PIN('A', 10);
  if (uart == UART2) tx = PIN('A', 2), rx = PIN('A', 3);
  if (uart == UART3) tx = PIN('D', 8), rx = PIN('D', 9);

  gpio_set_mode(tx, GPIO_MODE_AF);
  gpio_set_af(tx, af);
  gpio_set_mode(rx, GPIO_MODE_AF);
  gpio_set_af(rx, af);
  uart->CR1 = 0;                           // Disable this UART
  uart->BRR = freq / baud;                 // Set baud rate
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

static inline bool timer_expired(uint32_t *t, uint32_t prd, uint32_t now) {
  if (now + prd < *t) *t = 0;                    // Time wrapped? Reset timer
  if (*t == 0) *t = now + prd;                   // Firt poll? Set expiration
  if (*t > now) return false;                    // Not expired yet, return
  *t = (now - *t) > prd ? now + prd : *t + prd;  // Next expiration time
  return true;                                   // Expired, return true
}

static inline void clock_init(void) {                 // Set clock frequency
  SCB->CPACR |= ((3UL << 10 * 2) | (3UL << 11 * 2));  // Enable FPU
  FLASH->ACR |= FLASH_LATENCY | BIT(8) | BIT(9);      // Flash latency, caches
  RCC->PLLCFGR &= ~((BIT(17) - 1));                   // Clear PLL multipliers
  RCC->PLLCFGR |= (((PLL_P - 2) / 2) & 3) << 16;      // Set PLL_P
  RCC->PLLCFGR |= PLL_M | (PLL_N << 6);               // Set PLL_M and PLL_N
  RCC->CR |= BIT(24);                                 // Enable PLL
  while ((RCC->CR & BIT(25)) == 0) spin(1);           // Wait until done
  RCC->CFGR = (APB1_PRE << 10) | (APB2_PRE << 13);    // Set prescalers
  RCC->CFGR |= 2;                                     // Set clock source to PLL
  while ((RCC->CFGR & 12) == 0) spin(1);              // Wait until done
}
```

### startup.c

使用弱符号填充完整的中断向量表，使 `ETH_IRQHandler` 等可在别处被覆盖。

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

void __attribute__((weak)) DefaultIRQHandler(void) {
  for (;;) (void) 0;
}
#define WEAK_ALIAS __attribute__((weak, alias("DefaultIRQHandler")))

WEAK_ALIAS void NMI_Handler(void);
WEAK_ALIAS void HardFault_Handler(void);
WEAK_ALIAS void MemManage_Handler(void);
WEAK_ALIAS void BusFault_Handler(void);
WEAK_ALIAS void UsageFault_Handler(void);
WEAK_ALIAS void SVC_Handler(void);
WEAK_ALIAS void DebugMon_Handler(void);
WEAK_ALIAS void PendSV_Handler(void);
WEAK_ALIAS void SysTick_Handler(void);

WEAK_ALIAS void WWDG_IRQHandler(void);
WEAK_ALIAS void PVD_IRQHandler(void);
WEAK_ALIAS void TAMP_STAMP_IRQHandler(void);
WEAK_ALIAS void RTC_WKUP_IRQHandler(void);
WEAK_ALIAS void FLASH_IRQHandler(void);
WEAK_ALIAS void RCC_IRQHandler(void);
WEAK_ALIAS void EXTI0_IRQHandler(void);
WEAK_ALIAS void EXTI1_IRQHandler(void);
WEAK_ALIAS void EXTI2_IRQHandler(void);
WEAK_ALIAS void EXTI3_IRQHandler(void);
WEAK_ALIAS void EXTI4_IRQHandler(void);
WEAK_ALIAS void EXTI9_5_IRQHandler(void);
WEAK_ALIAS void EXTI15_10_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream0_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream1_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream2_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream3_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream4_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream5_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream6_IRQHandler(void);
WEAK_ALIAS void ADC_IRQHandler(void);
WEAK_ALIAS void CAN1_TX_IRQHandler(void);
WEAK_ALIAS void CAN1_RX0_IRQHandler(void);
WEAK_ALIAS void CAN1_RX1_IRQHandler(void);
WEAK_ALIAS void CAN1_SCE_IRQHandler(void);
WEAK_ALIAS void TIM1_BRK_TIM9_IRQHandler(void);
WEAK_ALIAS void TIM1_UP_TIM10_IRQHandler(void);
WEAK_ALIAS void TIM1_TRG_COM_TIM11_IRQHandler(void);
WEAK_ALIAS void TIM1_CC_IRQHandler(void);
WEAK_ALIAS void TIM2_IRQHandler(void);
WEAK_ALIAS void TIM3_IRQHandler(void);
WEAK_ALIAS void TIM4_IRQHandler(void);
WEAK_ALIAS void I2C1_EV_IRQHandler(void);
WEAK_ALIAS void I2C1_ER_IRQHandler(void);
WEAK_ALIAS void I2C2_EV_IRQHandler(void);
WEAK_ALIAS void I2C2_ER_IRQHandler(void);
WEAK_ALIAS void SPI1_IRQHandler(void);
WEAK_ALIAS void SPI2_IRQHandler(void);
WEAK_ALIAS void USART1_IRQHandler(void);
WEAK_ALIAS void USART2_IRQHandler(void);
WEAK_ALIAS void USART3_IRQHandler(void);
WEAK_ALIAS void RTC_Alarm_IRQHandler(void);
WEAK_ALIAS void OTG_FS_WKUP_IRQHandler(void);
WEAK_ALIAS void TIM8_BRK_TIM12_IRQHandler(void);
WEAK_ALIAS void TIM8_UP_TIM13_IRQHandler(void);
WEAK_ALIAS void TIM8_TRG_COM_TIM14_IRQHandler(void);
WEAK_ALIAS void TIM8_CC_IRQHandler(void);
WEAK_ALIAS void DMA1_Stream7_IRQHandler(void);
WEAK_ALIAS void FMC_IRQHandler(void);
WEAK_ALIAS void SDMMC1_IRQHandler(void);
WEAK_ALIAS void TIM5_IRQHandler(void);
WEAK_ALIAS void SPI3_IRQHandler(void);
WEAK_ALIAS void UART4_IRQHandler(void);
WEAK_ALIAS void UART5_IRQHandler(void);
WEAK_ALIAS void TIM6_DAC_IRQHandler(void);
WEAK_ALIAS void TIM7_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream0_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream1_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream2_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream3_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream4_IRQHandler(void);
WEAK_ALIAS void ETH_IRQHandler(void);
WEAK_ALIAS void ETH_WKUP_IRQHandler(void);
WEAK_ALIAS void CAN2_TX_IRQHandler(void);
WEAK_ALIAS void CAN2_RX0_IRQHandler(void);
WEAK_ALIAS void CAN2_RX1_IRQHandler(void);
WEAK_ALIAS void CAN2_SCE_IRQHandler(void);
WEAK_ALIAS void OTG_FS_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream5_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream6_IRQHandler(void);
WEAK_ALIAS void DMA2_Stream7_IRQHandler(void);
WEAK_ALIAS void USART6_IRQHandler(void);
WEAK_ALIAS void I2C3_EV_IRQHandler(void);
WEAK_ALIAS void I2C3_ER_IRQHandler(void);
WEAK_ALIAS void OTG_HS_EP1_OUT_IRQHandler(void);
WEAK_ALIAS void OTG_HS_EP1_IN_IRQHandler(void);
WEAK_ALIAS void OTG_HS_WKUP_IRQHandler(void);
WEAK_ALIAS void OTG_HS_IRQHandler(void);
WEAK_ALIAS void DCMI_IRQHandler(void);
WEAK_ALIAS void RNG_IRQHandler(void);
WEAK_ALIAS void FPU_IRQHandler(void);
WEAK_ALIAS void UART7_IRQHandler(void);
WEAK_ALIAS void UART8_IRQHandler(void);
WEAK_ALIAS void SPI4_IRQHandler(void);
WEAK_ALIAS void SPI5_IRQHandler(void);
WEAK_ALIAS void SPI6_IRQHandler(void);
WEAK_ALIAS void SAI1_IRQHandler(void);
WEAK_ALIAS void LTDC_IRQHandler(void);
WEAK_ALIAS void LTDC_ER_IRQHandler(void);
WEAK_ALIAS void DMA2D_IRQHandler(void);

extern void _estack(void);  // Defined in link.ld

// IRQ table
__attribute__((section(".vectors"))) void (*const tab[16 + 91])(void) = {
    // Cortex interrupts
    _estack, _reset, NMI_Handler, HardFault_Handler, MemManage_Handler,
    BusFault_Handler, UsageFault_Handler, 0, 0, 0, 0, SVC_Handler,
    DebugMon_Handler, 0, PendSV_Handler, SysTick_Handler,

    // Interrupts from peripherals
    WWDG_IRQHandler, PVD_IRQHandler, TAMP_STAMP_IRQHandler, RTC_WKUP_IRQHandler,
    FLASH_IRQHandler, RCC_IRQHandler, EXTI0_IRQHandler, EXTI1_IRQHandler,
    EXTI2_IRQHandler, EXTI3_IRQHandler, EXTI4_IRQHandler,
    DMA1_Stream0_IRQHandler, DMA1_Stream1_IRQHandler, DMA1_Stream2_IRQHandler,
    DMA1_Stream3_IRQHandler, DMA1_Stream4_IRQHandler, DMA1_Stream5_IRQHandler,
    DMA1_Stream6_IRQHandler, ADC_IRQHandler, CAN1_TX_IRQHandler,
    CAN1_RX0_IRQHandler, CAN1_RX1_IRQHandler, CAN1_SCE_IRQHandler,
    EXTI9_5_IRQHandler, TIM1_BRK_TIM9_IRQHandler, TIM1_UP_TIM10_IRQHandler,
    TIM1_TRG_COM_TIM11_IRQHandler, TIM1_CC_IRQHandler, TIM2_IRQHandler,
    TIM3_IRQHandler, TIM4_IRQHandler, I2C1_EV_IRQHandler, I2C1_ER_IRQHandler,
    I2C2_EV_IRQHandler, I2C2_ER_IRQHandler, SPI1_IRQHandler, SPI2_IRQHandler,
    USART1_IRQHandler, USART2_IRQHandler, USART3_IRQHandler,
    EXTI15_10_IRQHandler, RTC_Alarm_IRQHandler, OTG_FS_WKUP_IRQHandler,
    TIM8_BRK_TIM12_IRQHandler, TIM8_UP_TIM13_IRQHandler,
    TIM8_TRG_COM_TIM14_IRQHandler, TIM8_CC_IRQHandler, DMA1_Stream7_IRQHandler,
    FMC_IRQHandler, SDMMC1_IRQHandler, TIM5_IRQHandler, SPI3_IRQHandler,
    UART4_IRQHandler, UART5_IRQHandler, TIM6_DAC_IRQHandler, TIM7_IRQHandler,
    DMA2_Stream0_IRQHandler, DMA2_Stream1_IRQHandler, DMA2_Stream2_IRQHandler,
    DMA2_Stream3_IRQHandler, DMA2_Stream4_IRQHandler, ETH_IRQHandler,
    ETH_WKUP_IRQHandler, CAN2_TX_IRQHandler, CAN2_RX0_IRQHandler,
    CAN2_RX1_IRQHandler, CAN2_SCE_IRQHandler, OTG_FS_IRQHandler,
    DMA2_Stream5_IRQHandler, DMA2_Stream6_IRQHandler, DMA2_Stream7_IRQHandler,
    USART6_IRQHandler, I2C3_EV_IRQHandler, I2C3_ER_IRQHandler,
    OTG_HS_EP1_OUT_IRQHandler, OTG_HS_EP1_IN_IRQHandler, OTG_HS_WKUP_IRQHandler,
    OTG_HS_IRQHandler, DCMI_IRQHandler, 0, RNG_IRQHandler, FPU_IRQHandler,
    UART7_IRQHandler, UART8_IRQHandler, SPI4_IRQHandler, SPI5_IRQHandler,
    SPI6_IRQHandler, SAI1_IRQHandler, LTDC_IRQHandler, LTDC_ER_IRQHandler,
    DMA2D_IRQHandler};
```

### syscalls.c、Makefile

`syscalls.c` 与 [[11-时钟配置]] 相同，`_write()` 使用 `UART3`。Makefile 的 `SOURCES` 追加了 `mongose.c net.c packed_fs.c`：

```make
CFLAGS  ?=  -W -Wall -Wextra -Werror -Wundef -Wshadow -Wdouble-promotion \
            -Wformat-truncation -fno-common -Wconversion \
            -g3 -Os -ffunction-sections -fdata-sections -I. -Iinclude \
            -mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 $(EXTRA_CFLAGS)
LDFLAGS ?= -Tlink.ld -nostartfiles -nostdlib --specs nano.specs -lc -lgcc -Wl,--gc-sections -Wl,-Map=$@.map
SOURCES = main.c startup.c syscalls.c mongoose.c net.c packed_fs.c

ifeq ($(OS),Windows_NT)
  RM = cmd /C del /Q /F
else
  RM = rm -f
endif

build: firmware.elf

firmware.elf: $(SOURCES)
	arm-none-eabi-gcc $(SOURCES) $(CFLAGS) $(LDFLAGS) -o $@

firmware.bin: firmware.elf
	arm-none-eabi-objcopy -O binary $< $@

flash: firmware.bin
	st-flash --reset write firmware.bin 0x8000000

clean:
	$(RM) firmware.*
```

`link.ld` 与 [[03-启动代码与链接]] 中的版本相同。

### include 目录

存放 CMSIS 头文件：`stm32f429xx.h`、`system_stm32f4xx.h`、`core_cm4.h`、`cmsis_gcc.h`、`cmsis_version.h`、`cmsis_compiler.h`、`mpu_armv7.h`。

## 其他板卡适配

`steps/step-7-webserver/` 下还包含另外两种板卡：

- **ek-tm4c1294xl**：TI EK-TM4C1294XL（`hal.h`、`link.ld`、`main.c`、`startup.c` 等）
- **pico-w5500**：树莓派 Pico W5500（`hal.h`、`link.ld`、`main.c`、`startup.c` 及 `tools/`）

## 下一步

全部核心教程完成。可继续浏览 [[13-附录-工程模板]]，了解此仓库提供的其他板卡模板。
