---
tags:
  - 嵌入式
  - 计算机体系结构
  - DMA
source: "Computer_architecture/dma.md"
created: 2026-08-27
---

# DMA 简介 (Introduction to DMA)

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入钻研
>
> 将这些体系结构概念作为带参考答案的排序面试题来学习，并配有交互式深入钻研指南。
>
> 👉 **[浏览 MCU 与体系结构问题 →](https://embeddedinterviewlab.com/questions/domain/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=computer_architecture)** &nbsp;·&nbsp; **[浏览 MCU 与体系结构指南 →](https://embeddedinterviewlab.com/categories/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=computer_architecture)**

---

## DMA 简介

#### 什么是 DMA？
DMA (Direct Memory Access，直接内存访问) 指的是 I/O 设备在不需要 CPU 参与的情况下，直接在内存与其之间传输数据的能力。将 DMA 与程序控制 I/O (programmed I/O) 对比：在程序控制 I/O 中，CPU 通过 load 和 store 指令显式地复制数据。

#### 为什么要用 DMA？
* 更高的传输带宽（一次总线操作而非两次）
* 在传输期间 CPU 可以去处理其他事情
* 更快、更一致的响应时间（没有中断或轮询开销）

#### 为什么不使用 DMA？
* 硬件更复杂：I/O 设备必须知道如何成为总线主设备 (bus master)、发起事务等（同时仍要扮演从设备角色）
* 软件更复杂
* DMA 传输会把总线从 CPU 手里拿走（可能限制 CPU 做其他事情的能力）
* 设置传输有开销：需要传输足够多的字节才能抵消这个开销

#### DMA 控制器中需要设置哪些寄存器？
* 内存缓冲区起始地址
* 要传输的字节数
* 传输方向（来自/发往设备，总线上的读/写）
* 设备相关的控制项（例如磁盘访问的 head、cylinder、sector）

#### DMA 传输步骤

###### 图示
![[_assets/DMA_Steps.png]]

###### 细节
1. 程序向设备发出 I/O 请求
2. CPU 执行初始化例程 (initiation routine)（通常是设备驱动程序的一部分）
    * 使用程序控制 I/O（store 指令）来设置控制寄存器
    * 设备特定参数
    * DMA 参数（缓冲区地址/长度）
    * 最后一次写入：使能 (enable/start) 位
3. I/O 设备接口执行传输
    1. 仲裁以获得总线主控权
    2. 把地址放到总线上，断言控制信号以执行读/写（对内存来说看起来就像 CPU）
    3. 如果可用，很可能使用突发传输 (burst transfer)
    4. 大小/类型可以通过控制寄存器编程
    5. I/O 设备提供/消费数据
    6. 地址自增，字节计数递减
    7. 如果字节计数 > 0，则重复
    8. 释放总线，给其他设备机会
    9. 如果字节计数 == 0，在状态寄存器中设置完成位，产生中断
4. CPU 的 ISR 执行完成例程 (completion routine)（也是驱动程序的一部分）
    1. 检查错误，必要时重试
    2. 通知程序传输已完成（例如在内存中设置标志变量）
    3. 如果合适，设置下一次传输（调用初始化例程）
5. 程序注意到请求已完成
