---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Freertos/firmware_stack.md
created: 2026-08-27
---

## Freertos 固件栈

### 固件栈分层示意图

![[_assets/FreeRTOS_firmware_stack.png]]

- 无论底层硬件移植实现如何，用户代码都可以访问相同的 FreeRTOS API。
- FreeRTOS 并__不__阻止__用户代码__使用厂商提供的驱动、CMSIS 或原始硬件寄存器。
