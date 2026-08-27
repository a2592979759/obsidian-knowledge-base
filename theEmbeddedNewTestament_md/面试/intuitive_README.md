---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Company/intuitive/README.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的嵌入式题库、带 AI 反馈的编码练习以及系统设计指南进行准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)**

---

# 风扇控制应用演示（Fan Control App demo）

**目录（Table of Contents）：**
- 项目目标（Project Objective）
- 应用演示设计分析（App Demo Design Analysis）
  - 架构（Architecture）
  - 特性（Feature）
  - 设计细节（Design Details）
  - 局限性（Limitation）
  - 改进（Improvments）
- 使用（Usage）
  - 编译与清理（Compilation and cleanup）
  - 运行演示程序（Run the demo program）
  - 停止演示程序（Stop the demo program）
- 演示结果展示（Demo Result Exhibitaion）

## 项目目标（Project Objective）

任务是开发一个控制风扇转速的应用。该应用应满足以下要求：

    • 每个子系统的温度以 32 位浮点数（°C）形式通过 IPC 提供。

    • 子系统数量与风扇数量都应在启动时可配置，但你可以假定每个数量都有上界。你可以假定每个数量在启动后保持不变。

    • 每个风扇的速度通过向一个对每个风扇都不同的特定硬件寄存器写入 32 位无符号整数来设定。该整数以 PWM 计数（PWM counts）表示，与风扇占空比（duty cycle）成正比。

    • 对应 100% 占空比的 PWM 计数可能因风扇不同而不同。你可以假定 0 PWM 计数始终代表 0% 占空比。

风扇控制算法应按如下方式工作：

    1. 应采集来自各子系统的最新的温度测量值，并根据所有子系统中最新温度的最大值来计算风扇占空比。

    2. 所有风扇应被设置为相同的占空比。

    3. 如果温度低于或等于 25°C，风扇应以 20% 的占空比运行。

    4. 如果温度高于或等于 75°C，风扇应以 100% 的占空比运行。

    5. 如果测得的最大子系统温度介于 25°C 与 75°C 之间，风扇应以在线性插值于 20% 与 100% 占空比之间的占空比运行。

提交内容应包括：

    1. 一个小型演示程序，用于传达子系统温度并以 PWM 计数写入风扇占空比。

    2. 用于通过 IPC 读取温度测量值、配置应用以及写入硬件寄存器的最小接口（Minimalist interfaces），应按你认为合适的方式模拟（mock）出来。

对于你的测试程序，你可以随意设定风扇数量、子系统数量以及每个风扇的最大 PWM 计数。

## 应用演示设计分析（App Demo Design Analysis）

### 架构（Architecture）

![[_assets/App_demo_architecture.png]]

***风扇控制服务器（Fan Control Server）：***
- 一个独立程序，负责处理来自风扇控制客户端（fan control clients）的温度读取消息。

***风扇控制服务器 - 接收器（Fan Control Server - Receiver）：***
- 一个专门的功能（主循环 main loop），用于从服务器消息队列（server message queue）读取数据。如果收到来自客户端的消息，则操作风扇控制组（fan control group）。

***风扇控制服务器 - 定时器（Fan Control Server - Timer）：***
- 一个预设的定时器，重复唤醒并执行动作，例如向所有活跃模块客户端发送温度读取查询，并设置风扇控制组的转速。

***风扇控制服务器 - 风扇控制组（Fan Control Server - Fan Control Group）：***
- 一种抽象数据结构（abstration data structure），将所有模块及其风扇聚合在一起，同时处理与风扇硬件相关的操作，例如模块温度值更新、风扇转速占空比设置与计算。

***风扇控制客户端（Fan Control Client）：***
- 一个执行温度读取并发出温度的程序，既可以作为对服务器请求的回复，也可以在它认为合适的任何时候发出（例如因过热而急需提高风扇转速）。
- 在真实场景中，客户端模块进程通常还有其他任务。但在我们的应用演示中，假定客户端只有一个任务——读取温度并在需要时发送给服务器。

