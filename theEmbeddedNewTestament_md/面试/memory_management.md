---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/embeddedDesignTopics/memory_management.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 小型内存软件的内存分配（Memory Allocation for Small memory software）

### **分配技术（Allocation technique）**

- 固定分配（FIXED ALLOCATION） 在系统开始运行时预先分配对象

- 内存丢弃（MEMORY DISCARD） 以组为单位分配临时（transient）对象，通常位于栈（stack）上。

- 可变分配（VARIABLE ALLOCATION） 根据需要从堆（heap）中动态分配对象。

- 池化分配（POOLED ALLOCATION） 从预先分配的内存空间动态分配对象。

### **设计内存分配时需考虑的问题（Issues to consider when designing memory allocation）**

***碎片化（Fragmentation）***

碎片化是动态内存分配的一个重大问题。有两种碎片化：内部碎片化（internal fragmentation），当一个数据结构没有使用它被分配的全部内存时；以及外部碎片化（external fragmentation），当位于两个已分配结构之间的内存无法使用时，通常是因为它太小而无法存储其他任何东西。

***内存耗尽（Memory Exhaustion）***

无论你选择哪种分配策略，你都永远不可能有足够的内存来满足所有可能的情况：你可能通过固定分配没有预先分配足够的对象；或者从堆或栈内存请求可变分配时可能失败；或池化分配的内存池可能为空。迟早你会耗尽内存。在规划内存分配时，你还需要考虑如何处理内存耗尽。

- **固定大小的客户端内存（Fixed Size Client Memories）。** 你可以直接向你的用户或客户端组件暴露一个固定大小的内存模型。例如，许多掌上计算器让用户在十个内存中选择一个来保存一个值，而不会暗示系统可能有更多内存；许多组件在其接口（连接、任务、操作等）中支持最多固定数量的对象，并在超过该数量时产生错误。这种方法易于编程，但它降低了系统的可用性，因为它让用户或客户端组件完全承担处理内存耗尽的责任。

- **发出错误信号（Signal an error）。** 你可以向客户端发出内存耗尽错误信号。这种方法也让客户端负责处理该故障，但通常会给它们留下比提供固定数量的用户内存更多的选择。例如，如果图形编辑器程序没有足够的内存来处理大型图像，用户可能更愿意关闭其他应用以释放整个系统的更多内存。信号错误在内部更有问题，当一个组件向另一个组件发送错误时。尽管向客户端组件通知内存错误相当简单，通常通过异常或返回码实现，但编程让客户端组件正确处理错误则要困难得多（参见 PARTIAL FAILURE 模式）。

- **降低质量（Reduce quality）。** 你可以通过降低需要存储的数据的质量来减少需要分配的内存量。例如，你可以截断字符串并降低声音和图像的采样频率。降低质量可以维持系统吞吐量，但如果它丢弃了对用户重要的数据，则不可取。例如，在网络监控应用中使用更小的图像可能没问题，但在图形处理程序中则不行。

- **删除旧对象（Delete old objects）。** 你可以删除旧或不重要的对象来为新的或重要的对象释放内存。例如，电话交换机在创建新连接时可能耗尽内存，但它们可以通过终止已响铃最久的连接来重新获得内存，因为它最不可能被接听（FRESH WORK BEFORE STALE，[Meszaros 1998]）。类似地，许多消息日志通过只存储一定数量的消息并在新消息到达时删除旧消息，来防止溢出。

- **推迟新请求（Defer new requests）。** 你可以延迟分配请求（以及依赖它们的处理），直到有足够的内存可用。最简单且最常见的方法是让系统在当前任务完成之前不接受更多输入。例如，许多 MS Windows 应用将指针改为“请稍候”图标，通常是沙漏，意味着用户在此操作完成之前不能做其他任何事情。许多通信系统都有“流量控制（flow control）”机制来阻止进一步输入，直到当前输入已被处理。更简单的是批处理式处理，从文件或数据库中顺序读取元素，只有在你处理完上一个之后才读取下一个。更复杂的方法需要在系统中使用并发，使得一个任务可以阻塞或排队由另一个任务处理的请求。许多环境支持同步原语（synchronisation primitives），比如信号量（semaphores），或更高级的管道（pipes）或共享队列（shared queues），它们可以在无法满足请求时自动阻塞其客户端。在单线程系统中，组件接口可以支持回调（callbacks）或轮询（polling）来通知其客户端它们已完成处理某个请求。

- **忽略问题（Ignore the problem）。** 你可以完全忽略问题，让程序发生故障。不幸的是，这种策略在许多环境中是默认做法，尤其是在分页虚拟内存（paged virtual memory）被视为理所当然的地方。例如，互联网蠕虫（Internet worm）就是通过 UNIX finger 守护进程中的一个 bug 传播的，其中长消息可能会覆盖固定大小的缓冲区 [Page 1988]。这种方法实现起来微不足道，但可能产生极其严重的后果：利用 finger bug 的蠕虫使大部分互联网瘫痪了数天。

### **专用模式（Specialized Patterns）**

- **固定分配（FIXED ALLOCATION）** 通过预先分配结构来满足你的需求，并避免在正常处理期间进行动态内存分配，从而确保你始终有足够的内存。
- **可变分配（VARIABLE ALLOCATION）** 通过使用动态分配从堆中获取和归还内存，避免未使用的空闲内存空间。
- **内存丢弃（MEMORY DISCARD）** 通过将临时对象放入临时工作区并一次性丢弃整个工作区，简化临时对象的释放。
- **池化分配（POOLED ALLOCATION）** 通过按需预先分配大量相似对象，并维护一个可复用对象的“空闲列表（free list）”，避免可变分配的开销。
- **压缩（COMPACTION）** 通过在内存中移动已分配对象来移除碎片空间，从而避免内存碎片化。
- **引用计数（REFERENCE COUNTING）** 通过维护对每个共享对象的引用计数，并在其引用计数为零时删除每个对象，来管理共享对象。
- **垃圾回收（GARBAGE COLLECTION）** 通过定期识别未引用的对象并删除它们，来管理共享对象。

[内存分配器（A memory Allocator）](http://gee.cs.oswego.edu/dl/html/malloc.html)

[小型内存链接（Small Memory Links）](http://smallmemory.com/SmallMemoryLinks.html)

[小型内存软件（Small Memory Software）](http://smallmemory.com/ThinkingSmall.pdf)

## 相关页面
- [[crossMCUComm]] —— 跨 MCU 通信
- [[airControlSystem]] —— 空调控制系统示例
- [[systemDesign]] —— 系统设计总览
- [[cacheDesign]] —— 缓存设计示例
- [[embedded_interview_questions]] —— 嵌入式综合面试题

返回索引 [[00-索引]]
