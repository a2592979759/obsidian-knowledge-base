---
title: STM32 与嵌入式开发「全自动流程」
tags: [嵌入式, stm32, mcp, 全自动]
---

# STM32 与嵌入式开发「全自动流程」

## 一、Claude Code 能做什么（STM32 项目）

### 能做的
- 需求 → 架构设计：输出技术规格（MCU 选型、外设分配表、引脚规划、模块间接口、状态机设计）
- CubeMX 配置指导：告诉你 GUI 里每一步填什么（Pinout、Clock PLLM/N、NVIC、DMA/EXTI/TIM 参数）
- 代码：HAL 层之上的全部驱动逻辑（外设驱动、协议栈、业务逻辑、测试代码）

### 不能做的
- 替你点鼠标（CubeMX 没有 API）
- `.ioc` 文件按像素级修改（闭源格式）

### 实际操作流程
1. 你说需求 → 我出 spec（MCU 选型、外设分配、引脚清单）
2. 你打开 CubeMX 选 STM32F407VGT6
3. 我告诉你「搜 PB6→I2C1_SCL、PB7→I2C1_SDA、PB8→UART3_TX…」
4. 你照着点
5. 导出 `.ioc`，我 CLI 生成 HAL 骨架
6. 在骨架上写全部驱动代码

## 二、CubeMX 的正确心态（重要）
- **CubeMX 只管初始化代码，用一次生成完就别再打开**
- 正确流程：开 CubeMX → 配引脚/时钟 → 生成代码 → 提交 Git → 关掉 → 以后只改代码
- 而不是每天：改需求 → 开 CubeMX → 重配 → 重新生成 → 合并冲突
- 生成后就是普通 C 工程，可直接改外设初始化参数、写驱动、加中断、改引脚配置、重构
- **唯一坑**：若重新打开 CubeMX 点「Generate Code」，会覆盖不在 `/* USER CODE BEGIN */` 和 `/* USER CODE END */` 之间的代码。要么别再碰 CubeMX，要么改的内容都放 USER CODE 标记里

### CubeMX 生成的东西
```
Core/
├── Src/
│   ├── main.c            ← 你的逻辑
│   ├── gpio.c            ← 引脚初始化
│   ├── usart.c           ← 串口初始化
│   └── stm32f4xx_it.c    ← 中断处理
├── Inc/                  ← 头文件
└── Drivers/              ← HAL 库（不用改）
```

## 三、STM32 开发可用的 MCP Server

| MCP Server | 能干什么 | 状态 |
|---|---|---|
| shieldyguy/stm32-mcp | build / flash / debug（编译、烧录、调试） | ✅ 可用 |
| Adancurusul/embedded-debugger-mcp | 寄存器读写、内存查看、断点、AI 崩溃诊断（probe-rs/OpenOCD） | ✅ 可用 |
| okhsunrog/flashprobe-mcp | 烧录 + RTT 监控（probe-rs） | ✅ 可用 |
| royforlinux/openocd-mcu-mcp | OpenOCD 调试（断点、单步、寄存器读写） | ✅ 可用 |
| paulopalaoro/cortex-mcp-bridge | 在线调试状态读取（VSCode 扩展） | ✅ VSCode 集成 |

> 仍然没有：控制 CubeMX 的 MCP（原因：CubeMX 无公开 API）

### 实际可用的全链路
```
手写寄存器级代码 → gcc 编译 (shieldyguy/stm32-mcp)
                 → probe-rs 烧录 (flashprobe-mcp)
                 → 在线调试 (embedded-debugger-mcp)
                 → RTT 日志回读 (flashprobe-mcp)
```

### 全自动调试前提
1. 开发板连着调试器（ST-Link / J-Link / DAP-Link）
2. `arm-none-eabi-gcc` + probe-rs 或 openocd 已安装
3. 对应 MCP Server 已接入 Claude Code

接入后流程：写代码 → 编译 → 烧录 → 设断点 → 读寄存器/内存 → 看 RTT 日志 → 崩溃自动诊断，全流程不碰鼠标

## 四、PCB EDA 的 MCP Server

| EDA | MCP Server | 能力 |
|---|---|---|
| 立创 EDA (EasyEDA) | easyeda-mcp-pro、easyeda-copilot-mcp、easyeda-bridge-mcp | PCB 检查、BOM 采购、制造导出、AI 硬件审查 |
| Altium Designer | salitronic/eda-agent | 300+ 工具，DelphiScript 桥接 |
| KiCad | mixelpixx/KiCAD-MCP-Server、kicad-mcp 等 | 原理图/PCB、DRC/ERC、BOM、DFM、仿真 |
| Cadence | ❌ 没有 | 商业闭源，无公开 API |

### 总结：全自动控制链路
```
需求讨论 → 架构设计 (brainstorming)
        → STM32 代码生成 (手写寄存器级，全自动)
        → 编译/烧录/调试 (stm32-mcp + embedded-debugger-mcp)
        → PCB 设计 (EasyEDA 或 KiCad MCP，自动画板)
        → BOM/制造输出 (easyeda-mcp-pro / kicad-mcp)
```

## 五、Qt 相关的 MCP 服务

### Qt GUI 自动化控制
| 项目 | 说明 | GitHub |
|---|---|---|
| qt-pilot | headless Qt/PySide6 GUI 测试，AI 操控控件 | neatobandit0/qt-pilot |
| qt-mcp | 通用 Qt MCP，支持 PySide6 调试 | 0xCarbon/qt-mcp |
| qtmcp | Qt MCP Server，控制接口 | signal-slot/qtmcp |
| McpQTControl | 客户端控制 QT 程序（中文项目） | FlyMercurian/McpQTControl |

### Qt 官方/测试框架
| 项目 | 说明 | GitHub |
|---|---|---|
| squish-mcp | Qt 官方 Squish GUI 测试框架 MCP | TheQtCompanyRnD/squish-mcp |
| logos-qt-mcp | QML Inspector，检查/操控 QML 界面 | logos-co/logos-qt-mcp |

### Qt Creator IDE 插件
- qt-creator-mcp-plugin：让 AI 直接在 IDE 内操控 Qt Creator（davecotter/qt-creator-mcp-plugin）

> 推荐：qt-pilot 或 qt-mcp 直接控制 Qt 应用窗口/控件，配合 Claude Code 全自动调试

## 六、相关参考
- ewis/matlab-mcp-server：创建/修改变量、执行代码、调用 Simulink 仿真、画图分析
- 开发工具链：keil + vscode + git + claude + codex + claude design
