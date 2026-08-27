---
tags:
  - 网络
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/dhcp.md
created: 2026-08-27
---

## IPv4 寻址
```每个 IP 地址长 32 位（相当于 4 字节），因此总共有 2^32 种可能的 IP 地址。```

主机与物理链路之间的边界称为接口 (interface)。路由器与其任一链路之间的边界也称为接口。因此路由器有多个接口，每个链路一个。由于每台主机和路由器都能发送和接收 IP 数据报，IP 要求每个主机和路由器接口都有自己的 IP 地址。***因此，从技术上讲，IP 地址关联的是接口，而不是包含该接口的主机或路由器。***

![[_assets/interface_connection_1.png]]

用 IP 的术语来说，这个互联三个主机接口和一个路由器接口的网络构成了一个子网 (subnet)。IP 寻址为这个子网分配一个地址：223.1.1.0/24，其中 /24 记法（有时称为子网掩码）表示 32 位数量的最左边 24 位定义了子网地址。

![[_assets/interface_connection_2.png]]

形式为 a.b.c.d/x 的地址的 x 个最高有效位构成 IP 地址的网络部分，通常被称为地址的前缀 (prefix)（或网络前缀）。

## 获取主机地址：DHCP 协议
```主机地址也可以手动配置，但如今这项任务更常使用动态主机配置协议 (Dynamic Host Configuration Protocol, DHCP) 完成```

DHCP 允许主机自动获取（被分配）一个 IP 地址。除了主机 IP 地址分配外，DHCP 还允许主机了解附加信息，例如它的子网掩码、它的首跳路由器（通常称为默认网关）的地址，以及它的本地 DNS 服务器地址。

由于 DHCP 能够自动完成将主机接入网络所涉及的网络相关方面，它常被称为***即插即用协议 (plug-and-play protocol)***。随着主机加入和离开，DHCP 服务器需要更新其可用 IP 地址列表。每次主机加入时，DHCP 服务器从其当前可用地址池中分配一个任意地址；每次主机离开时，它的地址被归还到池中。

DHCP 是一种***客户端-服务器协议 (client-server protocol)***。客户端通常是新加入的主机，想要获取网络配置信息，包括给自己的一个 IP 地址。

![[_assets/dhcp_client_server.png]]

对于新加入的主机，DHCP 协议是一个四步过程（DORA）：

- **DHCP 服务器发现 (DHCP server Discovery)。** 新加入主机的首要任务是找到可以交互的 DHCP 服务器。这是通过 DHCP 发现报文完成的，客户端在 UDP 数据包中将其发送到端口 67。该 UDP 数据包被封装在 IP 数据报中。DHCP 客户端创建一个 IP 数据报，其中包含其 DHCP 发现报文、广播目的 IP 地址 255.255.255.255 以及“本主机”源 IP 地址 0.0.0.0。DHCP 客户端将该 IP 数据报交给链路层，链路层再将此帧广播到子网上所有节点。
- **DHCP 服务器提供 (DHCP server Offer(s))。** 收到 DHCP 发现报文的 DHCP 服务器会向客户端回应一个 DHCP 提供报文，该报文使用 IP 广播地址 255.255.255.255 广播到子网上所有节点。
- **DHCP 请求 (DHCP Request)。** 新加入的客户端会从一个或多个服务器提供中选择一个，并对其选中的提供回应一个 DHCP 请求报文，回显配置参数。
- **DHCP 确认 (DHCP ACK)。** 服务器对 DHCP 请求报文回应一个 DHCP 确认 (ACK) 报文，确认所请求的参数。

![[_assets/dhcp_protocol.png]]

- **DHCP 释放 (DHCP Release)** 最后，如果主机想要移动到其他网络，或者它已完成工作，它会向服务器发送 DHCPRELEASE 数据包，表示它想要断开连接。然后服务器将 IP 地址在存储中标记为可用，以便它可以被分配给其他机器。

## DHCP 报文格式

![[_assets/dhcp_message_format.png]]

[这里是 DHCP 数据包字段的详细解释](http://www.tcpipguide.com/free/t_DHCPMessageFormat.htm)

## 参考

http://www.tcpipguide.com/free/t_DHCPMessageFormat.htm

https://www.geeksforgeeks.org/how-dhcp-server-dynamically-assigns-ip-address-to-a-host/

https://www.youtube.com/watch?v=k4t-NJrKLgM&ab_channel=HowTo
