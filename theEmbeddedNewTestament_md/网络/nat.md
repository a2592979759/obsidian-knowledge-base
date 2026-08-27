---
tags:
  - 网络
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/nat.md
created: 2026-08-27
---

## NAT

**网络地址转换 (Network address translation, NAT)** 是一种通过修改流量路由设备传输途中数据包 IP 头部中的网络地址信息，将一个 IP 地址空间重新映射到另一个 IP 地址空间的方法。该技术最初用于在网络迁移时、或当上游互联网服务提供商被更换但无法路由该网络的地址空间时，避免需要为每台主机分配新地址。在 IPv4 地址枯竭的背景下，它已成为节约全球地址空间的一种流行且必不可少的工具。一个 NAT 网关的单一互联网可路由 IP 地址可以用于整个私有网络。

![[_assets/nat_translation.png]]

## NAT 类型

- **静态 NAT (Static NAT)** – 在这种类型中，单个未注册（私有）IP 地址与一个合法注册的（公共）IP 地址映射，即本地与全局地址之间一一对应。这通常用于 Web 托管。组织内一般不使用这种方式，因为有许多设备需要互联网访问，而要提供互联网访问就需要公共 IP 地址。假设有 3000 台设备需要访问互联网，组织就必须购买 3000 个公共地址，这将非常昂贵。

- **动态 NAT (Dynamic NAT)** – 在这种类型的 NAT 中，未注册的 IP 地址从公共 IP 地址池中被转换为一个已注册的（公共）IP 地址。如果地址池中的 IP 地址不空闲，那么数据包将被丢弃，因为只有固定数量的私有 IP 地址可以被转换为公共地址。这种方式也非常昂贵，因为组织必须购买许多全局 IP 地址来建立地址池。

- **端口地址转换 (Port Address Translation, PAT)** – 这也被称为 NAT 过载 (NAT overload)。在这种类型中，许多本地（私有）IP 地址可以转换为一个已注册的 IP 地址。使用端口号来区分流量，即哪些流量属于哪个 IP 地址。这是最常用的方式，因为它成本效益高，成千上万的用户可以通过仅仅使用一个真实的全局（公共）IP 地址连接到互联网。

## NAT 的优势

- NAT 节约合法注册的 IP 地址。
- 它提供隐私，因为设备的 IP 地址（发送和接收流量的那个）会被隐藏。
- 在网络演进时消除了地址重新编号的需要。

## NAT 的缺点

- 转换导致交换路径延迟。
- 某些应用程序在启用 NAT 时无法正常工作。
- 使 IPsec 等隧道协议复杂化。
- 此外，路由器作为网络层设备，本不应篡改端口号（传输层），但由于 NAT 而不得不这样做。

## 参考

[网络地址转换入门](https://nsrc.org/workshops/2018/btnog-wireless/presentations/00_NAT_Introduction.pdf)

这份幻灯片全面概述了 NAT，并通过校园网络设计与运维研讨班展示了其典型用例。

[Cisco − NAT 如何工作](http://academy.delmar.edu/Courses/download/CiscoIOS/NAT_HowItWorks.pdf)

Cisco 发布的一份文档，给出了 NAT 如何工作的一个有洞见的角度。

[网络地址转换的回顾性审视](http://web.cs.ucla.edu/~lixia/papers/08IEEE-NAT-Retrospect.pdf)

如今，网络地址转换器（即 NAT）无处不在。它们的普及并非由设计或规划推动，而是由互联网的持续增长推动，这不仅对 IP 地址空间提出了日益增长的需求，也对网络地址转换被认为能够促进的其他功能需求提出了要求。本文从个人视角介绍了 NAT 的历史，以回顾性的方式审视了它们的优缺点，以及我们可以从 NAT 的经验中学到的教训。

[NAT 维基百科](https://en.wikipedia.org/wiki/Network_address_translation)

[NAT GeeksforGeeks](https://www.geeksforgeeks.org/network-address-translation-nat/)

[NAT 的类型](https://www.geeksforgeeks.org/types-of-network-address-translation-nat/?ref=lbp)
