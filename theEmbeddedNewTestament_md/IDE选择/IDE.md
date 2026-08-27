---
tags:
  - IDE选择
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/IDE_Selection/IDE.md
created: 2026-08-27
---

## 关于嵌入式 IDE 及其功能(All about Embedded IDEs and their features)

### ***完全抽象的 IDE(Fully Abstracted IDE)***
#### **STM32CubeIDE**
用于 STM32 MCU 的 IDE。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux、macOS
调试器支持(Debugger Support) | GDB、STLink、JLink、JTrace 等
IDE 框架(IDE Framework) | Eclipse
编译器(Compiler) | GCC、可扩展(Extensible)
成本(Cost) | 免费(Free)
许可(License) | 专有-免费软件(Proprietary-freeware)

#### **ARM Mbed Studio**
一个面向物联网(IoT)的平台，提供非常大的中间件库以及在许多不同硬件厂商之间一致的开发环境。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、macOS 或在线（Mbed 在线）
调试器支持(Debugger Support) | PyOCD（用于有限的图形调试）或 GDB（仅控制台）
IDE 框架(IDE Framework) | Theia
编译器(Compiler) | ARM Compiler 6、GCC 和 IAR
成本(Cost) | 免费(Free)
许可(License) | Apache2.0

#### **Arduino IDE**
Arduino IDE 使用严格结构化的库，为用户提供 C/C++ 的一个方言子集来编写草图(sketches)。它尽可能在库内隐藏底层硬件的细节。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、macOS、Linux 或在线
调试器支持(Debugger Support) | 无(None)
IDE 框架(IDE Framework) | 专有的 Java、processing
编译器(Compiler) | ARM Compiler 6、GCC 和 IAR
成本(Cost) | 免费(Free)
许可(License) | GNU

### ***开源/免费 IDE(Open source/free IDEs)***
#### **AC6 System Workbench for STM32（S4STM32）**
AC6 是一家咨询公司，贡献了一个基于 Eclipse 的 IDE，面向 STM32 MCU。System Workbench 增加了一些对基于 STM 的探索板的支持，以帮助快速设置项目。如果你在用多核设备（STM32MP1 系列）开发应用，这很有用。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux、macOS
调试器支持(Debugger Support) | GDB
IDE 框架(IDE Framework) | Eclipse
编译器(Compiler) | GCC
成本(Cost) | 免费(Free)
许可(License) | 专有-免费软件(Proprietary-freeware)

#### **Eclipse CDT 和 GCC**
Eclipse CDT 是 C/C++ 开发的事实标准(de facto standard)。你需要自己提供一个编译器。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux、macOS
调试器支持(Debugger Support) | GDB
IDE 框架(IDE Framework) | Eclipse
编译器(Compiler) | GCC
成本(Cost) | 免费(Free)
许可(License) | Eclipse 公共许可(EPL)

#### **Visual Studio Code**
一个提供添加扩展能力的文本编辑器。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux、macOS
调试器支持(Debugger Support) | GDB、ST-link 等
IDE 框架(IDE Framework) | VScode
编译器(Compiler) | 多种(Many)
成本(Cost) | 免费(Free)
许可(License) | MIT

### ***专有 IDE(Proprietary IDE)***
#### **ARM/Keil uVision**
| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows
调试器支持(Debugger Support) | 多种(Many)
IDE 框架(IDE Framework) | 专有(Proprietary)
编译器(Compiler) | armcc、armClang、GCC
成本(Cost) | 免费-$$$
许可(License) | 专有(Proprietary)

#### **IAR Embedded Workbench**
| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows
调试器支持(Debugger Support) | 多种(Many)
IDE 框架(IDE Framework) | 专有(Proprietary)
编译器(Compiler) | 专有(Proprietary)
成本(Cost) | $$-$$$
许可(License) | 专有(Proprietary)

#### **Rowley CrossWorks**
比 Keil 和 IAR 稍微低一点的入门价格点。中间件从 IDE 单独授权。IDE 内不提供支持 FreeRTOS 感知的任务调试。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux 或 macOS
调试器支持(Debugger Support) | 多种(Many)
IDE 框架(IDE Framework) | 专有(Proprietary)
编译器(Compiler) | GCC、LLVM
成本(Cost) | $-$$$
许可(License) | 专有(Proprietary)

#### **SEGGER Embedded Studio**
非商业使用免费。中间件栈从 IDE 单独授权。IDE 内提供支持 FreeRTOS 感知的调试。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux 或 macOS
调试器支持(Debugger Support) | SEGGER
IDE 框架(IDE Framework) | 专有(Proprietary)
编译器(Compiler) | GCC、LLVM
成本(Cost) | $$-$$$，非商业使用免费
许可(License) | 专有(Proprietary)

#### **SysProgs Visual GDB**
其实不是一个 IDE，而是适用于 Visual Studio code 的插件。其目的是为与 GDB 调试器和 GNU make 工具交互提供一致的 UI（Visual Studio）。

| 字段(Field) | 支持项目(Supported Item)
----------------|-------
主机操作系统(Host OS) | Windows、Linux 或 macOS
调试器支持(Debugger Support) | 是(Yes)
IDE 框架(IDE Framework) | Visual Studio/code
编译器(Compiler) | GCC、ARM
多核调试(Multi-core debug) | 是(Yes)
成本(Cost) | $
许可(License) | 专有(Proprietary)

### IDE 特性比较表

| IDE | 免费? | 无代码大小限制 | J-link 支持 | FreeRTOS 内核感知调试 | 多平台
----------------|---|---|---|---|----
Keil uVision | V | X | V | X | X
IAR | X | N/A | V | V | X
Visual GDB | X | N/A | V | V | V
Rowley CrossWorks | X | N/A | V | V | V
VS Code | V | V | V | V | V
Eclipse CDT | V | V | V | V | V
AC6 S4STM32 | V | V | V | V | V
Arduino IDE | V | V | V | X | V
ARM MBed Studio | V | V | V | X | V
STM32CubeIDE | V | V | V | V | V
SEGGER Embedded Studio | V | V | V | V | V
