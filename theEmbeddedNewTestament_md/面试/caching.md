---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/caching.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 缓存（Caching）

负载均衡（load balancing）帮助你横向扩展（scale horizontally）到越来越多的服务器，而缓存（caching）则让你能够更好地利用已有的资源，并让原本无法实现的产品需求变得可行。缓存利用了局部性引用原理（locality of reference principle）：最近被请求的数据很可能再次被请求。它们几乎应用于计算的每一层：硬件、操作系统、Web 浏览器、Web 应用等等。缓存就像是短期记忆：它的空间有限，但通常比原始数据源更快，并保存着最近访问过的内容。缓存可以存在于架构的所有层级，但通常出现在最靠近前端（front end）的层级，它们在那里被实现，以便快速返回数据而不给下游层级增加负担。

### **应用服务器缓存（Application server cache）**
将缓存直接放在请求层（request layer）节点上，可以实现响应数据的本地存储。每当向服务发出请求时，如果本地缓存中存在数据，节点就会快速返回本地缓存的数据。如果缓存中没有，请求节点就会从磁盘查询数据。请求层节点上的缓存既可以位于内存中（速度非常快），也可以位于节点的本地磁盘上（比访问网络存储更快）。

当你扩展到多个节点时会发生什么？如果请求层被扩展为多个节点，仍然可以让每个节点各自拥有自己的缓存。但是，如果负载均衡器在节点之间随机分发请求，同一个请求就会到达不同的节点，从而增加缓存未命中（cache miss）。克服这一障碍的两个选择是全局缓存（global cache）和分布式缓存（distributed cache）。

### **内容分发网络（Content Distribution Network，CDN）**
CDN 是一种缓存，适用于为大量静态媒体提供服务的网站。在典型的 CDN 设置中，请求会首先向 CDN 请求某段静态媒体；如果 CDN 本地有该内容，就会直接提供。如果不可用，CDN 就会向后端服务器查询该文件，将其缓存到本地，然后提供给请求的用户。

如果我们正在构建的系统规模还不够大，不足以拥有自己的 CDN，我们可以通过使用轻量级 HTTP 服务器（如 Nginx）在单独的子域（例如 static.yourservice.com）上提供静态媒体，来为将来的过渡做准备，之后再通过 DNS 从你的服务器切换到 CDN。

### **缓存失效（Cache Invalidation）**

虽然缓存很棒，但它确实需要一些维护，以保持缓存与事实来源（source of truth，例如数据库）的一致性。如果数据库中的数据被修改，就应该使缓存中的对应数据失效；否则，这会导致应用行为不一致。

解决这个问题被称为缓存失效（cache invalidation）；有三种主要的方案：

***直写缓存（Write-through cache）：*** 在该方案下，数据同时写入缓存和对应的数据库。缓存的数据允许快速检索，而且由于相同的数据被写入永久存储，我们将在缓存和存储之间获得完整的数据一致性。此外，该方案还能确保在崩溃、断电或其他系统中断情况下不会丢失任何东西。

不过，尽管直写最大程度地降低了数据丢失的风险，但由于每次写操作都必须执行两次才能向客户端返回成功，该方案在写操作上存在较高延迟的缺点。

***旁路缓存（Write-around cache）：*** 该技术与直写缓存类似，但数据被直接写入永久存储，绕过缓存。这可以减少缓存被那些随后不会再读取的写操作淹没，但其缺点是，对最近写入的数据进行读取请求会产生“缓存未命中”，必须从较慢的后端存储读取，并承受较高的延迟。

***回写缓存（Write-back cache）：*** 在该方案下，数据只写入缓存，并立即向客户端确认完成。对永久存储的写入在指定时间间隔后或在特定条件下才进行。这对写密集型应用来说带来低延迟和高吞吐量，然而，这种速度伴随着在崩溃或其他不利事件（adverse event）时数据丢失的风险，因为写入数据的唯一副本就在缓存中。

### **缓存淘汰策略（Cache eviction policies）**
以下是一些最常见的缓存淘汰策略：

1. 先进先出（First In First Out，FIFO）：缓存首先淘汰最先访问的块，而不考虑它之前被访问的频率或次数。
2. 后进先出（Last In First Out，LIFO）：缓存首先淘汰最近访问的块，而不考虑它之前被访问的频率或次数。
3. 最近最少使用（Least Recently Used，LRU）：首先丢弃最近最少使用的条目。
4. 最近最常使用（Most Recently Used，MRU）：与 LRU 相反，首先丢弃最近最常使用的条目。
5. 最不经常使用（Least Frequently Used，LFU）：统计一个条目被需要的频率。那些最少被使用的条目首先被丢弃。
6. 随机替换（Random Replacement，RR）：在必要时随机选择一个候选条目并将其丢弃以腾出空间。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[memCache]] —— memCache 实现
- [[consistentHashing]] —— 一致性哈希
- [[loadBalancing]] —— 负载均衡
- [[systemDesign]] —— 系统设计总览
- [[cacheDesign]] —— 缓存设计示例

返回索引 [[00-索引]]
