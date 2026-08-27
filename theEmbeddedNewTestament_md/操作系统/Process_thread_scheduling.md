---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Process/Process_thread_scheduling.md
created: 2026-08-27
---


## OS 调度(OS Scheduling)

### 调度器与分派器(Scheduler and dispatcher)

### 分派器(Dispatcher)

### 调度算法/策略(Scheduling algorithm/policy)

### 上下文切换(Context Switch)

### 什么是 PCB(进程控制块 process control block)?
与每个进程相关联的信息(也称为任务控制块 task control block)。进程表、任务结构(task struct)或切换帧(switchframe)中的条目是操作系统内核中的一种数据结构，包含管理特定进程调度所需的信息。

* 进程状态 – 运行中、等待中等等
* 程序计数器 – 下一条要执行的指令的位置
* CPU 寄存器 – 所有与进程相关的寄存器的内容
* CPU 调度信息 – 优先级、调度队列指针
* 内存管理信息 – 分配给该进程的内存
* 记账信息 – 已用 CPU 时间、自启动以来经过的时钟时间和时间限制
* I/O 状态信息 – 分配给进程的 I/O 设备、打开的文件列表

https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/

### 什么是 TCB(线程控制块 thread control block)?

与表示进程的进程控制块(PCB)非常相似，线程控制块(TCB)表示系统中生成的线程。它包含关于线程的信息，例如其 ID 和状态。

其组成部分定义如下:

* 线程 ID: 这是操作系统在创建线程时分配给线程的唯一标识符。
* 线程状态: 这些是线程的状态，随着线程在系统中推进而变化。
* CPU 信息: 它包含操作系统需要了解的一切，例如线程已经推进了多远以及正在使用什么数据。
* 线程优先级: 它表示该线程相对于其他线程的权重(或优先级)，这有助于线程调度器确定接下来应该从就绪队列(READY queue)中选择哪个线程。
* 一个指向触发创建此线程的进程的指针。
* 一个指向此线程创建的线程的指针。

https://www.geeksforgeeks.org/thread-control-block-in-operating-system/?ref=rp

### 调度标准(Scheduling criterias)

* CPU 利用率。我们希望尽可能让 CPU 保持繁忙。概念上，CPU 利用率可以从 0% 到 100%。在真实系统中，它应处于 40%(轻负载系统)到 90%(重负载系统)之间。
* 吞吐量(Throughput)。如果 CPU 忙于执行进程，那么工作就完成了。工作的一种度量是每个时间单位内完成的进程数量，称为吞吐量。
* 周转时间(Turnaround time)。从特定进程的角度来看，重要的标准是执行该进程需要多长时间。从提交进程到完成之间的时间间隔就是周转时间。周转时间是花在等待进入内存、在就绪队列中等待、在 CPU 上执行以及进行 I/O 的时间之和。
* 等待时间(Waiting time)。CPU 调度算法不影响进程执行或进行 I/O 的时间量，它只影响进程在就绪队列中等待的时间量。等待时间是花在就绪队列中等待的时段之和。
* 响应时间(Response time)。从提交请求直到产生第一个响应的时间。这个度量称为响应时间，它是开始响应所花的时间，而不是输出响应所花的时间。

### 多线程的好处(Benefits of multithreading)
多线程编程的好处可以归纳为四大类:

* 响应性(Responsiveness)。将交互式应用进行多线程化，即使程序的一部分被阻塞或正在执行耗时操作，也能让程序继续运行，从而提升对用户的响应性。
* 资源共事(Resource sharing)。进程只能通过共享内存或消息传递等技术共享资源。这些技术必须由程序员显式安排。然而，线程默认共享它们所属进程的内存和资源。
* 经济性(Economy)。为进程创建分配内存和资源是昂贵的。因为线程共享它们所属进程的资源，所以创建和上下文切换线程更加经济。经验上衡量开销差异可能很困难，但一般来说，创建和管理进程比线程要耗时得多。
* 可扩展性(Scalability)。多线程的好处可以在多处理器架构中得到显著提升，此时线程可以在不同处理器上并行运行。单线程进程只能在一个处理器上运行，无论有多少可用的处理器。在多 CPU 机器上进行多线程化会提高并行度。
