---
tags:
  - 嵌入式
  - 总线协议
  - I2C
source: "Bus_Protocol/i2c.md"
created: 2026-08-27
---

# I2C (Inter-Integrated Circuit)

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入钻研
>
> 将这些总线协议概念作为带参考答案的排序面试题来学习，并配有交互式深入钻研指南。
>
> 👉 **[浏览外设与协议问题 →](https://embeddedinterviewlab.com/questions/domain/peripherals?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=bus_protocol)** &nbsp;·&nbsp; **[阅读协议主题指南 →](https://embeddedinterviewlab.com/topics?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=bus_protocol)**

---

```I2C (Inter-Integrated Circuit) 协议是一种旨在允许多个“外围”数字集成电路（“芯片”）与一个或多个“控制器”芯片通信的协议。```

## 简介

I2C (Inter-Integrated Circuit) 协议是一种旨在允许多个“外围”数字集成电路（“芯片”）与一个或多个“控制器”芯片通信的协议。与串行外设接口 (Serial Peripheral Interface, SPI) 一样，它只用于单个设备内部的短距离通信。与异步串行接口（如 RS-232 或 UART）一样，它只需要两根信号线来交换信息。

![I2C](https://cdn.sparkfun.com/assets/learn_tutorials/8/2/I2C-Block-Diagram.jpg)

## 为什么要用 I2C？
### 串行 UART 端口有什么问题？
因为串行端口是异步的（没有传输时钟数据），使用它们的设备必须事先约定一个数据速率。两个设备还必须具有接近相同的时钟速率，并保持该速率——两端时钟速率差异过大就会导致数据混乱。

异步串行端口需要硬件开销——两端的 UART 相对复杂，必要时难以用软件精确实现。每个数据帧至少包含一个起始位和一个停止位，这意味着每发送 8 位数据就要占用 10 位的传输时间，这会蚕食数据速率。

异步串行端口的另一个核心缺陷是：它们天生只适合两个、且仅两个设备之间的通信。虽然可以把多个设备接到同一个串行端口，但总线竞争 (bus contention)（两个设备同时试图驱动同一根线）始终是个问题，必须小心处理以防损坏相关设备，通常需要借助外部硬件。

最后，数据速率也是个问题。虽然异步串行通信没有理论限制，但大多数 UART 设备只支持一组固定的波特率，其中最高通常约 230400 比特/秒。

总结：

- 异步串行通信要求两端有相同的时钟速率，且时钟速率差异不能过大，否则会导致数据混乱。
- 仅支持点对点通信，不支持多负载。
- 硬件复杂，因此更昂贵。
- UART 也消耗传输的比特：发送的 10 位中，至少有 2 位不是数据。
- 数据速率慢。

### SPI 有什么问题？
SPI 最明显的缺点是所需的引脚数量。用 SPI 总线把一个控制器连接到单个外设需要四根线；每增加一个外设设备，就需要在控制器上增加一个片选 (chip select) I/O 引脚。连接的快速激增使其在需要把一个控制器连接到大量设备的场景中不受欢迎。另外，每个设备的大量连接可能使紧凑 PCB 布局中的信号布线更加困难。

SPI 只允许总线上有一个控制器，但它支持任意数量的外设（只受连接到总线的设备驱动能力和可用片选引脚数量的限制）。

SPI 适用于高数据速率的全双工（同时发送和接收数据）连接，对某些设备支持超过 10MHz 的时钟速率（因此每秒 1 千万比特），并且速度扩展性很好。两端的硬件通常是一个非常简单的中存器，便于用软件实现。

总结：

- 使用太多引脚。每增加一个从设备就需要额外的 SS 线。
- 只允许一个主/控制器。
- 不支持热插拔 (hot swap)。

### I2C：两者之最优
I2C 像异步串行一样只需要两根线，但这两根线最多可支持 1008 个外设设备。此外，与 SPI 不同，I2C 支持多控制器系统，允许多个控制器与总线上所有外设设备通信（尽管控制器设备之间不能通过总线互相通信，必须轮流使用总线线路）。

数据速率介于异步串行和 SPI 之间；大多数 I2C 设备能以 100kHz 或 400kHz 通信。I2C 有一些开销；每发送 8 位数据，必须额外传输一位元数据（“ACK/NACK”位，我们稍后会讨论）。

实现 I2C 所需的硬件比 SPI 复杂，但比异步串行简单。它几乎可以很轻松地用软件实现。

总结：

- 只需要两根线。
- 多控制器系统，允许多个主机和多个从机。如果地址是 7 位，则允许 128 个从设备。
- 不同模式下速度可以很快，只浪费 1 位（ACK/NACK）。
- 硬件复杂度高于 SPI 但低于 UART。

## I2C 硬件

```I2C 总线驱动器是“开漏”的 (open drain)，意味着它们可以把对应的信号线拉低，但不能将其驱动为高。```

每条 I2C 总线由两个信号组成：SCL 和 SDA。SCL 是时钟信号，SDA 是数据信号。时钟信号总是由当前总线控制器生成；有些外设设备可能偶尔强制时钟拉低，以延迟控制器发送更多数据（或是要求更多时间来准备数据，然后才让控制器将其时钟输出）。这被称为“时钟拉伸 (clock stretching)”，在协议页面有描述。

与 UART 或 SPI 连接不同，I2C 总线驱动器是 ***“开漏” (open drain)*** 的，意味着它们可以把对应的信号线拉低，但不能将其驱动为高。因此，不会出现一个设备试图把线拉高、另一个试图把它拉低的总线竞争情况，消除了损坏驱动器或系统过度耗散电力的可能。每根信号线上都有一个 ***上拉电阻 (pull-up resistor)***，在没有设备将其拉低时把信号恢复到高电平。

![I2C diagram](https://cdn.sparkfun.com/assets/learn_tutorials/8/2/I2C_Schematic.jpg)

电阻的选择随总线上的设备而异，但一个好的经验法则是从 4.7kΩ 电阻开始，必要时向下调整。I2C 是一个相当健壮的协议，可用于短距离布线（2-3m）。对于长距离或设备很多的系统，较小的电阻更好。

    上拉电阻选择的经验法则： 4.7kΩ

这里有个好链接介绍如何选择电容和上拉电阻值及其背后的原因：

[I2C 典型配置](https://www.i2c-bus.org/i2c-primer/typical-i2c-bus-setup/)

## 时钟、拉伸、仲裁
### 时钟生成 (Clock Generation)
SCL 时钟总是由 I2C 主机生成。规范要求时钟信号的低相位和高相位有最小周期。因此，实际时钟速率可能低于标称时钟速率，例如在高电容导致上升时间较长 (large rise times) 的 I2C 总线上。

### 时钟拉伸 (Clock Stretching)
I2C 设备可以通过拉伸 SCL 来减慢通信：在 SCL 低相位期间，总线上的任何 I2C 设备都可以额外将 SCL 拉低以阻止其再次上升，从而减慢 SCL 时钟速率或暂时停止 I2C 通信。这也被称为时钟同步 (clock synchronization)。

注意：I2C 规范没有为时钟拉伸规定任何超时条件，也就是说任何设备都可以*想拉多久就拉多久*地按住 SCL。

### 仲裁 (Arbitration)
多个 I2C 多主机可以连接到同一条 I2C 总线并并发运行。通过不断监视 SDA 和 SCL 的起始与停止条件，它们可以判断总线当前是空闲还是忙碌。如果总线忙，主机会推迟待处理的 I2C 传输，直到停止条件指示总线再次空闲。

然而，两个主机可能同时开始一次传输。在传输过程中，主机不断监视 SDA 和 SCL。如果其中一个检测到 SDA 在本应为高时却为低，它就假设另一个主机处于活动状态并立即停止传输。这个过程称为仲裁 (arbitration)。

## 协议
```两种帧：地址帧 (address frame) 和数据帧 (data frame)。```

消息被分成两种类型的帧：一个 ***地址帧***（控制器指示消息要发送给哪个外设），以及一个或多个 ***数据帧***（从控制器传给外设或反之的 8 位数据消息）。数据在 **SCL 变低之后** 放到 SDA 线上，**在 SCL 线变高之后被采样**。时钟边沿与数据读/写之间的时间由总线上的设备定义，会因芯片而异。

![protocol](https://cdn.sparkfun.com/assets/learn_tutorials/8/2/I2C_Basic_Address_and_Data_Frames.jpg)

数据是在 SCL 为高时被采样的，所以 SDA 线在 SCL 为高时必须保持稳定。SDA 被允许在 SCL 为低时改变：

![[_assets/I2C_sampling.png]]

### 起始与停止条件 (Start and Stop Condition)
- 所有事务都以一个 **START (S)** 条件开始，并以 **STOP (P)** 条件结束。
- SDA 线在 SCL 为 **高** 时发生 **高到低** 跳变定义了一个 START 条件。
- SDA 线在 SCL 为高时发生 **低到高** 跳变定义了一个 **STOP** 条件。

![[_assets/I2C_start_stop.png]]

START 和 STOP 条件总是由主机产生。总线在 START 条件之后被认为处于忙碌状态。STOP 条件之后经过一段时间，总线被认为再次空闲。

如果发出重复起始 (repeated START, Sr) 而不是 STOP 条件，总线保持忙碌。在这方面，START (S) 和重复起始 (Sr) 条件在功能上是相同的。

### 字节格式 (Byte format)
```放在 SDA 线上的每个字节必须是 8 位长。```

每次传输可传输的字节数不受限制。每个字节后面必须跟一个应答位 (Acknowledge bit)。数据先传最高有效位 (**MSB**)（见图 6）。如果从机在执行完其他功能（例如处理内部中断）之前无法接收或发送另一个完整字节，它可以保持时钟线 SCL 为低，迫使主机进入等待状态 ***(时钟拉伸 clock stretching)***。当从机准备好下一个字节并释放时钟线 SCL 时，数据传输继续。

![[_assets/I2C_byte_format.png]]

### 应答 (ACK) 与非应答 (NACK)
```应答发生在每个字节之后。```

应答位允许接收方通知发送方：该字节已被成功接收，可以发送下一个字节。主机生成所有时钟脉冲，包括应答的第九个时钟脉冲。

应答信号定义如下：**发送方在应答时钟脉冲期间释放 SDA 线，以便接收方可以把 SDA 线拉低，并在该时钟脉冲的高电平期间保持稳定为低**。还必须考虑建立时间 (set-up) 和保持时间 (hold times)。

```当 SDA 在第九个时钟脉冲期间保持为高时，这被定义为非应答 (Not Acknowledge) 信号。```

然后主机可以产生一个 STOP 条件来中止传输，或产生一个重复 START 条件来开始新的传输。有五种情况会导致产生 NACK：

1. 总线上没有与所发送地址对应的接收方，因此没有设备响应该应答。
2. 接收方因正在执行某些实时功能而无法接收或发送，且尚未准备好与主机开始通信。
3. 传输过程中，接收方收到它无法理解的数据或命令。
4. 传输过程中，接收方无法再接收更多数据字节。
5. 主机-接收方必须向从机-发送方发出传输结束的信号。

根据第 5 条，NACK 有时在传输末尾发送（从机不把 SDA 线拉低来发送 ACK），主机将发送停止条件来结束传输。

### 时钟同步 (Clock synchronization)

```一个同步的 SCL 时钟的 LOW 周期由 LOW 周期最长的主机决定，其 HIGH 周期由 HIGH 周期最短的主机决定。```

两个主机可以在空闲总线上同时开始发送，必须有一种方法来决定谁接管总线并完成传输。这通过时钟同步和仲裁完成。**在单主机系统中，不需要时钟同步和仲裁。**

时钟同步通过 I2C 接口到 SCL 线的线与 (wired-AND) 连接来完成。这意味着 SCL 线的一个高到低跳变会让相关主机开始计时它们的 LOW 周期，一旦某个主机时钟变低，它就把 SCL 线保持在该状态，直到时钟到达 HIGH 状态（见图 7）。然而，如果另一个时钟仍在其 LOW 周期内，该时钟的低到高跳变可能不会改变 SCL 线的状态。因此，SCL 线被 LOW 周期最长的主机保持为低。LOW 周期较短的主机在这段时间进入高电平等待状态。

![[_assets/I2C_clock_sync.png]]

当所有相关主机都计完它们的 LOW 周期后，时钟线被释放并变为高。此时主机的时钟与 SCL 线状态没有差别，所有主机开始计时它们的 HIGH 周期。第一个完成其 HIGH 周期的主机再次把 SCL 线拉低。

### 仲裁 (Arbitration)

```仲裁与同步一样，指的是仅在系统使用多于一个主机时才需要的部分协议。```

从机不参与仲裁过程。主机只有在总线空闲时才能开始传输。两个主机可能在 START 条件的最小保持时间 (tHD;STA) 内产生一个 START 条件，从而在总线上产生一个有效的 START 条件。此时需要仲裁来决定哪个主机将完成其传输。

![[_assets/I2C_arbitration.png]]

### 时钟拉伸 (Clock stretching)
```时钟拉伸通过把 SCL 线保持为低来暂停事务。事务必须等到该线再次释放为高才能继续。```

时钟拉伸是可选的，事实上大多数从机设备不包含 SCL 驱动器，因此无法拉伸时钟。

一个设备可能能够以较快的速率接收字节，但需要更多时间来存储接收到的字节或准备下一个要发送的字节。从机可以在接收并应答一个字节后保持 SCL 线为低，以此作为一种握手过程迫使主机进入等待状态，直到从机准备好下一个字节传输。

### 寻址 (Addressing)

```在 START 条件 (S) 之后，会发送一个从机地址。该地址是 7 位长，后面跟一个数据方向位 (R/W)——‘0’表示发送 (WRITE)，‘1’表示请求数据 (READ)（参考图 10）。```

数据传输总是由 **主机制造的 STOP 条件 (P)** 终止。然而，如果主机仍希望在总线上通信，它可以产生一个 **重复起始条件 (Sr)** 并在不首先产生 STOP 条件的情况下寻址另一个从机。在这种传输中，可以有各种读/写格式的组合。

![[_assets/I2C_data_transfer.png]]

![[_assets/I2C_1st_byte.png]]

可能的传输格式：
- ***主机-发送方 向 从机-接收方 发送。*** 传输方向不改变。从机接收方应答每个字节。

![[_assets/I2C_transfer_figure_1.png]]

- ***主机在第一个字节后立即读取从机。*** 在第一次应答的瞬间，主机-发送方变成主机-接收方，从机-接收方变成从机-发送方。这第一次应答仍然**由从机**产生。**主机**产生后续的应答。**STOP 条件由主机制造**，它**在 STOP 条件之前**发送一个**非应答 (A)**。

![[_assets/I2C_transfer_figure_2.png]]

- ***组合格式。*** 在一次传输中方向改变时，START 条件和从机地址都被重复，但 R/W 位反转。如果主机-接收方发送一个重复 START 条件，它会在重复 START 条件之前发送一个非应答 (A)。

![[_assets/I2C_transfer_figure_3.png]]

### 10 位寻址
```10 位从机地址由紧随 START 条件 (S) 或重复起始条件 (Sr) 之后的前两个字节构成。```

第一个字节的前 7 位是组合 1111 0XX，其中最后两位 (XX) 是 10 位地址的两个最高有效位 (MSB)；第一个字节的第 8 位是决定消息方向的 R/W 位。

![[_assets/I2C_10bit_address_transfer.png]]

## I2C 代码示例
发送 I2C 起始信号：
```c
void I2C_Start(void)
{
    I2C_SDA_High();     //SDA=1
    I2C_SCL_High();     //SCL=1
    I2C_Delay();
    I2C_SDA_Low();
    I2C_Delay();
    I2C_SCL_Low();
    I2C_Delay();
}
```
发送 I2C 停止信号：
```c
void I2C_Stop(void)
{
    I2C_SDA_Low();
    I2C_SCL_High();
    I2C_Delay();
    I2C_SDA_High();
    I2C_Delay();
}
```

发送一个字节
```c
u8 I2C_SendByte(uint8_t Byte)
{
    uint8_t i;
 
    /* 先发送 MSB */
    for(i = 0 ; i < 8 ; i++)
    {
        if(Byte & 0x80)
        {
            I2C_SDA_High();
        }
        else
        {
            I2C_SDA_Low();
        }
        I2C_Delay();
        I2C_SCL_High();
        I2C_Delay();
        I2C_SCL_Low();
        I2C_Delay();
 
        if(i == 7)
        {
            I2C_SDA_High();   /* 完成后释放 SDA */
        }
        Byte <<= 1;           /* 左移 */
 
        I2C_Delay();
    }
}　
```

读取一个字节：
```c
u8 I2C_ReadByte(void)
{
    uint8_t i;
    uint8_t value;
 
    /* 读取 MSB */
    value = 0;
    for(i = 0 ; i < 8 ; i++)
    {
        value <<= 1;
        I2C_SCL_High();
        I2C_Delay();
        if(I2C_SDA_READ())
        {
            value |= 0x1;
        }
        I2C_SCL_Low();
        I2C_Delay();
    }
 
    return value;
}
```

产生一个 ACK：
```c
void I2C_Ack(void)
{
    I2C_SDA_Low();
    I2C_Delay();
    I2C_SCL_High();
    I2C_Delay();
    I2C_SCL_Low();
    I2C_Delay();
    
    /* 释放 SDA 线 */
    I2C_SDA_High();
}
```

产生一个 NACK：
```c
void I2C_NoAck(void)
{
    I2C_SDA_High();
    I2C_Delay();
    I2C_SCL_High();
    I2C_Delay();
    I2C_SCL_Low();
    I2C_Delay();
}
```

读取 ACK：
```c
uint8_t I2C_WaitToAck(void)
{
    uint8_t redata;
 
    I2C_SDA_High();
    I2C_Delay();
    I2C_SCL_High();
    I2C_Delay();
    
    /* 如果 SDA 为低，则正在设置 ACK，返回 0 表示成功 */
    if(I2C_SDA_READ())
    {
        redata = 1;
    }
    else
    {
        redata = 0;
    }
    I2C_SCL_Low();
    I2C_Delay();
 
    return redata;
}
```

### 控制器发送示例

给定以下时序图，请使用之前列出的方法编写传输函数：

![timing diagram](https://i.imgur.com/IL0Ha8b.png)

图分解：

![diagram breakdown](https://i.imgur.com/VBHwFOr.png)

      1. 发送 START 信号。
      2. 发送 7 位地址 + 1 位 R/W。
      3. 外设发送 ACK。
      4. 发送寄存器地址，这是一个 8 位数据。
      5. 外设发送 ACK。
      6. 发送一个字节数据。
      7. 外设发送 ACK。
      8. 发送一个字节 CRC。
      9. 外设发送 ACK 或 NACK。
      10. 发送 STOP 信号。

代码：
```c
#define WRITE 0x0
#define READ 0x1

u8 I2C_WriteBytes(void)
{
    int ret;

    I2C_Start();                                //1
 
    I2C_SendByte((slave_Addr << 1) | WRITE);    //2
    ret = I2C_WaitToAck();                      //3
    if (ret) goto BAIL;
 
    I2C_SendByte(reg_Addr);                     //4
    ret = I2C_WaitToAck();                      //5
    if (ret) goto BAIL;
 
    I2C_SendByte(data);                         //6
    ret = I2C_WaitToAck();                      //7
    if (ret) goto BAIL;

    I2C_SendByte(crc);                          //8
    I2C_WaitToAck();                            //9

    /* 第 9 步无需检查 ACK，因为 ACK 和 NACK 都可以 */
    I2C_Stop();                     //10

BAIL:
    /* 失败处理流程 */
}
```

### 控制器读取示例

![controller read](https://i.imgur.com/HQ6HIUT.png)

时序分析：

![analysis](https://i.imgur.com/AjsozXQ.png)

     1. 发送 START 信号。
     2. 发送 7 位地址 + 1 位 R/W。
     3. 外设发送 ACK。
     4. 发送寄存器地址，这是一个 8 位数据。
     5. 外设发送 ACK。
     6. 控制器发送重复起始（本质上是 START）。
     7. 控制器发送 7 位地址 + 1 位 R/W。
     8. 外设发送 ACK。
     9. 外设发送一个字节（控制器读取）。
     10. 控制器发送 ACK。
     11. 外设发送一个字节 CRC（控制器读取）。
     12. 控制器发送 NACK。
     13. 控制器发送 STOP 信号。

代码：
```c
#define WRITE 0x0
#define READ 0x1

u8 I2C_ReadBytes(void)
{
    u8 data;
    u8 crc;
    int ret;
 
    I2C_Start();                                //1
 
    I2C_SendByte((slave_Addr << 1) | WRITE);    //2
    ret = I2C_WaitToAck();                      //3
    if (ret) goto BAIL;
 
    I2C_SendByte(Reg_Addr);                     //4
    ret = I2C_WaitToAck();                      //5
    if (ret) goto BAIL;
    
    /* 发送重复 START (SR) */
    I2C_Start();                                //6
 
    I2C_SendByte((slave_Addr << 1) | READ);     //7
    ret = I2C_WaitToAck();                      //8
    if (ret) goto BAIL;
 
    data = I2C_ReadByte();                      //9
    I2C_Ack();                                  //10
    
    crc = I2C_ReadByte();                       //11
    I2C_NoAck();                                //12
    
    I2C_Stop();                                 //13

BAIL:
    /* 失败处理流程 */
}
```

## I2C 速度
```现在有五种运行速度类别：标准模式 (Standard-mode)、快速模式 (Fast-mode, Fm)、快速+模式 (Fast-mode Plus, Fm+)。```

高速模式 (High-speed mode, Hs-mode) 设备向下兼容——任何设备都可以在更低的总线速度下运行。超快速模式 (Ultra Fast-mode) 设备与先前版本不兼容，因为总线是单向的。

- 双向总线：
    - 标准模式 (Standard-mode, Sm)，比特率最高 100 kbit/s
    - 快速模式 (Fast-mode, Fm)，比特率最高 400 kbit/s
    - 快速+模式 (Fast-mode Plus, Fm+)，比特率最高 1 Mbit/s
    - 高速模式 (High-speed mode, Hs-mode)，比特率最高 3.4 Mbit/s
- 单向总线：
    - 超快速模式 (Ultra Fast-mode, UFm)，比特率最高 5 Mbit/s

## I2C 的优点与缺点
I2C 通信的优点？

- 它是同步通信协议，所以主机和从机不需要精确的振荡器。
- 它只需要两根线，一根用于数据 (SDA)，另一根用于时钟 (SCL)。
- 它让用户可以根据需求灵活选择传输速率。
- 在 I2C 总线上，总线上的每个设备都可以独立寻址。
- 它遵循主机和从机关系。
- 它有能力在 I2C 总线上处理多个主机和多个从机。
- I2C 有一些重要特性，如仲裁、时钟同步和时钟拉伸。
- I2C 提供 ACK/NACK（应答/非应答）特性，有助于错误处理。

I2C 接口的局限性是什么？
- 半双工通信，因此一次只沿一个方向（因为单条数据总线）传输数据。
- 由于总线被许多设备共享，调试 I2C 总线（检测哪个设备行为异常）的问题相当困难。
- I2C 总线由多个从设备共享，如果其中任何一个从设备行为异常（无限期地把 SCL 或 SDA 拉低），总线将停滞，不再发生进一步通信。
- I2C 使用阻性上拉 (resistive pull-up) 作为其总线，限制了总线速度。
- 总线速度直接取决于总线电容，意味着更长的 I2C 总线走线会限制总线速度。
- 施加协议开销，降低了吞吐量。
- 需要上拉电阻，这会：
  - 限制时钟速度
  - 在空间极受限的系统中占用宝贵的 PCB 面积
  - 增加功耗

## SMBus - 系统管理总线 (System Management Bus)

```SMBus 允许热插拔，并具有超时特性：如果通信时间过长则重置设备。```

SMBus 使用 I2C 硬件和 I2C 硬件寻址，但增加了第二层软件来构建特殊系统。特别是，它的规范包含一个地址解析协议 (Address Resolution Protocol)，可以进行动态地址分配。

硬件和软件的动态重配置允许总线设备被“热插拔” (hot-plugged) 并立即使用，而无需重启系统。设备被自动识别并分配唯一地址。这一优势带来了即插即用 (plug-and-play) 的用户界面。在两种协议中，系统主机 (System Host) 与系统中所有其他设备（可以具有主机或从机的名称和功能）之间存在一个非常有用的区别。

SMBus 和 I2C 协议基本上是相同的：SMBus 主机能够在协议级控制 I2C 设备，反之亦然。SMBus 时钟定义在 10 kHz 到 100 kHz，而 I2C 根据模式可以是 0 Hz 到 100 kHz、0 Hz 到 400 kHz、0 Hz 到 1 MHz 和 0 Hz 到 3.4 MHz。这意味着以低于 10 kHz 运行的 I2C 总线不符合 SMBus，因为 SMBus 设备可能超时。

### 超时特性 (Time-out feature)
SMBus 具有超时特性，如果通信时间过长则重置设备。I2C 可以是一条“直流”总线，意味着当主机访问某个从机时，从机在执行某些例程时拉伸主机的时钟。这会通知主机从机忙，但不想丢失通信。从机设备会在其任务完成后允许继续。I2C 总线协议对这段延迟没有限制，而对于 SMBus 系统，它被限制在 35 ms。

SMBus 协议只是假设：如果某事耗时过长，说明总线上有问题，所有设备必须重置以清除这种模式。从机设备随后不允许把时钟保持为低过久。

## 参考
[I2C 规范 NXP](https://www.nxp.com/docs/en/user-guide/UM10204.pdf)

https://learn.sparkfun.com/tutorials/i2c/all

https://www.edn.com/design-calculations-for-robust-i2c-communications/

https://www.cnblogs.com/Tangledice/p/7622794.html

https://www.i2c-bus.org/
