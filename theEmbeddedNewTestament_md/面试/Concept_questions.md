---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Concept/Concept_questions.md"
created: 2026-08-27
---

> ## 🚀 交互式练习这些嵌入式面试题
>
> 在 **[EmbeddedInterviewLab](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=concept_questions)** 上获取**由社区评分的、可搜索题库**，包含模型答案、点赞，以及真实的"我被问到过"数据。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=concept_questions)** &nbsp;·&nbsp; 📚 **[浏览所有知识主题 →](https://embeddedinterviewlab.com/topics?utm_source=github&utm_medium=referral&utm_campaign=topics&utm_content=concept_questions)**

---

## 面试概念问题

### 概念问题与链接表格

索引 | 问题 | 难度 | 重要性
--|-----------------|----------|---|---
1 | [什么是死锁（Deadlock）？如何避免和预防？](https://practice.geeksforgeeks.org/problems/deadlock-in-os/) | 简单 | ***** 
2 | [#include <FileName> 和 #include "FileName" 的基本区别是什么？](https://practice.geeksforgeeks.org/problems/header-file/) | 简单 | **
3 | [解释操作系统中的抖动（Thrashing）](https://practice.geeksforgeeks.org/problems/thrashing-in-os/) | 中等 | ****
4 | [OSI 模型的不同层有哪些？](https://practice.geeksforgeeks.org/problems/what-are-the-different-layers-of-osi-model/) | 简单 | ***** 
5 | [解释 Linux 中的启动过程（booting process）？](https://practice.geeksforgeeks.org/problems/booting-in-linux) | 简单 | ***** 
6 | [如果你杀死一个父进程会发生什么？](https://practice.geeksforgeeks.org/problems/child-parent-process) | 中等 | ***** 
7 | [什么是信号量（semaphore）？](https://practice.geeksforgeeks.org/problems/what-is-a-semaphore) | 简单 | *****
8 | [进程如何处理中断？](https://practice.geeksforgeeks.org/problems/handle-interrupt) | 简单 | *****
9 | [解释 TCP/IP 模型，以及各层使用的协议及其功能](https://practice.geeksforgeeks.org/problems/tcpip-model/) | 中等 | ***
10 | [C 中的预处理器指令 "pragma"](https://practice.geeksforgeeks.org/problems/pragma-question/) | 中等 | ***
11 | [进程（Process）和线程（thread）的区别](https://practice.geeksforgeeks.org/problems/difference-between-process-and-thread/) | 简单 | *****
12 | [C 中 void 指针的大小](https://practice.geeksforgeeks.org/problems/size-of-void-pointer-in-c) | 简单 | ***
13 | [如何用信号量实现监视器（monitors）？](https://cis.temple.edu/~giorgio/cis307/readings/monitor.html) | 难 | *
14 | [优先级反转（priority inversion）的解决方案是什么？](https://practice.geeksforgeeks.org/problems/what-is-the-solution-to-priority-inversion) | 简单 | ****
15 | [什么是优先级反转（priority inversion）？](https://practice.geeksforgeeks.org/problems/what-is-priority-inversion) | 简单 | ****
16 | [为什么调度器区分 I/O 密集型程序和 CPU 密集型程序很重要？](https://practice.geeksforgeeks.org/problems/why-is-it-important-for-the-scheduler-to-distinguish-i0-bound-programs-from-cpu-bound-programs) | 中等 | **
17 | [什么是处理器亲和性（Processor Affinity）？](https://practice.geeksforgeeks.org/problems/what-is-processor-affinity) | 简单 | **


#### **外设设备通常可以访问处理器的中断，以便在特定事件发生时引起处理器的注意。请简要描述这种中断机制。**
需要考虑的要点： 
1. 什么是中断（Interrupt）？

   中断是硬件或软件向处理器发出的信号，表示一个需要立即关注的事件发生了。 

   中断有两种类型：软件中断（Software）和硬件中断（Hardware）。

   硬件中断（外部中断，external interrupt）可分为两类：
   可屏蔽中断（blockable interrupt）和不可屏蔽中断（unblockable interrupt）。 

   可屏蔽中断是指可以被屏蔽的中断，通常由不那么重要的外设设备发出，例如打印机。不可屏蔽中断必须由操作系统处理，例如磁盘读取错误。 

2. 中断操作期间发生了什么？

   每当发生中断时，控制器会完成当前指令的执行，然后开始执行中断服务例程（Interrupt Service Routine，ISR）或中断处理程序（Interrupt Handler）。ISR 告诉处理器或控制器在中断发生时该做什么。中断可以是硬件中断，也可以是软件中断。

[中断流程](http://hi.csdn.net/attachment/200910/18/10307_1255838664t2Or.jpg)

1. 中断是如何实现的？ 
2. 把大量代码放进中断处理程序往往更简单，但这在实践中可能导致糟糕的结果。为什么？
   1. 上半部/下半部（Upper half/bottom half）
   2. 任务小任务（tasklet）

实用链接：

[CPU 中断机制](https://blog.csdn.net/qq_36894974/article/details/79172603)

[中断、异常与系统调用](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/5_Interrupts.pdf)


#### **什么是信号量（semaphore）和互斥量（mutex）？什么是优先级反转（priority inversion）？在 FreeRTOS 中互斥量与信号量有何不同？**
   1. 信号量和互斥量是任务间的同步与信令（signaling）机制。
   2. 信号量是一种信令机制，分为两类：
      1. 计数信号量（Counting Semaphore）：最常用于管理一个对同时使用用户数量有限制的共享资源。例如同时的 Socket 连接数量。
      ![[_assets/counting_semaphore.png]]

      2. 二进制信号量（Binary Semaphore）是计数上限为 1 的计数信号量，通常用于同步目的。
      ![[_assets/binary_semaphore.png]]

   3. 然而，使用二进制信号量可能导致优先级反转（priority inversion），即低优先级任务抢占高优先级任务的执行。 
   ![[_assets/priority_inversion.png]]

   4. 在 FreeRTOS 中，互斥量与二进制信号量的区别在于：获取了互斥量锁（mutex lock）的任务会得到优先级提升（priority boost），调度器会临时提升持有锁的任务的优先级，使其他任务无法抢占它。一旦互斥量锁被释放，它的优先级就会恢复正常。通过这种方式，它解决了二进制信号量所具有的优先级反转问题。
   ![[_assets/priority_inversion_mutex.png]]

---

## 相关页面

- [[embedded_interview_questions]]
- [[Common_embedded_interview]]
- [[I2C_interview_questions]]
- [[SPI_interview_questions]]
- [[prepare]]

返回索引 [[00-索引]]
