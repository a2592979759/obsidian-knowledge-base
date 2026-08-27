---
tags:
  - 嵌入式
  - 总线协议
  - UART
source: "Bus_Protocol/Uart.md"
created: 2026-08-27
---

# UART (Universal Asynchronous Receiver-Transmitter)

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入钻研
>
> 将这些总线协议概念作为带参考答案的排序面试题来学习，并配有交互式深入钻研指南。
>
> 👉 **[浏览外设与协议问题 →](https://embeddedinterviewlab.com/questions/domain/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=bus_protocol)** &nbsp;·&nbsp; **[阅读协议主题指南 →](https://embeddedinterviewlab.com/topics?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=bus_protocol)**

---

```“UART” 代表通用异步收发器 (Universal Asynchronous receiver-transmitter)。它是微控制器内部存在的一个硬件外设。```

## 什么是 UART？

“UART” 代表通用异步收发器 (Universal Asynchronous receiver-transmitter)。它是微控制器内部存在的一个硬件外设。UART 的功能是把**传入和传出的数据转换为串行比特流 (serial binary stream)**。从外设接收到的 8 位串行数据通过串行到并行转换被转换成并行形式，而来自 CPU 的并行数据则被转换成串行形式。这些数据以调制形式存在，并按约定的波特率 (baud rate) 传输。

串行通信的最后一块拼图，是找到既能生成串行数据包、又能控制那些物理硬件线路的东西。这就是 UART。

通用异步收发器 (UART) 是负责实现串行通信的一块电路。本质上，UART 充当并行与串行接口之间的中介。UART 的一端是一条约由八根数据线（外加一些控制引脚）组成的总线，另一端是两根串行线——RX 和 TX。

![Uart connection](https://img-blog.csdnimg.cn/20181225114440688.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N0ZXJubHljb3Jl,size_16,color_FFFFFF,t_70)

![Parallel Serial Conversion](https://cdn.sparkfun.com/assets/d/1/f/5/b/50e1cf30ce395fb227000000.png)

### *框图 (Block Diagram)*

UART 由以下核心组件组成，即发送器 (transmitter) 和接收器 (receiver)。发送器由发送保持寄存器 (Transmit hold register)、发送移位寄存器 (Transmit shift register) 和控制逻辑 (control logic) 组成。类似地，接收器由接收保持寄存器 (Receive hold register)、接收移位寄存器 (Receiver shift register) 和控制逻辑组成。通常，发送器和接收器都配有一个波特率发生器 (baud rate generator)。

![Block diagram](https://www.codrey.com/wp-content/uploads/2017/10/UART-Block-Diagram.png)

波特率发生器 (baud rate generator) 生成发送器和接收器发送/接收数据的速度。发送保持寄存器包含待发送的数据字节。发送移位寄存器和接收移位寄存器将比特向左或向右移位，直到一个字节的数据发送/接收完毕。

除此之外，还提供一个读或写控制逻辑，用来指示何时读/写。波特率发生器生成从 110 bps（每秒比特数）到 230400 的速度。大多数微控制器支持更高的波特率，如 115200 和 57600，以实现更快的数据传输。像 GPS 和 GSM 这样的设备使用 4800 和 9600 这样较慢的波特率。

## 为什么用 UART？

像 SPI（串行外设接口，serial peripheral interface）和 USB（通用串行总线，Universal Serial Bus）这样的协议用于快速通信。当不需要高速数据传输时，就使用 UART。它是一种廉价的通信设备，带有一个发送器/接收器。它发送数据需要一根线，接收数据需要另一根线。

![Uart usage](https://www.codrey.com/wp-content/uploads/2017/10/UART-Interface.png)

可以使用 RS232-TTL 转换器或 USB-TTL 转换器与 PC（个人计算机）连接。RS232 和 UART 之间的共同点是它们都不需要时钟来发送和接收数据。UART 帧由 1 个起始位、1 或 2 个停止位以及一个用于串行数据传输的校验位组成。

## 协议格式 (Protocol Format)

UART 以一个起始位 '0' 开始通信。起始位启动串行数据的传输，停止位结束数据传输事务。

![Uart packet](https://www.codrey.com/wp-content/uploads/2017/10/UART-Protocol-format.png)

![Protocol](https://img-blog.csdnimg.cn/20181226094646598.png)

![In transmission](https://img-blog.csdnimg.cn/20181226100234335.png)

它还配有一个校验位 (parity bit)（偶校验或奇校验）。偶校验位用 '0' 表示（1 的个数为偶数），奇校验位用 '1' 表示（1 的个数为奇数）。

### *起始位 (Start Bit)*
UART 数据发送线在不发送数据时通常保持在高电平。为了开始数据传输，发送方 UART 将发送线拉低一个时钟周期。当接收方 UART 检测到高到低的电压跳变时，它就开始按波特率的频率读取数据帧中的比特。

起始位是逻辑低电平。起始位向接收方发出信号：一个新字符即将到来。

### *数据帧 (Data Frame)*
数据帧包含实际传输的数据。如果使用校验位，它可以有 5 到 8 位长。如果不使用校验位，数据帧最长达 9 位。在大多数情况下，数据先发送最低有效位。

### *校验位 (Parity Bit)*
奇偶校验 (parity) 描述一个数的奇偶性。校验位是接收方 UART 判断传输过程中数据是否发生改变的一种方式。比特可能因电磁辐射、波特率不匹配或长距离数据传输而改变。接收方 UART 读取数据帧后，统计值为 1 的比特数量，并检查总数是偶数还是奇数。如果校验位是 0（偶校验），数据帧中 1 的比特总数应为偶数；如果校验位是 1（奇校验），数据帧中 1 的比特总数应为奇数。当校验位与数据匹配时，UART 就知道传输没有错误。但如果校验位是 0 而总数是奇数，或者校验位是 1 而总数是偶数，UART 就知道数据帧中的比特发生了改变。

### *停止位 (Stop Bit)*
为了表示数据包结束，发送方 UART 至少用两个比特时长将数据发送线从低电平驱动到高电平。

### *UART 错误 (UART Error)*
* 帧错误 (framing error) 发生在找不到规定的起始位和停止位时。如果在预期停止位时数据线不在期望状态，就会发生帧错误。
* 欠载错误 (Under run error) 发生在 UART 发送器已发送完一个字符而发送缓冲区为空时。在异步模式下，这被视为没有数据需要发送的指示。
* 校验错误 (parity error) 发生在 1 比特数量的奇偶性与校验位规定的不一致时。使用校验位是可选的，所以只有在校验功能被启用时才会出现此错误。
* 断点条件 (break condition) 发生在接收端输入处于空号电平 (space level) 超过一段时间时，通常是超过一个字符时间。

## 哪些设备使用 UART 协议？

* 像 SIM900/800 这样的 GPS 模块使用 UART 协议
* 指纹传感器 (finger print sensor) 使用 UART 协议
* RFID 模块使用 UART 协议
* 串行调试端口使用 UART 驱动打印来自外部世界的数据
* 我们可以用它向嵌入式设备发送命令、从嵌入式设备接收命令
* GPS、GSM/GPRS 调制解调器、Wi-Fi 芯片等中的通信都使用 UART
* 用于主机访问 (mainframe access) 以连接不同的计算机

## UART 速度 (波特率 - 比特/秒)
UART 支持多种波特率：

    300, 600, 1200, 2400, 4800, 9600, 19200, 38400, 57600, 74880, 115200, 230400, 256000, 460800, 921600, 1843200, 3686400。

## UART 的优点与缺点
### 优点

* 只需两根线接口
* 不需要时钟信号
* 具有错误检测功能：带有一个校验位检查
* 只要两端都做好设置，数据包的结构可以更改
* 广泛使用的方法，资料齐全

### 缺点

* 数据帧最长仅限 9 位
* 不支持多从机或多主机系统
* 每个 UART 的波特率必须彼此在 10% 以内

## UART 与 USART
USART 是 UART 的基本形式。技术上它们并不完全相同。但两者的定义是一样的。它们都是把并行数据转换为串行比特、以及反过来的微控制器外设。

UART 和 USART 的主要区别是：UART 只支持异步通信，而 USART 既支持同步也支持异步通信。为了便于理解，这里是 USART 和 UART 之间的对比。

![[_assets/uart_vs_usart.png]]

## UART 是协议还是硬件？

更像是硬件而不是协议：

[UART 硬件还是协议讨论](https://electronics.stackexchange.com/questions/399550/uart-a-protocol-or-hardware)

## 关键要点
* 串行通信 (Serial Communication)
* 通信类型：异步 (Asynchronous) - 无时钟。
* UART 是全双工通信，但它也可以工作在半双工和单工方式。
* 发送器和接收器应使用相同波特率
* 数据格式和传输速度可配置。
* UART 包含一个移位寄存器，这是在串行与并行形式之间转换的基本方法。

## 参考链接

https://aruneworld.com/embedded/embedded-protocol/uart/

https://www.freebsd.org/doc/en_US.ISO8859-1/articles/serial-uart/index.html#uart

[Linux 8250 uart 驱动](https://elixir.bootlin.com/linux/v5.6.19/source/drivers/tty/serial/8250)

https://www.codrey.com/embedded-systems/uart-serial-communication-rs232/

https://blog.csdn.net/zjy900507/article/details/79789671?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-3&spm=1001.2101.3001.4242

https://blog.csdn.net/ayang1986/article/details/106729103

https://www.analog.com/en/analog-dialogue/articles/uart-a-hardware-communication-protocol.html#
