---
tags:
  - 网络
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/LTE.md
created: 2026-08-27
---

## **简介**

[***LTE 百科***](https://sites.google.com/site/lteencyclopedia/home)

这是一个全面介绍 LTE 技术的网站。

## **架构**

![[_assets/LTE_architecture.png]]

[***LTE 架构基础***](https://www.netmanias.com/en/post/techdocs/5904/lte-network-architecture/lte-network-architecture-basic)

本文档作为“LTE”领域的首篇技术文档，对 LTE 网络架构进行了简要概述。首先定义 LTE 网络参考模型，并描述其基本的分组演进系统 (Evolved Packet System, EPS) 实体及各实体的功能。接着描述 EPS 实体之间的接口以及跨接口的协议栈。最后解释用户流量如何通过 LTE 网络交付以提供互联网服务。

[[_assets/Long-term_evolution_network_architecture.pdf|长期演进网络架构]]

这是一篇描述 LTE 网络架构的详细论文。

用户面协议栈 (User Plane Protocol Stack)：

![[_assets/LTE_user_plane_protocol_stack.png]]

控制面协议栈 (Control Plane Protocol Stack)：

![[_assets/LTE_control_plane_protocol.png]]

## **LTE 网络**

### **LTE 标识符**

LTE 标识：

![LTE 标识](https://www.netmanias.com/en/?m=attach&no=3068)

[***UE 与 ME 标识符***](https://www.netmanias.com/en/?m=view&id=techdocs&no=5905&tag=80&page=4)

作为 LTE 标识的第一篇文档，本文档（第一部分，LTE 标识 I）将 LTE 标识分为不同的组，并描述其中两组，即用户设备标识符 (User Equipment Identifiers, UE IDs) 和移动设备标识符 (Mobile Equipment identifiers, ME IDs)。首先解释诸如 IMSI、GUTI、S-TMSI、IP 地址和 C-RNTI 等 UE ID，然后讨论在 S1-MME 和 X2 接口上标识的 UE ID。接着解释诸如 IMEI 和 IMEISV 等 ME ID。最后简要总结 UE 与 ME ID 的特性。

[***NE 与位置标识符***](https://www.netmanias.com/en/?m=view&id=techdocs&no=5906&tag=80&page=4)

作为 LTE 标识的第二篇文档，本文档（第二部分，LTE 标识 II）描述网络设备标识符 (Network Equipment identifiers, NE IDs) 和位置标识符组。MME、eNB 和 P-GW 等一些 NE 属于 NE ID 组，本文先解释 GUMMEI、MMEI、Global eNB ID、eNB ID、ECGI、ECI 和 P-GW ID 等 NE ID。然后讨论标识 UE 位置的 TAC、TAI 等位置 ID。最后简要总结这些 ID 的特性。

[***EPS 会话/承载标识符***](https://www.netmanias.com/en/?m=view&id=techdocs&no=5907)

作为 LTE 标识的第三篇文档，本文档（第三部分，LTE 标识 III）涵盖与用户流量交付相关的 EPS 会话/承载 ID 组。文中描述了会话/承载 ID，如分组数据网络 (Packet Data Network, PDN) ID（接入点名称 (Access Point Name, APN)）、EPS 承载 ID、E-RAB ID、数据无线承载 (Data Radio Bearer, DRB) ID、隧道端点标识符 (Tunnel Endpoint Identifier, TEID) 和链接的 EPS 承载标识 (Linked EPS Bearer Identity, LBI)，随后总结这些 ID 的特性。最后列出三篇 LTE 标识文档所涵盖的所有 LTE ID。

### **LTE IP 地址分配**

IP 分配类型：

![LTE IP 分配](https://www.netmanias.com/en/?m=attach&no=17118)

动态 IP 地址分配：

![动态 IP 地址分配](https://www.netmanias.com/en/?m=attach&no=17117)

静态 IP 地址分配：

![静态 IP 地址分配](https://www.netmanias.com/en/?m=attach&no=17116)

[***LTE IP 地址分配方案 I：基础***](https://www.netmanias.com/en/?m=view&id=techdocs&no=7246&tag=80)

本文档将描述 LTE 网络如何为访问网络的用户分配 IP 地址。根据分配方式的不同，IP 地址可以是动态的也可以是静态的。下面我们将讨论这两种类型有何不同，以及它们是如何分配的。

[***LTE IP 地址分配方案 II：双城案例***](https://www.netmanias.com/en/?m=view&id=techdocs&no=7257)

本文档介绍了一个具体的 IP 地址分配案例——在 LTE 网络内地理上分离的位置进行分配。在动态分配的情况下，无论用户在何处接入，动态选出的 P-GW 都会为用户动态分配一个用于 PDN 连接的 IP 地址。而在静态分配的情况下，用户总是对应一个特定的 P-GW 和一个 IP 地址——指定的 P-GW 会为用户分配一个静态 IP 地址用于 PDN 连接。这里我们将以服务两个城市的 LTE 网络为例，描述 IP 地址分配的不同方式和过程，并看看它们之间有何不同。

## **LTE EPS 移动性管理**

### EPS 移动性管理场景

![EPS 场景](https://www.netmanias.com/en/?m=attach&no=3423)

[***EMM 场景中的十一个 EMM 案例***](https://www.netmanias.com/en/?m=view&id=techdocs&no=6002)

本文档通过使用一个 EMM 场景并在其中定义十一个 EMM 案例，来定义将在后续文档中进一步讨论的 EMM 过程。它简要解释了每个 EMM 案例中的用户体验和设备操作，并讨论在 EMM 过程前后 UE 的 EMM、ECM 和 RRC 状态如何变化。

### **LTE 切换**

[***无 TAU 的切换 - 第 1 部分. LTE 切换概述***](https://www.netmanias.com/en/?m=view&id=techdocs&no=6224)

本文档及之后的两篇文档将讨论这样的切换过程：当 UE 在仍通过其接入的 LTE 网络获得服务时，与当前服务小区断开，并在同一跟踪区 (Tracking Area, TA) 内连接到新服务小区（正如我们上一篇文档中定义的第 6 个 EMM 案例）。本文档将提供 LTE 切换的基本概念和相关过程，并定义后续文档将要涵盖的切换类型和范围。

[***无 TAU 的切换 - 第 2 部分. X2 切换***](https://www.netmanias.com/en/?m=view&id=techdocs&no=6257)

本文档将描述在 LTE 内部环境中执行的 X2 切换过程，正如我们技术文档“EMM 场景中的十一个 EMM 案例”中所定义的 EMM 案例 6。首先讨论与 X2 协议上切换相关的特性，然后介绍 X2 切换的详细过程。我们将了解如何在 EPC 不介入的情况下准备并执行 eNB 之间的切换，以及如何在切换中断期间通过两个 eNB 之间的直接隧道转发下行 (DL) 数据包，以实现无缝服务。我们还将了解切换后 EPC 如何参与 EPS 承载路径的切换。最后，我们将考察在 X2 切换过程前后 EPS 实体中的信息元素有何不同。

[***无 TAU 的切换 - 第 3 部分. S1 切换***](https://www.netmanias.com/en/?m=view&id=techdocs&no=6286)

本文档将描述在 LTE 内部环境中执行的 S1 切换过程，正如我们技术文档“EMM 场景中的十一个 EMM 案例”中所定义的 EMM 案例 6。首先讨论与 S1 协议上切换相关的特性，然后介绍 S1 切换的详细过程。我们将了解 EPC (MME) 如何介入 eNB 之间切换的准备，以及如何在切换中断期间通过经过 S-GW 的间接隧道转发下行 (DL) 数据包，以实现无缝服务。我们还将了解切换后 EPC 如何参与 EPS 承载路径的切换。最后，我们将考察在 S1 切换过程前后 EPS 实体中的信息元素有何不同。
