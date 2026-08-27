---
tags:
  - 网络
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/Multicast.md
created: 2026-08-27
---

## 组播 (Multicast)

### **简介**

[***5 分钟组播入门视频***](https://www.youtube.com/watch?v=W5oMvrMRM3Q&ab_channel=DataKnox)

这是一个 5 分钟的短视频，向你概述组播的应用/用途/设计。

[***组播入门 - 斯坦福***](https://www.slac.stanford.edu/grp/scs/net/talk/multicast-slac/sld005.htm)

这是斯坦福 SLAC 制作的一份简短的组播入门幻灯片。

[***通过 TCP/IP 实现组播 HOWTO - Linux 文档项目***](https://tldp.org/HOWTO/Multicast-HOWTO.html)

一个介绍性链接，谈论组播的细节。

### **IP 组播路由**

[[_assets/Introduction_to_IP_Multicast_Routing.pdf|IP 组播路由介绍论文 - 斯坦福]]

本文的第一部分描述了组播的优势、组播骨干网 (Multicast Backbone, MBONE)、D 类寻址以及互联网组管理协议 (Internet Group Management Protocol, IGMP) 的运作。

第二部分探讨了一系列组播路由协议可能采用的算法：

- 泛洪 (Flooding)
- 生成树 (Spanning Trees)
- 逆向路径广播 (Reverse Path Broadcasting, RPB)
- 截断的逆向路径广播 (Truncated Reverse Path Broadcasting, TRPB)
- 逆向路径组播 (Reverse Path Multicasting, RPM)
- 核心基树 (Core-Based Trees)

第三部分是本文的主体。它描述了前述算法如何在当今可用的组播路由协议中实现。
- 距离向量组播路由协议 (Distance Vector Multicast Routing Protocol, DVMRP)
- 组播 OSPF (Multicast OSPF, MOSPF)
- 协议无关组播 (Protocol-Independent Multicast, PIM)

======================================================

[***组播入门 - Cisco***](https://www.cisco.com/c/dam/en/us/products/collateral/ios-nx-os-software/ip-multicast/prod_presentation0900aecd80310883.pdf)

来自 Cisco 的更详细的幻灯片，涵盖以下主题：

- IP 组播入门
- 部署 IP 组播
- 组播安全
- 组播网络管理
- 高级 IP 组播
- IP 组播架构
- Catalyst 6500 的故障排除
- TECRST-1008 企业级 IP 组播

### **IGMP**

[***什么是 IGMP - Geeks for Geeks***](https://www.geeksforgeeks.org/what-is-igmpinternet-group-management-protocol/)

这是一篇关于 IGMP 如何工作及其应用的通用介绍文章。

### **IGMP 侦听**

[***IPv4 组播流量的 IGMP 侦听 - Cisco***](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst6500/ios/12-2SY/configuration/guide/sy_swcg/ipv4_igmp_snooping.pdf/ipv6_mld_snooping.pdf)

关于 IGMP 侦听如何为二层设备工作的更详细描述。

## 参考
