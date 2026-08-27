---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/systemDesign.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 系统设计面试准备（System Design Interview Preparation）

## 4S 法（4S Approach）
- 场景（Scenario）
- 服务（Service）
- 存储（Storage）
- 规模（Scale） 

## 7 步法（7-step Approach）
### **第 1 步：明确需求（Step 1: Requirements clarifications）**

针对我们试图解决问题的确切范围提出疑问，总是一个好主意。设计类问题大多是开放式的，并没有一个唯一正确的答案。这就是为什么在面试早期澄清歧义变得至关重要。花足够时间定义系统最终目标的候选人，总是有更大机会在面试中取得成功。此外，由于我们只有 35-40 分钟来设计一个（据称）大型系统，我们应该明确我们将重点关注系统的哪些部分。

### **第 2 步：粗略估算（Step 2: Back-of-the-envelope estimation）**

估算我们要设计的系统的规模，总是一个好主意。这也能在之后专注于扩展、分区、负载均衡和缓存时派上用场。

系统预期达到什么规模（例如，新推文数量、推文浏览量、每秒时间线生成数等）？
我们需要多少存储空间？如果用户可以在推文中包含照片和视频，我们就需要不同的存储需求。
我们预期的网络带宽使用量是多少？这在决定如何管理流量以及在服务器之间平衡负载时至关重要。

### **第 3 步：系统接口定义（Step 3: System interface definition）**

定义系统需要提供哪些 API。这将确立系统所期望的确切契约，并确保我们没有遗漏任何需求。我们这类 Twitter 风格服务的一些 API 示例可以是：

```
postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, …)  
generateTimeline(user_id, current_time, user_location, …)  
markTweetFavorite(user_id, tweet_id, timestamp, …)  
```

### **第 4 步：定义数据模型（Step 4: Defining data model）**

在面试早期定义数据模型，将澄清数据如何在不同的系统组件之间流动。之后，它将为数据分区和管理提供指导。候选人应该识别系统的各个实体、它们之间如何互动，以及数据管理的不同方面，如存储、传输、加密等。以下是我们这类 Twitter 风格服务的一些实体：

```
User: UserID, Name, Email, DoB, CreationData, LastLogin, etc.
Tweet: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.
UserFollow: UserID1, UserID2
FavoriteTweets: UserID, TweetID, TimeStamp
```
### **第 5 步：高层设计（Step 5: High-level design）**

绘制一个包含 5-6 个框的框图，代表我们系统的核心组件。我们应该识别出足够多的组件，以端到端地解决实际问题。

对于 Twitter，在高层次上，我们需要多个应用服务器来服务所有的读写请求，并在它们前面放置负载均衡器以进行流量分发。如果我们假定会有比写入多得多的读取流量，我们可以决定使用独立的服务器来处理这些场景。在后端，我们需要一个高效的数据库来存储所有的推文，并支持海量的读取。我们还需要一个分布式文件存储系统来存储照片和视频。

- 由于我们要存储海量数据，我们应该如何对数据进行分区以分发到多个数据库？ 
- 我们是否应该尝试把某个用户的全部数据存储在同一数据库中？这可能导致什么问题？
- 我们将如何处理高热度用户（hot users），他们大量发推或关注很多人？
- 由于用户的时间线会包含最近（也最相关）的推文，我们是否应该尝试存储数据，使其针对扫描最新推文进行优化？
- 我们应在多大程度上以及在哪个层级引入缓存来加速？
- 哪些组件需要更好的负载均衡？

### **第 6 步：详细设计（Step 6: Detailed design）**
对两三个主要组件进行深入探讨；面试官的反馈应该始终引导我们了解系统的哪些部分需要进一步讨论。我们应该提出不同的方案、它们的优缺点，并解释为什么我们会偏爱一种方案而非另一种。记住，没有唯一答案；唯一重要的是在考虑系统约束的同时，权衡不同选项之间的取舍。

- 由于我们要存储海量数据，我们应该如何对数据进行分区以分发到多个数据库？我们是否应该尝试把某个用户的全部数据存储在同一数据库中？这可能导致什么问题？
- 我们将如何处理高热度用户，他们大量发推或关注很多人？
- 由于用户的时间线会包含最近（也最相关）的推文，我们是否应该尝试存储数据，使其针对扫描最新推文进行优化？
- 我们应在多大程度上以及在哪个层级引入缓存来加速？
- 哪些组件需要更好的负载均衡？

### **第 7 步：识别并解决瓶颈（Step 7: Identifying and resolving bottlenecks）**
尝试讨论尽可能多的瓶颈，以及缓解它们的不同方法。

我们的系统中是否存在任何单点故障？我们正在做什么来缓解它？
我们是否有足够的数据副本，以至于即使失去几台服务器，仍然能够服务用户？
同样，我们是否为不同的服务运行了足够的副本，使得少数故障不会导致整个系统停机？
我们如何监控服务的性能？每当关键组件故障或其性能下降时，我们会收到警报吗？

## 相关页面
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征
- [[CAPTheorem]] —— CAP 定理
- [[loadBalancing]] —— 负载均衡
- [[dataPartitioning]] —— 数据分区
- [[caching]] —— 缓存

返回索引 [[00-索引]]
