---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/longpollingWebSocketsServerEvents.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 长轮询 vs WebSocket vs 服务器发送事件（Long-Polling vs WebSockets vs Server-Sent Events）
```长轮询（Long-Polling）、WebSocket 和服务器发送事件（Server-Sent Events，SSE）是在客户端（如 Web 浏览器）和 Web 服务器之间流行的通信协议。```

首先，让我们理解一个标准 HTTP Web 请求是什么样的。以下是常规 HTTP 请求的一系列事件：

1. 客户端打开一个连接，并向服务器请求数据。
2. 服务器计算响应。
3. 服务器在已打开的请求上向客户端发送响应。

### **Ajax 轮询（Ajax Polling）**

轮询（Polling）是绝大多数 AJAX 应用所采用的标准技术。基本思想是客户端反复轮询（或请求）服务器以获取数据。客户端发出一个请求，然后等待服务器返回数据。如果没有可用数据，则返回一个空响应。

1. 客户端打开连接，并使用常规 HTTP 向服务器请求数据。
2. 请求的网页以固定间隔（例如 0.5 秒）向服务器发送请求。
3. 服务器计算响应并发送回去，就像常规 HTTP 流量一样。
4. 客户端周期性重复上述三个步骤，以从服务器获取更新。
5. 轮询的问题在于，客户端必须不断地询问服务器是否有新数据。结果，大量的响应都是空的，从而产生 HTTP 开销。

### **HTTP 长轮询（HTTP Long-Polling）**

这是传统轮询技术的一种变体，允许服务器在数据可用时随时向客户端推送信息。

```使用长轮询时，客户端像正常轮询一样向服务器请求信息，但期望服务器可能不会立即响应。这就是为什么此技术有时被称为“挂起的 GET（Hanging GET）”。```

- 如果服务器没有可供客户端使用的数据，它不会发送空响应，而是持有该请求并等待，直到某些数据变得可用。
- 一旦数据可用，就会向客户端发送一个完整响应。客户端随后立即重新向服务器请求信息，这样服务器就几乎总是有一个等待中的可用请求，可以用来在响应事件时交付数据。

使用 HTTP 长轮询的应用的基本生命周期如下：

1. 客户端使用常规 HTTP 发出初始请求，然后等待响应。
2. 服务器延迟其响应，直到有可用的更新或发生超时。
3. 当有更新可用时，服务器向客户端发送一个完整响应。
4. 客户端通常会发送一个新的长轮询请求，要么在收到响应后立即发送，要么在等待一段可接受的延迟时间后发送。
5. 每个长轮询请求都有一个超时。由于超时导致连接关闭后，客户端必须周期性地重新连接。

### **WebSocket**

WebSocket 在单个 TCP 连接上提供全双工（Full duplex）通信信道。它在客户端与服务器之间提供一条持久连接，双方可以使用它在任何时间开始发送数据。客户端通过一个称为 WebSocket 握手（WebSocket handshake）的过程建立 WebSocket 连接。***如果该过程成功，那么服务器和客户端就可以在任何时间双向交换数据***。WebSocket 协议以较低的开销实现客户端与服务器之间的通信，促进与服务器之间的实时数据传输。这是通过为服务器提供一种标准化的方式来实现的，即无需被客户端请求就能向浏览器发送内容，并允许在保持连接打开的同时来回传递消息。通过这种方式，客户端与服务器之间可以进行双向（bi-directional）的持续对话。

### **服务器发送事件（Server-Sent Events，SSEs）**

```从服务器到客户端的单向会话。```

在 SSE 下，客户端与服务器建立持久且长期连接。**服务器使用此连接向客户端发送数据。** 如果客户端想要向服务器发送数据，则需要使用另一种技术/协议。

1. 客户端使用常规 HTTP 向服务器请求数据。
2. 请求的网页打开到服务器的一个连接。
3. 每当有新信息可用时，服务器就向客户端发送数据。

当我们只需从服务器到客户端的实时流量，或者当服务器循环生成数据并会向客户端发送多个事件时，SSE 是最佳选择。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[systemDesign]] —— 系统设计总览
- [[crossMCUComm]] —— 跨 MCU 通信
- [[loadBalancing]] —— 负载均衡
- [[proxies]] —— 代理服务器
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征

返回索引 [[00-索引]]
