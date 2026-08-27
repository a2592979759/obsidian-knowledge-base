---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/proxies.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 代理服务器（Proxies）

代理服务器（proxy server）是位于客户端与后端服务器之间的中间服务器。客户端连接到代理服务器来请求诸如网页、文件、连接等服务。

```简而言之，代理服务器是一种软件或硬件，充当客户端向其他服务器请求资源的中间人。```

通常，代理被用来 ***过滤请求、记录请求，或者有时转换请求***（通过添加/移除头、加密/解密，或压缩资源）。代理服务器的另一个优点是 ***它的缓存可以服务大量请求***。如果多个客户端访问某个特定资源，代理服务器可以将其缓存并服务给所有客户端，而无需访问远程服务器。

![代理服务器（Proxies server）](https://s3-us-west-1.amazonaws.com/umbrella-blog-uploads/wp-content/uploads/2020/02/24162741/What-is-a-Proxy-Server_Cisco-Umbrella-Blog-Forward-Proxy-Reverse-Proxy-Example-Image-1024x427.png)

### **代理服务器类型（Proxy Server Types）**

代理可以驻留在客户端的本地服务器上，也可以位于客户端与远程服务器之间的任何位置。以下是几个著名的代理服务器类型：

***开放代理（Open Proxy）***

开放代理是任何互联网用户都可以访问的代理服务器。通常，代理服务器只允许网络组内的用户（即封闭代理，closed proxy）存储和转发诸如 DNS 或网页等互联网服务，以减少和控制该组使用的带宽。然而，对于开放代理，互联网上的任何用户都能使用此转发服务。有两种著名的开放代理类型：

- **匿名代理（Anonymous Proxy）：** 该代理揭示自己是一台服务器，但不披露初始 IP 地址。尽管这个代理服务器很容易被发现，但对某些用户来说可能是有益的，因为它隐藏了他们的 IP 地址。

- **透明代理（Transparent Proxy）：** 这种代理服务器同样表明自己的身份，并且在 HTTP 头的支持下，最初的 IP 地址可以被看到。使用这类服务器的主要好处在于它能够缓存网站。

***反向代理（Reverse Proxy）***

反向代理代表客户端从一台或多台服务器检索资源。然后这些资源被返回给客户端，看起来就好像它们源自代理服务器本身一样。

## 参考（Reference）

Grokking the System Design Interview by Educative.io

## 相关页面
- [[loadBalancing]] —— 负载均衡
- [[caching]] —— 缓存
- [[systemDesign]] —— 系统设计总览
- [[longpollingWebSocketsServerEvents]] —— 长轮询/WebSocket/服务器事件
- [[keyCharacterDistributedSystem]] —— 分布式系统关键特征

返回索引 [[00-索引]]
