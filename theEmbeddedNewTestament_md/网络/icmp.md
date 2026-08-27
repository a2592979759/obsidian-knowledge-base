---
tags:
  - 网络
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/icmp.md
created: 2026-08-27
---

## 互联网控制消息协议 (Internet Control Message Protocol, ICMP)

由于 IP 没有内置的机制来发送错误和控制消息，它依赖互联网控制消息协议 (ICMP) 来提供错误控制。它用于报告错误和管理查询。它是一种支持协议，被路由器等网络设备用来发送错误消息和操作信息。

ICMP 通常被视为 IP 的一部分，但在架构上**它正好位于 IP 之上**，因为 ICMP 消息承载在 IP 数据报内部。也就是说，ICMP 消息作为 IP 载荷被承载，就像 TCP 或 UDP 报文段作为 IP 载荷被承载一样。类似地，当主机收到将 ICMP 指定为上层协议的 IP 数据报时，它将数据报的内容解复用到 ICMP，就像它会把数据报的内容解复用到 TCP 或 UDP 一样。

## ICMP 头部和消息类型

![[_assets/icmp_header.png]]

ICMP 消息有一个类型字段和一个代码字段，并且包含头部以及触发 ICMP 消息生成的那个 IP 数据报的前 8 个字节（这样发送方就能确定导致错误的数据报）。

![[_assets/icmp_message_types.png]]

## 应用

- **Ping**：著名的 ping 程序向指定主机发送 ICMP 类型 8 代码 0 的消息。目的主机看到回显请求，发回类型 0 代码 0 的 ICMP 回显应答。

- **Traceroute**：源端的 Traceroute 向目的端发送一系列普通的 IP 数据报。这些数据报中的每一个都携带一个带有一个不太可能使用的 UDP 端口号的 UDP 报文段。当第 n 个数据报到达第 n 个路由器时，第 n 个路由器观察到该数据报的 TTL 刚刚过期。根据 IP 协议的规则，路由器丢弃该数据报并向源端发送一条 ICMP 警告消息（类型 11 代码 0）。这条警告消息包含路由器的名称及其 IP 地址。当这条 ICMP 消息回到源端时，源端从计时器获得往返时间，并从 ICMP 消息中获得第 n 个路由器的名称和 IP 地址。其中有一个数据报最终会一路到达目的主机。由于该数据报带有一个使用不太可能端口号的 UDP 报文段，目的主机会向源端发送一个端口不可达的 ICMP 消息（类型 3 代码 3）。

### Ping 是如何工作的？

1. 当 ping 程序初始化时，它会打开一个原始 ICMP 套接字，以便可以直接使用 IP，绕过 TCP 和 UDP。
2. Ping 格式化一个 ICMP 类型 8 消息，即回显请求 (Echo Request)，并使用“sendto”函数将其发送到指定的目标地址。系统提供 IP 头部和数据链路层封装。
3. 随着 ICMP 消息被接收，ping 有机会检查每个数据包，挑出那些感兴趣的项目。
4. 通常行为是截取 ICMP 类型 0 消息，即回显应答 (Echo Replies)，其标识字段值与程序的 PID 匹配。

[Ping 究竟如何工作](http://www.galaxyvisions.com/pdf/white-papers/How_does_Ping_Work_Style_1_GV.pdf)

这篇启发性的文章详细介绍了著名的 ping 应用程序是如何工作的。

## 参考

https://www.geeksforgeeks.org/internet-control-message-protocol-icmp/

https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol#:~:text=The%20ICMP%20header%20starts%20after,code%20of%20that%20ICMP%20packet.
