---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/keyCharacterDistributedSystem.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 分布式系统的关键特征（Key Characteristics of Distributed Systems）
- 可扩展性（Scalability）
- 可靠性（Reliability）
- 可用性（Availability）
- 效率（Efficiency）
- 可维护性（Serviceability or Manageability）

### ***可扩展性（Scalability）***

可扩展性是一个系统、进程或网络增长并管理日益增长需求的能力。任何能够持续演进以支持不断增长工作量的分布式系统，都被认为是可扩展的。

系统可能出于许多原因而需要扩展，例如数据量增加或工作量增加（例如事务数量）。一个可扩展系统希望在扩展时保持性能不下降。

通常，一个系统的性能尽管被设计（或声称）为可扩展，但随着系统规模增大，由于管理或环境成本而下降。例如，网络速度可能会变慢，因为机器之间往往相隔较远。更一般地说，某些任务可能无法分布，要么是因为它们固有的原子性（atomic）本质，要么是因为系统设计中的某些缺陷。在某个时刻，这样的任务会限制由分布式带来的加速。一个可扩展的架构会避免这种情况，并尝试在参与的所有节点之间均匀平衡负载。

**水平扩展 vs. 垂直扩展（Horizontal vs. Vertical Scaling）**：水平扩展（horizontal scaling）是指通过向资源池中添加更多服务器来扩展，而垂直扩展（vertical scaling）是指通过向现有服务器添加更多算力（CPU、RAM、存储等）来扩展。

使用水平扩展往往更容易动态扩展，只需向现有池中添加更多机器；垂直扩展通常受限于单台服务器的容量，超过该容量的扩展往往涉及停机，并且有一个上限。

### ***可靠性（Reliability）***

根据定义，可靠性是一个系统在给定时期内发生故障的概率。简而言之，一个分布式系统即使在其一个或多个软件或硬件组件发生故障时仍能持续提供服务，就被认为是可靠的。可靠性代表了任何分布式系统的主要特征之一，因为在这种系统中，任何一台出故障的机器总是可以被另一台健康的机器替换，从而保证请求任务的完成。

一个可靠的分布式系统通过软件组件和数据的冗余来实现这一点。显然，冗余是有代价的，一个可靠系统必须付出这个代价，通过消除每一个单点故障来实现这种服务韧性（resilience）。

### ***可用性（Availability）***

根据定义，可用性是一个系统在特定时期内保持运行以执行其所需功能的时间。它是对一个系统、服务或机器在正常条件下保持运行的时间百分比的一种简单衡量。一架每月可以飞行很多小时而几乎不停飞的飞机，可以说具有高可用性。可用性考虑了可维护性、维修时间、备品可用性以及其他后勤因素。如果一架飞机因维护而停飞，那么在那段时间内它被认为是不可用的。

可靠性是考虑到可能出现的各种真实世界条件的可用性。一架能够安全度过任何可能天气的飞机，比一架对可能出现的情况存在弱点的飞机更可靠。

**可靠性 vs. 可用性（Reliability Vs. Availability）**
如果一个系统是可靠的，那它就是可用的。然而，如果一个系统是可用的，它并不一定是可靠的。换句话说，高可靠性有助于实现高可用性，但即使产品不可靠，也可以通过最小化维修时间并确保备件在需要时始终可用，来实现高可用性。

### ***效率（Efficiency）***

衡量其效率的两个标准是：响应时间（response time）或延迟（latency），它表示获得第一项数据的延迟；以及吞吐量（throughput）或带宽（bandwidth），它表示在给定时间单位（例如一秒）内交付的数据项数量。这两个衡量标准对应于以下单位成本：

- 系统节点全局发送的消息数量，无论消息大小如何。
- 消息的大小，代表数据交换的容量。

由分布式数据结构支持的操作的复杂性（例如，在分布式索引中搜索特定键）可以表征为这些成本单位之一的函数。

### ***可维护性（Serviceability or Manageability）***

在设计分布式系统时的另一个重要考虑因素是，它的运行和维护有多容易。可维护性是指系统被维修或维护的难易程度和速度；如果修复一个故障系统的时间增加，那么可用性就会降低。对于可维护性需要考虑的事项包括：在问题出现时诊断和理解问题的难易程度、进行更新或修改的难易程度，以及系统运行起来有多简单。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[systemDesign]] —— 系统设计总览
- [[CAPTheorem]] —— CAP 定理
- [[Redundancy&Replication]] —— 冗余与复制
- [[dataPartitioning]] —— 数据分区
- [[indexes]] —— 索引

返回索引 [[00-索引]]
