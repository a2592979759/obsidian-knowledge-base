---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Process/IPC.md
created: 2026-08-27
---

## 进程间通信(Interprocess Communications)

#### **进程间通信的方法**
进程间通信(inter-process communication, IPC)是一组接口，通常为了实现程序在一系列进程之间通信而编写。这让运行中的程序能够在操作系统中并发执行。以下是 IPC 的方法：

- 管道 Pipes(同一进程)
    - 这允许数据只沿一个方向流动。类似于单工系统(键盘)。输出数据通常被缓冲，直到输入进程收到它，二者必须具有共同的起源。
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_pipes.htm)

- 命名管道 Named Pipes(不同进程)
    - 这是一种具有特定名称的管道，可用于没有共同进程起源的进程间。例如 FIFO，其中写入管道的细节首先被命名。
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_named_pipes.htm)

- 消息队列 Message Queuing
    - 这允许使用单个队列或几个消息队列在进程之间传递消息。这由系统内核管理，这些消息通过 API 进行协调。
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_message_queues.htm)

- 信号量 Semaphores
    - 用于解决与同步相关的问题并避免竞态条件(race condition)。它们是大于或等于 0 的整数值。
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_semaphores.htm)

- 共享内存 Shared memory
    - 这允许通过定义的内存区域交换数据。在数据访问共享内存之前必须获得信号量值。
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_semaphores.htm)

- 套接字 Sockets
    - 这种方法主要用于客户端和服务器之间通过网络进行通信。它允许一种标准的连接，与计算机和操作系统无关。

- 信号 Signals
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_signals.htm)

- 内存映射 Memory Mapping
    - [解释与示例](https://www.tutorialspoint.com/inter_process_communication/inter_process_communication_memory_mapping.htm)

#### 详细解释
- [共享地址空间 Shared Address Space](https://courses.engr.illinois.edu/cs241/sp2012/lectures/29-IPC.pdf)(共享内存、内存映射文件)
- [由操作系统从一个地址空间传输到另一个地址空间的消息](https://courses.engr.illinois.edu/cs241/sp2012/lectures/30-IPC.pdf)(文件、管道和 FIFO)
- [一些 POSIX 示例](https://courses.engr.illinois.edu/cs241/fa2010/ppt/31-IPC.pdf)


#### 实现
[Brendan 的多任务教程](https://wiki.osdev.org/Brendan%27s_Multi-tasking_Tutorial)

本教程将描述为使用"每任务内核栈(kernel stack per task)"的内核实现多任务和任务切换的一种方法。它的设计目的是让读者能够以可测试的步骤实现一个功能完善的调度器(同时避免常见陷阱)，然后再进入下一步。
