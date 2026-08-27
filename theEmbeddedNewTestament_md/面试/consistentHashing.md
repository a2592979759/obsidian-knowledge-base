---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/consistentHashing.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 一致性哈希（Consistent Hashing）

分布式哈希表（Distributed Hash Table，DHT）是分布式可扩展系统所使用的基本组件之一。哈希表需要一个键（key）、一个值（value）和一个哈希函数（hash function），其中哈希函数将键映射到值所存储的位置。

假设我们要设计一个分布式缓存系统。给定 'n' 个缓存服务器，一个直观的哈希函数是 'key % n'。它简单且常用。但它有两个主要缺点：

1. 它不能横向扩展（horizontally scalable）。每当向系统添加一个新的缓存主机时，所有现有的映射都会失效。如果缓存系统中包含大量数据，这在维护上会是一个痛点。实际上，要安排停机时间来更新所有的缓存映射是很困难的。
   
2. 它可能无法负载均衡（load balanced），尤其是对于分布不均匀的数据。在实际中，可以轻松假定数据不会均匀分布。对于缓存系统而言，这意味着一些缓存变得过热（hot）且饱和，而其他缓存则空闲且几乎为空。

在这种情况下，一致性哈希是改进缓存系统的好方法。

### **什么是一致性哈希？（What is Consistent Hashing?）**

一致性哈希对于分布式缓存系统和 DHT 是一种非常有用的策略。它允许我们以某种方式将数据分布到集群中，这种方式 **能最大限度地减少在添加或移除节点时所需的重组。因此，缓存系统将更容易扩展或缩减**。

```在一致性哈希中，当哈希表调整大小时（例如向系统添加一个新的缓存主机），只需要重新映射 'k/n' 个键，其中 'k' 是键的总数，'n' 是服务器的总数。回想一下，在使用 'mod' 作为哈希函数的缓存系统中，所有键都需要被重新映射。```

在一致性哈希中，对象会尽可能映射到同一台主机。当一台主机从系统中移除时，该主机上的对象由其他主机分担；当添加一台新主机时，它会从少数几台主机那里取走自己应得的部分，而不影响其他主机的份额。

### **它是如何工作的？（How does it work?）**

作为一个典型的哈希函数，一致性哈希将键映射到一个整数。假设哈希函数的输出范围是 [0, 256]。想象该范围内的整数被放置在一个环（ring）上，使得这些值首尾相连。

一致性哈希的工作方式如下：

1. 给定一组缓存服务器，将它们哈希到该范围内的整数。
2. 要将一个键映射到某台服务器，
    1. 将其哈希为一个整数。
    2. 在环上顺时针移动，直到找到遇到的第一个缓存。
    3. 该缓存就是包含此键的缓存。参见下面的动画示例：key1 映射到缓存 A；key2 映射到缓存 C。

![一致性哈希（Consistent Hashing）](https://uploads.toptal.io/blog/image/129309/toptal-blog-image-1551794743400-9a6fd84dca83745f8b6ca95a2890cdc2.png)

要添加一台新服务器，比如 D，原本位于 C 的键将被拆分。其中一部分会被转移到 D，而其他键则不受影响。

要移除一个缓存，或者如果某个缓存发生故障，比如 A，所有原本映射到 A 的键都会落入 B，只有那些键需要被移动到 B；其他键不受影响。

对于负载均衡，正如我们开头讨论的，真实数据本质上是被随机分布的，因此可能并不均匀。这可能会使缓存上的键不平衡。

为了处理这个问题，我们为缓存添加“虚拟副本（virtual replicas）”。我们不再将每个缓存映射到环上的单一节点，而是将其映射到环上的多个节点，即副本。这样，每个缓存就与环上的多个部分相关联。

如果哈希函数“混排得足够好”，随着副本数量的增加，键会更均衡。

### **进阶阅读（Advance Reading）**
[一致性哈希终极指南（The Ultimate Guide For consistent Hashing）](https://www.toptal.com/big-data/consistent-hashing)

在这篇文章中，它首先回顾了哈希的一般概念及其目的，接着描述了分布式哈希及其带来的问题。随后，这就把我们引向了标题中的主题。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[consistentHashing_impl]] —— 一致性哈希实现
- [[caching]] —— 缓存
- [[dataPartitioning]] —— 数据分区
- [[memCache]] —— memCache 实现
- [[systemDesign]] —— 系统设计总览

返回索引 [[00-索引]]
