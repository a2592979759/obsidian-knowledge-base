---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/indexes.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 索引（Indexes）

说到数据库，索引（indexes）是众所周知的。迟早会有一个时刻，数据库性能不再令人满意。当这种情况发生时，你应该最先考虑的事情之一就是数据库索引。

在数据库的某张特定表上创建索引的目标，是让搜索该表并找到我们想要的行或行组变得更快。索引可以使用数据库表的一个或多个列来创建，为快速随机查找（random lookups）和高效访问有序记录（ordered records）提供基础。

```索引就像目录，它提供对大型数据库内容的快速检索。``` 

### **示例：图书馆目录（Example: A Library Catalog）**

图书馆目录是一种登记册，其中包含图书馆里所有书籍的列表。目录的组织方式类似于数据库表，通常有四列：书名、作者、主题和出版日期。通常有两个这样的目录：一个按书名排序，另一个按作者姓名排序。这样，你可以想到一位你想阅读的作者，然后浏览他的书；或者在你不知道作者姓名时，查找一本你确切知道想读的具体书名。这些目录就像是书籍数据库的索引。它们提供了容易按相关信息进行搜索的排序后数据列表。

简单来说，索引是一种数据结构，可以被视为一个目录，它将我们指向真实数据所在的位置。因此，当我们在表的某一列上创建索引时，我们会把该列以及一个指向整行的指针存储在索引中。

就像传统的关系型数据存储一样，我们也可以将此概念应用于更大的数据集。索引的技巧在于，我们必须仔细考虑用户将如何访问数据。对于规模达到数 TB 但有效载荷（payload）非常小（例如 1 KB）的数据集，索引是优化数据访问所必需的。在如此大的数据集中找到一个小载荷可能是一个真正的挑战，因为我们不可能在合理的时间内遍历那么多数据。此外，如此大的数据集很可能分布在多个物理设备上——这意味着我们需要某种方法来找到所需数据的正确物理位置。索引是做到这一点的最佳方式。

### **索引如何降低写入性能？（How do Indexes decrease write performance?）**

索引可以 ***极大地加速数据检索，但它本身可能因为额外的键而变得很大，这会使数据插入和更新变慢***。

当为一个带有活动索引的表添加行或更新现有行时，我们不仅要写入数据，还必须更新索引。这会降低写入性能。这种性能退化适用于该表的所有插入、更新和删除操作。因此，应当避免在表上添加不必要的索引，并且应当移除不再使用的索引。重申一下，添加索引是为了提高搜索查询的性能。

```如果数据库的目标是提供一个经常被写入而很少被读取的数据存储，那么在这种情况下，降低更常见操作（即写入）的性能，可能并不值得我们用从读取中获得的性能提升来交换。```

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[dataPartitioning]] —— 数据分区
- [[caching]] —— 缓存
- [[systemDesign]] —— 系统设计总览
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征
- [[loadBalancing]] —— 负载均衡

返回索引 [[00-索引]]
