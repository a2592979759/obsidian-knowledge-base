---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/Redundancy&Replication.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 冗余与复制（Redundancy and Replication）

### **冗余（Redundancy）**
冗余是指 ***复制系统的关键组件或功能***，其目的是提高系统的可靠性，通常以备份或故障安全（fail-safe）的形式出现，或者用来改善系统的实际性能。例如，如果某个文件只有一份副本且仅存储在一台服务器上，那么失去这台服务器就意味着丢失这个文件。由于丢失数据通常不是好事，我们可以创建文件的重复或冗余副本来解决这个问题。

```冗余在消除系统中的单点故障（single points of failure）方面起着关键作用，并在危机发生时提供所需的备份。例如，如果生产环境中运行着同一服务的两个实例，其中一个发生故障，系统就可以故障转移（failover）到另一个。```


### **复制（Replication）**

复制是指 ***共享信息以确保冗余资源之间的一致性***，这些资源可以是软件或硬件组件，其目的是提高可靠性、容错性（fault-tolerance）或可访问性。

复制被广泛应用于许多数据库管理系统（DBMS）中，通常在原始数据与副本之间采用主从（primary-replica）关系。主服务器（primary server）获得所有更新，然后将这些更新传递给从服务器（replica servers）。每个副本都会输出一条消息，表明它已成功接收更新，从而允许发送后续更新。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征
- [[dataPartitioning]] —— 数据分区
- [[CAPTheorem]] —— CAP 定理
- [[systemDesign]] —— 系统设计总览
- [[caching]] —— 缓存

返回索引 [[00-索引]]
