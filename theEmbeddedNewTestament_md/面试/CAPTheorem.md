---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/CAPTheorem.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## CAP 定理（CAP Theorem）

```CAP 定理指出，一个分布式软件系统不可能同时提供以下三种保证中的两种以上（CAP）：一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance）。```

当我们设计一个分布式系统时，在 CAP 之间进行取舍几乎是我们首先要考虑的事情。CAP 定理指出，在设计分布式系统时，我们只能从以下三个选项中选择两个：

***一致性（Consistency）：*** 所有节点在同一时间看到相同的数据。一致性是通过在允许进一步读取之前更新多个节点来实现的。

***可用性（Availability）：*** 每个请求都会得到成功/失败的响应。可用性是通过在不同服务器之间复制数据来实现的。

***分区容错性（Partition tolerance）：*** 尽管存在消息丢失或部分故障，系统仍能继续工作。一个具备分区容错性的系统能够承受任何不会导致整个网络失败的故障。数据在节点和网络的组合中充分复制，以保持系统在间歇性瘫痪期间正常运行。

![CAP 定理（CAP theorem）](https://miro.medium.com/max/922/1*tmttEOAU9xacJgw6vrsAuA.jpeg)

我们无法构建一个既持续可用、又顺序一致、还能容忍任何分区故障的通用数据存储。我们只能构建一个具备这三种属性中任意两种的系统。因为要保持一致性，所有节点都应该以相同的顺序看到相同的更新集。但如果网络失去某个分区，一个分区中的更新可能无法在其他分区被客户端读取之前到达——而这个客户端可能刚读取过新更新的分区，随后又去读取了过时的分区。应对这种可能性的唯一办法是停止服务于来自过时分区的请求，但这样一来服务就不再是 100% 可用了。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[systemDesign]] —— 系统设计总览
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征
- [[dataPartitioning]] —— 数据分区
- [[caching]] —— 缓存
- [[loadBalancing]] —— 负载均衡

返回索引 [[00-索引]]
