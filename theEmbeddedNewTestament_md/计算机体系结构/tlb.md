---
tags:
  - 嵌入式
  - 计算机体系结构
  - TLB
source: "Computer_architecture/tlb.md"
created: 2026-08-27
---

# TLB 简介 (Introduction to TLB)

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入钻研
>
> 将这些体系结构概念作为带参考答案的排序面试题来学习，并配有交互式深入钻研指南。
>
> 👉 **[浏览 MCU 与体系结构问题 →](https://embeddedinterviewlab.com/questions/domain/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=computer_architecture)** &nbsp;·&nbsp; **[浏览 MCU 与体系结构指南 →](https://embeddedinterviewlab.com/categories/mcu-architecture?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=computer_architecture)**

---

## TLB 简介

#### 什么是 TLB？
翻译后备缓冲 (Translation Lookaside Buffer, TLB) 不过是一种特殊的缓存，用于跟踪最近使用过的地址翻译。TLB 中包含最近使用过的页表项 (page table entries)。给定一个虚拟地址 (virtual address)，处理器会检查 TLB 中是否存在对应的页表项（TLB 命中 / TLB hit），此时取出帧号并构成真实地址。如果在 TLB 中找不到页表项（TLB 未命中 / TLB miss），则用页号作为索引去处理页表。TLB 首先检查该页是否已在主内存中；如果不在主内存中，则产生一次缺页 (page fault)，然后更新 TLB 以包含新的页表项。

#### TLB 内部包含什么？
典型的 TLB 可能有 32、64 或 128 个条目，并且被称为全相联 (fully associative)。基本上这意味着任何一次翻译可以位于 TLB 中的任何位置，而且硬件会并行地搜索整个 TLB 以找到所需的翻译。一个典型的 TLB 条目看起来像这样：**[VPN | PFN | 其他位 (other bits)]**。更有趣的是那些其他位。例如，TLB 通常有一个**有效位 (valid bit)**，它指示该条目是否包含有效的翻译。同样常见的还有**保护位 (protection bits)**，用于决定一个页如何被访问（就像在页表中那样）。例如，代码页可能被标记为可读可执行，而堆页可能被标记为可读可写。还可能有一些其他字段，包括地址空间标识符 (address space identifier)、**脏位 (dirty bit)** 等等。

#### TLB 命中 vs TLB 未命中
![[_assets/tlb.jpg]]
![[_assets/tlb_flow.png]]

###### TLB 命中 (TLB hit)
从虚拟地址中提取虚拟页号 (VPN, virtual page number)，并检查 TLB 是否保存了该 VPN 的翻译。如果保存了，就是一次 TLB 命中。

###### TLB 未命中 (TLB miss)
如果 CPU 在 TLB 中找不到该翻译（TLB 未命中），硬件会访问页表来找到翻译，并且假设该进程产生的虚拟内存引用是有效且可访问的，就把该翻译更新到 TLB 中。最后，一旦 TLB 更新完毕，硬件重试当前指令。这一次，翻译在 TLB 中找到，内存引用就被快速处理了。
