---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Company/Verkada/Phone_screen.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的嵌入式题库、带 AI 反馈的编码练习以及系统设计指南进行准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)**

---

# 准备（Preparation）

招聘人员（Recruiter）提到这次面试将围绕***并发（concurrency）***展开。

### 关于并发的领域内知识（In-domain knowledge about concurrency）

1. 进程与线程（Process & Threads）
   1. 多线程（Multithreading）
   2. 多进程（Multiprocessing）
   3. 调度器与分派器（Scheduler & dispatcher）
2. 同步（Synchronization）
   1. 互斥量（Mutex）
   2. 条件变量（Condition Variables）
   3. 信号量（Semaphore）
   4. 自旋锁（spinlocks）
   5. 死锁与优先级反转（Deadlock & priority inversion）
   6. 可重入性（reentrancy）
   7. volatile（易变变量）
3. 进程间通信（IPC, Inter-Process Communication）
   1. 信号（Signal）
   2. 管道（Pipe）
   3. linux
      1. POSIX 线程（POSIX threads）
      2. fork
      3. 消息队列（messagequeue）
      4. 管道（pipe）
   4. 实时操作系统（RTOS, e.g. freeRTOS）
      1. 信号（signal）
      2. 信号量（Semaphore）
      3. 互斥量（mutex）
      4. 消息队列（messagequeue）


### 并发编码题（Concurrency Coding Problems）

1. [生产者与消费者问题（Producer and consumer problem）](https://shivammitra.com/c/producer-consumer-problem-in-c/#)
   1. [读者-写者问题（Reader Writer Problem）](https://shivammitra.com/reader-writer-problem-in-c/)
   2. [FCFS 调度算法（FCFS Scheduling Algorithm）](https://codophobia.github.io/operating%20system/fcfs-scheduling-program/)
   3. [非抢占式优先级调度（Nonpreemptive Priority Scheduling）](https://shivammitra.com/operating%20system/nonpreemptive-priority-scheduling/)
   4. [抢占式优先级调度（Preemptive Priority Scheduling）](https://shivammitra.com/operating%20system/preemptive-priority-program/)
   5. [SJF 调度（SJF Scheduling）](https://shivammitra.com/operating%20system/sjf-scheduling-program/)
   6. [SRTF 调度（SRTF Scheduling）](https://shivammitra.com/operating%20system/srtf-scheduling-program/)
   7. [时间片轮转调度（Round Robin Scheduling）](https://shivammitra.com/operating%20system/roundrobin-scheduling-program/)
2. 环形缓冲区读写（ring buffer read/write）
3. [Leetcode 并发标签问题（Leetcode Concurrency tag problems）](https://leetcode.com/problemset/concurrency/)

### 电话面试问题（Phone Screen Questions）

## 相关页面
- [[Phone_screen]] —— Verkada 电话面试
- [[topics_prepare]] —— Facebook 面试主题准备
- [[commonBehavior]] —— 常见行为面试题
- [[prepare]] —— 通用嵌入式面试准备清单
- [[onSite_prepare]] —— 现场面试准备

返回索引 [[00-索引]]
