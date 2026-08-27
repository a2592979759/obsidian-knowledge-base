---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Preparation/prepare.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入学习
>
> 用社区评分的题库、带 AI 反馈的编码练习和面试指南来完善你的准备。
>
> 👉 **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

## 通用嵌入式面试的准备事项


### 操作系统/计算机体系结构：
- 进程与线程（Process and Threads）
- 进程调度（Process Scheduling）
- 同步 IPC 机制（mutex/spinlock/semaphore）
- 内存管理（虚拟内存/分页/地址转换, Virtual memory/paging/translation）
- 生产者消费者问题（Producer and Consumer Problem）
- 缓存（缓存一致性/缓存行, cache coherency/cache line）
- 内存映射 IO/IO 映射 IO（Memory mapped IO/IO Mapped IO）
- 中断（Interrupt）**(已完成)**
- 上下文切换（进程/ISR, Context switch）
- 寄存器/指令/执行（Registers/instructions/execution）**(已完成)**
- ARM 程序员模型（ARM programmer's model）**(已完成)**

### 总线协议（Bus protocol）**(已完成)**
- UART **(已完成)**
- SPI **(已完成)**
- I2C **(已完成)**
- RS232/422/485 **(已完成)**

### Linux 内核概念：
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

### C 编程：
- 什么是 static 关键字？**(已完成)**
- volatile 是干什么用的？**(已完成)**
- 宏的使用（Macro usage）**(已完成)**

### 网络
- OSI 网络分层（Network OSI layers）**(已完成)**
- TCP/UDP
- IP **(已完成)**
- 以太网（Ethernet）
- 路由器/交换机（Router/Switch）

### 算法与数据结构
- 链表（Linked list）**(已完成)**
  - 反转链表 **(已完成)**
  - 从链表中删除一个节点 **(已完成)**
  - 删除重复项 **(已完成)**
  - 检测环 **(已完成)**
  - 用链表实现哈希表 **(已完成)**
  - 用链表实现队列 **(已完成)**
- 字符串（String）
  - 反转字符串 **(已完成)**
  - 检查是否为回文（palindrome）**(已完成)**
  - 实现 strstr() **(已完成)**
  - 实现 atoi()/itoa()
- 环形缓冲区（Circular buffer）
- 状态机（State machine）**(已完成)**
- 栈（Stack）**(已完成)**
- 队列（Queue）**(已完成)**
- 二叉搜索树（Binary search tree）**(已完成)**
  - DFS **(已完成)**
  - BFS **(已完成)**
- 内存 API（Memory api）
  - 安全的 memcpy/memmove **(已完成)**
  - sizeof **(已完成)**
  - 对齐内存分配（alligned_malloc）**(基本完成)**
  - 用固定字节 API 读写任意字节的封装 API
- 位操作（Bitewise operation）**(已完成)**
  - 汉明距离（hamming distance）**(已完成)**
  - 不用 '+' 做加法（A plus B）**(已完成)**
  - 反转位（reverse bits）**(已完成)**
  - 2 的幂（Power of 2）**(已完成)**
  - 二进制加法（Add binary）**(已完成)**
  - 只出现一次的数（Single number）**(已完成)**
- 检查字节序（Check endianess）**(已完成)**

---

## 相关页面

- [[onSite_prepare]]
- [[embedded_interview_questions]]
- [[Common_embedded_interview]]
- [[Concept_questions]]
- [[00-索引]]

返回索引 [[00-索引]]