***服务器消息队列（Server Msg Queue）***
- 一种 FIFO 单向 IPC 机制，供客户端将温度及其他消息传给服务器。客户端只能向队列写入，而服务器只能读取。

***客户端消息队列（Client Msg Queue）***
- 一种 FIFO 单向 IPC 机制，供服务器将查询消息传给客户端。服务器只能向队列写入，而客户端只能读取。

***Fan_hw***
- 风扇硬件实例数据结构，其中包含与风扇控制逻辑相关的硬件层厂商特定信息，例如寄存器偏移量（register offsets）以及将占空比转换为 PWM 计数的特定函数。

### 特性（Feature）

- ***模块热插拔（Module Hotplug）：*** 支持客户端模块的动态添加/删除。风扇服务器只会为这些活跃客户端开启风扇，以避免为未使用的模块保持风扇运转而产生过多噪音。
- ***优先级消息投递（Priority Message Delivery）：*** 温度读取消息根据数值被划分为 4 个不同的优先级。读数越高，消息在队列中的优先级越高。高优先级消息将被放置在队列前部，以确保更快地投递。
- ***单个模块风扇转速调整（Individual Module Fan Speed Adjustment）：*** 任何特定模块的风扇转速都可以根据其最新的温度读数单独调整，而不影响其他模块。
- ***动态模块风扇数量配置（Dynamic Module Fan Number Configuration）：*** 尽管在演示程序中假定每个模块只有 1 个风扇，但为任何模块分配任意数量的风扇都非常容易。
- ***心跳温度读取查询（Heartbeat Temperature Reading Query）：*** 服务器会每隔一段时间唤醒，并通过向每个活跃客户端的消息队列发送查询消息来请求温度读数。
- ***紧急请求处理（Urgent Request Processing）：*** 除了例行地设置通用风扇转速外，它还能够在收到紧急客户端请求时立即响应。一旦收到因过热事件导致的消息，服务器会直接将对应过热模块的风扇转速设置到最大。
- ***硬件抽象层（Hardware Abstaction Layer）：*** 通用的硬件接口与硬件抽象数据结构确保可以轻松支持不同厂商的风扇。硬件支持就像在数据结构中替换几个寄存器值和函数指针一样简单。

### 设计细节（Design Details）

将在小组讨论（panel discussion）中详述。

### 局限性（Limitation）

- ***缺少时间感知（No Sense of Time）：*** 控制服务器不知道读数发生的时刻，因此风扇转速调整可能基于某个模块的过期（out-of-date）数据。

### 改进（Improvments）

- ***添加模块读数时间戳与强制踢出（Add Module Reading Timestamp and Force Kick）：*** 为了解决时间感知的局限性，客户端也可记录读数的时间戳值并包含在要投递的消息中。知道这一点后，如果某个模块没有回复服务器的查询消息且其温度读数长时间未更新，服务器可以强制踢出（force kick）该不响应的模块。这些模块很可能已经意外离线，因此不应再把它们的旧读数纳入考虑。
- ***采用类 PID 算法以提升冷却性能（PID-like algorithm to enhance cooling performance）：*** 用 PID 算法替代当前简单的线性算法，肯定有助于提升整体系统的冷却性能。

## 使用（Usage）

### 编译与清理（Compilation and cleanup）

***构建（Build）：***
```
    make all
```

***清理（Clean up）：***
```
    make clean
```

### 运行演示程序（Run the demo program）

***运行风扇控制服务器（Run fan_control server）：***
```
    sudo ./fan_control_server <number-of-module>|<Enter>
```
它接受第一个参数作为允许连接的模块数量。如果未指定，将使用 20 作为默认值。

