---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Preparation/onSite_prepare.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入学习
>
> 用社区评分的题库、带 AI 反馈的编码练习和面试指南来完善你的准备。
>
> 👉 **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

## 现场面试准备

### 现场面试流程拆解

***Facebook***

1. 编码轮 I
2. 编码轮 II
3. 行为轮（Behavior round）
4. 领域内系统设计（In-domain system design）
5. 通用系统设计（General system design）

***Amazon***

1. 编码轮 I
2. 编码轮 II
3. 行为轮
4. 领域内系统设计
5. 通用系统设计

***Apple***

1. 编码轮 I
2. 编码轮 II
3. 行为轮
4. 领域内系统设计
5. 通用系统设计

### 编码轮准备
**Leetcode：**

[编码轮面试指南](LeetCode_for_embedded_advanced)

**算法与数据结构：**
- 链表（Linked list）
  - 反转链表
  - 从链表中删除一个节点
  - 删除重复项
  - 检测环
  - 用链表实现哈希表
  - 用链表实现队列
- 字符串（String）
  - 反转字符串
  - 检查是否为回文（palindrome）
  - 实现 strstr()
  - 实现 atoi()/itoa()
- 环形缓冲区（Circular buffer）
- 状态机（State machine）
- 栈（Stack）
- 队列（Queue）
- 二叉搜索树（Binary search tree）
  - DFS
  - BFS
- 内存 API（Memory api）
  - 安全的 memcpy/memmove
  - sizeof
  - 对齐内存分配（alligned_malloc）
  - 用固定字节 API 读写任意字节的封装 API
- 位操作（Bitewise operation）
  - 汉明距离（hamming distance）
  - 不用 '+' 做加法（A plus B）
  - 反转位（reverse bits）
  - 2 的幂（Power of 2）
  - 二进制加法（Add binary）
  - 只出现一次的数（Single number）
- 检查字节序（Check endianess）
- 排序算法（Sorting Algorithm）
  - 冒泡排序（Bubble Sort）
  - 归并排序（Merge Sort）
  - 快速排序（Quick Sort）

### 行为轮准备
[行为面试指南](Behavior/general.md)

### 系统设计轮准备
**缓存（Cache）**


### 领域内设计轮

**操作系统/计算机体系结构：**
- 进程与线程（Process and Threads）
- 进程调度（Process Scheduling）
- 同步 IPC 机制（mutex/spinlock/semaphore）
- 内存管理（虚拟内存/分页/地址转换, Virtual memory/paging/translation）
- 生产者消费者问题（Producer and Consumer Problem）
- 缓存（缓存一致性/缓存行, cache coherency/cache line）
- 内存映射 IO/IO 映射 IO（Memory mapped IO/IO Mapped IO）
- 中断（Interrupt）
- 上下文切换（进程/ISR, Context switch）
- 寄存器/指令/执行（Registers/instructions/execution）
- ARM 程序员模型（ARM programmer's model）

**总线协议（Bus protocol）**
- UART
- SPI
- I2C
- RS232/422/485

**Linux 内核概念：**
- Linux 启动序列（Linux boot sequence）
- 缓冲区共享：DMA Buf & ION
- 等待事件/等待队列（Wait events/Wait queues）
- ISR 处理（上半部/下半部, Top half/Bottom half）
- 下半部——Tasklet/Workqueue/SoftIRQ
- 平台驱动（Platform driver）
- 驱动探测（模块初始化、驱动注册、兼容字符串, Driver probe: Module init, driver register, compatibility string）
- 设备树（设备节点与解析, Device Tree: Device nodes and parsing）
- 时钟/稳压器/GPIO/Pinctrl（Clocks/Regulators/GPIO/Pinctrl）
- IOMMU/MMU
- 定时器库（Timer Library）
- IOCTL、notify dirent
- kmalloc/vmalloc
- kmap/mmap/ioremap
- sysfs/debugfs/procfs

**C 编程：**
- 什么是 static 关键字？
- volatile 是干什么用的？
- 宏的使用（Macro usage）

**网络（Network）**
- OSI 网络分层（Network OSI layers）
- TCP/UDP
- IP
- 以太网（Ethernet）
- 路由器/交换机（Router/Switch）

---

## 相关页面

- [[prepare]]
- [[embedded_interview_questions]]
- [[LeetCode_for_embedded_advanced]]
- [[00-索引]]

返回索引 [[00-索引]]
