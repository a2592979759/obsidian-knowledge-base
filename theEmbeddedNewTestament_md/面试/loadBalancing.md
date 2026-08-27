---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/loadBalancing.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 负载均衡（Load Balancing）

负载均衡器（Load Balancer，LB）是任何分布式系统的另一个关键组件。它有助于将流量分散到一组服务器上，以提高应用、网站或数据库的响应能力和可用性。负载均衡器在分发请求的同时，还跟踪所有资源的状态。如果某台服务器无法接取新请求、没有响应或错误率升高，负载均衡器就会停止向该服务器发送流量。

通常，负载均衡器位于客户端与服务器之间，接收传入的网络和应用流量，并使用各种算法将流量分发到多个后端服务器上。通过在多个服务器之间平衡应用请求，负载均衡器降低了单台服务器的负载，并防止任何一台应用服务器成为单点故障，从而提高整体的应用可用性和响应能力。

![负载均衡器（Load Balancer）](https://miro.medium.com/max/1586/1*tEaZGz-p1-E2ytNjl5RPJg.jpeg)

为了充分利用可扩展性和冗余，我们可以尝试在系统的每一层都进行负载均衡。我们可以在三个位置添加负载均衡器（LB）：

- 用户与 Web 服务器之间
- Web 服务器与内部平台层（例如应用服务器或缓存服务器）之间
- 内部平台层与数据库之间。

![[_assets/load_balancer.png]]

### **负载均衡的好处（Benefits of Load Balancing）**

- 用户体验到更快、不间断的服务。用户不必等待某台苦苦挣扎的服务器完成其先前任务。相反，他们的请求会立即被传递给一个更可用的资源。
  
- 服务提供商体验到更少的停机时间和更高的吞吐量。即使某台服务器完全故障，也不会影响最终用户的体验，因为负载均衡器会将其绕过，路由到一台健康的服务器。
  
- 负载均衡使系统管理员更容易处理传入请求，同时减少用户的等待时间。
  
- 智能负载均衡器提供诸如预测性分析（predictive analytics）之类的优势，在流量瓶颈发生之前就将其识别出来。因此，智能负载均衡器为组织提供了可行的洞察。这些对于自动化至关重要，并能帮助推动业务决策。
  
- 系统管理员体验到更少的组件故障或过载。负载均衡不是让单个设备执行大量工作，而是让多个设备各自执行一小部分工作。

### **负载均衡算法（Load Balancing Algorithms）**

***负载均衡器如何选择后端服务器？***

负载均衡器在将请求转发到后端服务器之前会考虑两个因素。它们首先会确保所选择的服务器确实能适当地响应请求，然后使用预先配置的算法从健康的服务器集合中选择一台。我们马上就会讨论这些算法。

***健康检查（Health Checks）*** - 负载均衡器只应该将流量转发给“健康的”后端服务器。为了监控后端服务器的健康状况，“健康检查”会定期尝试连接到后端服务器，以确保服务器正在监听。如果某台服务器未通过健康检查，它会自动从服务器池中移除，并且在它再次响应健康检查之前，不会将流量转发给它。

算法：

- 最少连接法（Least Connection Method）— 该方法将流量导向当前活动连接最少的服务器。当存在大量持久连接且这些连接在服务器之间分布不均时，这种方法非常有用。
  
- 最小响应时间法（Least Response Time Method）— 该算法将流量导向活动连接最少且平均响应时间最低的服务器。
  
- 最少带宽法（Least Bandwidth Method）— 该方法选择当前以每秒兆比特（Mbps）衡量的服务流量最少的服务器。
  
- 轮询法（Round Robin Method）— 该方法在服务器列表中循环，并将每个新请求发送给下一台服务器。到达列表末尾后，又从开头重新开始。当服务器规格相同且没有太多持久连接时，它最为有用。
  
- 加权轮询法（Weighted Round Robin Method）— 加权轮询调度旨在更好地处理具有不同处理能力的服务器。每台服务器被分配一个权重（一个表示处理能力的整数值）。权重高的服务器比权重低的服务器先接收新连接，权重高的服务器比权重低的服务器获得更多的连接。
  
- IP 哈希（IP Hash）— 在该方法下，会计算客户端 IP 地址的哈希，以将请求重定向到一台服务器。

### **冗余负载均衡器（Redundant Load Balancers）**

负载均衡器可能成为单点故障；为了克服这一点，可以将第二个负载均衡器连接到第一个负载均衡器，从而组成一个集群。每个负载均衡器都会监控另一个的健康状况，并且由于它们都能同样胜任流量服务与故障检测，在主负载均衡器故障时，第二个负载均衡器会接管。

### 进阶阅读（Advance Readings）

[什么是负载均衡（What is load balancing）](https://avinetworks.com/what-is-load-balancing/)

[面向规模的系统架构入门（Introduction to architecting systems for scale）](https://lethain.com/introduction-to-architecting-systems-for-scale/)


## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[caching]] —— 缓存
- [[systemDesign]] —— 系统设计总览
- [[dataPartitioning]] —— 数据分区
- [[consistentHashing]] —— 一致性哈希
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征

返回索引 [[00-索引]]