***运行风扇控制客户端（Run fan_control client）：***
```
    sudo ./fan_control_client <module-id>
```
它接受第一个参数作为模块 ID。服务器将使用该值来标识模块。请注意，如果已存在相同 ID 的模块，客户端将不会运行。

```注意：请使用 'sudo' 来运行客户端/服务器演示程序。否则消息队列将无法创建！```

### 停止演示程序（Stop the demo program）

使用 Ctrl-C 为每个程序产生终止信号。

## 演示结果（Demo Result）

```
sudo ./fan_control_server 10

sudo ./fan_control_client 0
sudo ./fan_control_client 1
sudo ./fan_control_client 2
sudo ./fan_control_client 3
...
```

***风扇控制服务器输出（Fan_control server output）：***
```
Max module number to be conncted: 10
Start receiving from message queue!
timer_handler triggered.

No active module. Continue.

timer_handler triggered.

Server: temp val 0.000000 from module 0.
Server: client 0 queue name /fan-control-client-0
Start receiving from message queue!
Server: temp val 93.000000 from module 0.
Server: Received urgent msg from module 0. Adjust speed now!
Start receiving from message queue!
timer_handler triggered.

=======================
Active module number: 1
Max temperature: 93
Temperature by Module: 93
[93 0 0 0 0 0 0 0 0 0 ]
Fan speed duty cycle: 100
=======================

Server: Query all active module for temperature.
Server: temp val 29.000000 from module 0.
Start receiving from message queue!
Server: temp val 0.000000 from module 1.
Server: client 1 queue name /fan-control-client-1
Start receiving from message queue!
Server: temp val 93.000000 from module 1.
Server: Received urgent msg from module 1. Adjust speed now!
Start receiving from message queue!
timer_handler triggered.

=======================
Active module number: 2
Max temperature: 93
Temperature by Module: 93
[29 93 0 0 0 0 0 0 0 0 ]
Fan speed duty cycle: 100
=======================

```

***风扇控制客户端 0 输出（Fan_control client 0 output）：***
```
Module ID: 0
Client 0: Send module ATTACH message.
Client 0 Temp_val: 83
Client 0 Temp_val: 86
Client 0 Temp_val: 77
Client 0 Temp_val: 15
Client 0 Temp_val: 93
Client 0: Module Temperature above threshold! Send urgent message!.
Client 0 Temp_val: 35
Client 0: Receive msg from server. Reply..
Client 0 Temp_val: 86
Client 0 Temp_val: 92
Client 0: Module Temperature above threshold! Send urgent message!.
Client 0 Temp_val: 49
Client 0 Temp_val: 21
Client 0 Temp_val: 62
Client 0: Receive msg from server. Reply..
Client 0 Temp_val: 27
Client 0 Temp_val: 90
Client 0 Temp_val: 59
Client 0 Temp_val: 63
Client 0 Temp_val: 26
Client 0: Receive msg from server. Reply..
```

***风扇控制客户端 1 输出（Fan_control client 1 output）：***
```
Module ID: 1
Client 1: Send module ATTACH message.
Client 1 Temp_val: 83
Client 1 Temp_val: 86
Client 1 Temp_val: 77
Client 1 Temp_val: 15
Client 1 Temp_val: 93
Client 1: Module Temperature above threshold! Send urgent message!.
Client 1 Temp_val: 35
Client 1: Receive msg from server. Reply..
Client 1 Temp_val: 86
Client 1 Temp_val: 92
Client 1: Module Temperature above threshold! Send urgent message!.
Client 1 Temp_val: 49
Client 1 Temp_val: 21
Client 1 Temp_val: 62
Client 1: Receive msg from server. Reply..

```

## 相关页面
- [[intuitive_README]] —— Intuitive 风扇控制项目
- [[amazon]] —— Amazon
- [[lyft]] —— Lyft
- [[tesla]] —— Tesla
- [[prepare]] —— 通用嵌入式面试准备清单

返回索引 [[00-索引]]
