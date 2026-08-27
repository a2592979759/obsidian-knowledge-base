---
tags:
  - 操作系统
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Interrupt.md
created: 2026-08-27
---

## 目录
1.  用户空间(User Space)
2.  内核空间(Kernel Space)
3.  虚拟内存(Virtual Memory)
4.  文件系统(File System)
5.  调度(Scheduling)
6.  中断(Interrupt)
    1. 什么是中断(Interrupt)?
        中断是硬件或软件向处理器发出的信号，表示发生了需要立即处理的事件。

        中断有两种类型：软件中断(Software Interrupt)和硬件中断(Hardware Interrupt)。

        硬件中断(external interrupt，外部中断)可以分为两类：
        可屏蔽中断(blockable interrupt)和不可屏蔽中断(unblockable interrupt)。

        可屏蔽中断是那些可以被屏蔽的中断，通常由不太重要的外围设备发出，例如打印机。不可屏蔽中断必须由操作系统处理，例如磁盘读取错误。

    2. 中断操作期间会发生什么?
        每当发生中断时，控制器会完成当前指令的执行，然后开始执行中断服务例程(Interrupt Service Routine, ISR)或中断处理程序(Interrupt Handler)。ISR 告诉处理器或控制器当发生中断时该做什么。中断既可以是硬件中断，也可以是软件中断。

        ![alt text](http://hi.csdn.net/attachment/200910/18/10307_1255838664t2Or.jpg "中断分类")

    3. 中断是如何实现的?
        1. 上半部/下半部 (Upper half/bottom half)
        2. tasklet

        有用的链接:

        [CPU中断机制](https://blog.csdn.net/qq_36894974/article/details/79172603)
        [中断、异常与系统调用 Interrupts, Exceptions, and System Calls](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/5_Interrupts.pdf)
7.  系统调用(System call)
8.  进程间通信(Interprocess communication)
9.  多处理/多线程(Mutiprocessing/Multithreading)
10. RTOS


## 参考(Reference)

[Linux 内核中的中断 Interrupt in Linux Kernel](https://linux-kernel-labs.github.io/refs/heads/master/lectures/interrupts.html#:~:text=In%20Linux%20the%20interrupt%20handling,interrupt%20and%20the%20interrupt%20controller.)

[操作系统如何工作的基本原理 Basics of How Operating Systems Work](http://faculty.salina.k-state.edu/tim/ossg/Introduction/OSworking.html)

[ARM Cortex-M 中断](https://www.youtube.com/watch?v=uFBNf7F3l60&list=PLRJhV4hUhIymmp5CCeIFPyxbknsdcXCc8&index=9&ab_channel=EmbeddedSystemswithARMCortex-MMicrocontrollersinAssemblyLanguageandC)
