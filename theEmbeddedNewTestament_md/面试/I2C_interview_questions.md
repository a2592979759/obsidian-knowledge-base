---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Concept/I2C_interview_questions.md"
created: 2026-08-27
---

> ## 🚀 交互式练习 I2C 面试题
>
> 这些笔记为 **[EmbeddedInterviewLab](https://embeddedinterviewlab.com/questions/topic/i2c?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=i2c)** 上的**由社区评分的、可搜索题库**提供支撑——包含模型答案、点赞，以及真实的"我被问到过"信号。
>
> 👉 **[浏览 I2C 题库 →](https://embeddedinterviewlab.com/questions/topic/i2c?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=i2c)** &nbsp;·&nbsp; 📚 **[阅读 I2C 深度专题 →](https://embeddedinterviewlab.com/topics/i2c?utm_source=github&utm_medium=referral&utm_campaign=topic&utm_content=i2c)**

---

### URL 列表

https://aticleworld.com/i2c-interview-questions/

[用简单的话说什么是 I2C？我们在哪里使用这个协议？为什么大多数 SOC 都会有 I2C 协议？](https://www.quora.com/What-is-I2C-in-simple-terms-Where-do-we-use-this-protocol-Why-would-most-SOCs-have-I2C-protocol)


### 深入 I2C 问题

***问：解释 I2C 协议的物理层（physical layer）***

***答：***

I2C 是纯粹的（pure）主从（master-slave）通信协议，它可以是多主（multi-master）或多从（multi-slave）的，但我们通常在 I2C 通信中看到的是单一主机（single master）。在 I2C 中只使用两根线进行通信，一个是数据总线（SDA），另一个是时钟总线（CLK）。

所有从机和主机都连接在同一条数据和时钟总线上，这里要记住的重要事情是：这些总线之间通过"线与"（WIRE-AND）配置连接，这是通过将两个引脚都设为开漏（open drain）来实现的。线与配置允许 I2C 连接多个节点到总线上，而不会因为信号争用（signal contention）造成短路。

开漏（open-drain）允许主机和从机将线路拉低并释放为高阻态（high impedance state）。因此，在这种主机和从机都释放总线的情况下，需要一个上拉电阻（pull-up resistor）将线路拉高。上拉电阻的取值非常重要，关系到 I2C 系统的设计，因为错误的上拉电阻取值可能导致信号丢失。

注意：我们知道 I2C 通信协议支持多个主机和多个从机，但大多数系统设计只包含一个主机。

***问：解释 I2C 协议的操作和帧（frame）***

***答：***

I2C 是一种芯片到芯片（chip to chip）的通信协议。在 I2C 中，通信总是由主机发起。当主机想与从机通信时，它先发出一个起始位（start bit），随后是从机地址（slave address）加读/写位（read/write bit）。

发出起始位后，所有从机都进入监听（attentive）模式。如果发送的地址与总线上某个从机的地址匹配，那么该从机会向主机发送一个确认位（ACKNOWLEDGEMENT，ACK）。

在收到 ACK 位后，主机开始通信。如果没有从机的地址与发送的地址匹配，那么主机会收到一个非确认位（NOT-ACKNOWLEDGEMENT，NACK）。在这种情况下，主机要么发出停止位（stop bit）来停止通信，要么在线上发出一个重复起始位（repeated start bit）开始新的通信。

当我们在 I2C 中发送或接收字节时，总是会在每个数据字节传输完成后收到一个 NACK 位或 ACK 位。

在 I2C 中，每个时钟周期始终传输一位。在 I2C 中传输的一个字节可以是设备地址、寄存器地址，或者是写入从机或从从机读取的数据。

在 I2C 中，除了起始条件、停止条件和重复起始条件外，SDA 线在时钟高电平阶段始终是稳定的。SDA 线只在时钟低电平阶段改变状态。

见下图，
![[_assets/i2c-frame.jpg]]

***问：什么是起始位（START bit）和停止位（STOP bit）？***

***答：***

**起始条件（Start Condition）**：
SDA 和 SCL 线的默认状态是高电平。主机在线上发出起始条件以开始通信。在 SCL 线为高电平时，SDA 线从高到低的跳变称为起始条件（START condition）。起始条件总是由主机发出。在发出起始位之后，I2C 总线被视为忙（busy）。

![[_assets/i2start.jpg]]

**停止条件（Stop Condition）**：
停止条件（STOP condition）由主机发出以停止通信。在 SCL 线为高电平时，SDA 线从低到高的跳变称为停止条件。停止条件总是由主机发出。在发出停止位之后，I2C 总线被视为空闲（free）。

![[_assets/stop-min.jpg]]

*注意：起始条件和停止条件总是由主机发出。*

***问：什么是重复起始条件（repeated start condition）？***

***答：***

重复起始条件与起始条件类似，但两者彼此不同。重复起始是由主机在停止条件之前发出的（即总线不处于空闲状态时）。

当主机不想失去对总线的控制时，会发出重复起始条件。当主机想开始新的通信而不发出停止条件时，重复起始对主机是有利的。

*注意：当 I2C 总线上连接了多个主机时，重复起始是有利的。*

***问：I2C 的标准总线速度是多少？***
I2C 有以下速度模式：

模式 | 速度
-----|---------
标准模式（Standard-mode）	                 | 100 kbit/s
快速模式（Fast-mode）	                  | 400 kbit/s
快速模式增强（Fast-mode Plus）	               | 1 Mbit/s
高速模式（High-speed mode）	            | 3.4 Mbit/s

***问：限制 I²C 总线上可挂载设备数量的因素是什么？***

***答：***

这取决于总电容（total capacitance）。


***问：谁发送起始位？***

***答：***

在 I2C 中由主机发送起始位。

***问：I2C 总线的最大总线长度是多少？***

***答：***

这取决于总线负载（电容）和速度。基本上 I2C 不是为长距离通信设计的。它被限制在几米内。对于快速模式和使用电阻上拉的场景，据 NXP 文档 "UM10204.pdf" 所述，电容应小于 200pF。所以如果你的导线是 20pF/25cm，并且还有 80pF 的杂散和输入电容，那么你的电缆长度被限制在 1.5m。但这只是一个粗略的假设。在实际场景中可能会有所不同。

***问：什么是总线仲裁（bus arbitration）？***

***答：***

仲裁（arbitration）在多主机（multi-master）情况下是必需的，即多个主机试图同时与一个从机通信时。在 I2C 中，仲裁是通过 SDA 线实现的。

例如，
假设 I2C 总线上有两个主机试图同时与一个从机通信，那么它们都会在总线上发出起始条件。I2C 总线的 SCL 时钟会通过线与逻辑（wired and logic）同步。

![[_assets/arbitration-min.jpg]]

在上述情况下，只要 SDA 线的状态与各个主机在总线上驱动的状态一致，一切都会正常。如果任何主机发现 SDA 线的状态与它驱动的不同，那么它们就会退出通信并失去仲裁。

*注意：失去仲裁的主机将等待直到总线空闲。*

***问：什么是 I2C 时钟拉伸（clock stretching）？***

***答：***

在 I2C 中，通信可以通过时钟拉伸（clock stretching）来暂停，即持续将 SCL 线拉低，直到 SCL 线再次释放为高电平才能继续。

![[_assets/i2c-clock-stretch.jpg]]

在 I2C 中，从机能够以较快的速度接收一个字节的数据，但有时从机需要更多时间来接收到的字节，在这种情况下，从机会把 SCL 线拉低以暂停传输，而在处理完接收到的字节后，它再次释放 SCL 线为高电平，从而恢复通信。

时钟拉伸是从机驱动 SCL 线的一种方式，但事实上，大多数从机并不会驱动 SCL 线。

注意：在 I2C 通信协议中，大多数 I2C 从机设备不使用时钟拉伸功能，但每个主机都应该支持时钟拉伸。

***问：什么是 I2C 时钟同步（clock synchronization）？***

***答：***

与 Rs232 不同，I2C 是同步通信（synchronous communication），在这种通信中，时钟总是由主机产生，并且这个时钟由主机和从机共享。在多主机的情况下，所有主机都产生各自的 SCL 时钟，因此所有主机的时钟必须同步。在 I2C 中，这种时钟同步是通过线与逻辑实现的。

为了更好地理解，我举个例子，两个主机试图与一个从机通信。在这种情况下，两个主机都产生各自的时钟，主机 M1 产生 clk1，主机 M2 产生 clk2，而在总线上观察到的时钟是 SCL。

![[_assets/Clock-sync.jpg]]

SCL 时钟将是 clk1 和 clk2 的与（Anding，clk1 & clk2），最有趣的是 SCL 线的最高逻辑 1 由逻辑 1 最低的那个 CLK 决定。

## 更多 I2C 问题

1. 在 I2C 中，系统运行时可以添加和移除设备吗（热插拔, Hot swapping）？
2. I2C 的标准总线速度是多少？
3. 在标准 I2C 通信中可以连接多少个设备？
4. I2C 通信中节点的两个角色是什么？
5. I2C 通信中的操作模式有哪些？
6. 什么是总线仲裁？
7. I2C 通信的优点和局限性是什么？
8. I2C 通信需要多少根线？I2C 中涉及的信号有哪些？
9. 什么是起始位和停止位？
10. 主机如何表示它是地址还是数据？它如何告知从机它将要读还是写？
11. 在 I2C 中可以有多个主机吗？
12. 写事务中，主机监视最后一个 ACK 并发出停止条件——对/错？
13. 读事务中，主机不确认它接收到的最后一个字节并发出停止条件——对/错？
14. 什么是 SPI 通信？
15. SPI 通信需要多少根线？
16. SPI 总线规定的 4 个逻辑信号是什么？
17. SPI 从机是否确认数据的接收？
18. SPI 的吞吐量比 I2C 高——对/错？
19. 在微处理器和 DSP 之间进行数据通信，使用 I2C 还是 SPI 更好？
20. 从 ADC 进行数据通信，使用 I2C 还是 SPI 更好？
21. 在每个 SPI 时钟周期同时使用 MOSI 和 MISO 可以实现全双工（duplex）通信——对/错？
22. 可以将 SPI 从机连接成菊花链（daisy chain）吗？
23. 移位寄存器（shift register）在 SPI 主设备和从设备中的作用是什么？
24. 主机如何传达它即将停止数据的传输？
25. 什么是位敲打（bit banging）？

## 更多链接

https://aticleworld.com/i2c-interview-questions/

---

## 相关页面

- [[SPI_interview_questions]]
- [[UART_interview_questions]]
- [[CAN_interview_questions]]
- [[Concept_questions]]
- [[embedded_interview_questions]]

返回索引 [[00-索引]]
