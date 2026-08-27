---
tags:
  - 网络
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/AVB_TSN.md
created: 2026-08-27
---

## 音频/视频桥接与时间敏感网络 (Audio/Video Bridging and Time Sensitive Networks)

### **简介**

[[_assets/2014-11-20_AVnu-Automotive-White-Paper_Final_Approved.pdf|Avu 联盟汽车白皮书]]

这份白皮书概述了使用 AVB 技术进行车载联网的特性与优势。

### **构成 AVB 的基础标准**

![[_assets/AVN_layers.png]]

***IEEE 802.1AS (gPTP)***：桥接局域网中时间敏感应用的定时与同步。

- 这会自动选择一个设备作为主时钟（master clock），然后该时钟将时间分发到整个桥接局域网 / IP 子网内的所有其他节点。
- 802.1AS 时钟不用作媒体时钟。相反，802.1AS 时间被用作节点之间的共享时钟基准，用于将媒体时钟从讲者（talker）传送到听者（listener）。
- 这样的基准消除了修复数据包传递延迟的需要，也无需在存在大量网络抖动的情况下计算长期平均值来估计发送端的实际媒体速率。
- 基于已批准的 IEEE 1588-2008 (PTP) 标准。

[***IEEE 802.1AS 与 IEEE 1588 的 IEEE 幻灯片***](https://www.itu.int/dms_pub/itu-t/oth/06/38/T06380000040002PDFE.pdf)

[***测量 802.1AS 从时钟精度***](https://www.keysight.com/us/en/assets/7019-0404/technical-overviews/Measuring-802-1AS-Slave-Clock-Accuracy.pdf)

***IEEE 802.1Q-2012 (SRP)***：虚拟桥接局域网 - 修正案 9：流预留协议 (Stream Reservation Protocol, SRP)。

- 这允许在桥接局域网 / IP 子网中的讲者与听者之间建立流预留。

***IEEE 802.1Q-2012 (FQTSS)***：虚拟桥接局域网 - 修正案 11：时间敏感流的转发与排队。

- 这描述了一种用于整形网络流量的令牌桶方法，从而可以控制预留流的延迟和带宽。

***IEEE 802.1BA***：“音频/视频桥接 (AVB) 系统”

- 这引用并定义了在构建 AVB 系统时相关的 IEEE 802.1 和其他标准。

***IEEE 1722***：“桥接局域网中时间敏感应用的第二层传输协议。”

- 这规定了用于确保音频与视频终端站之间互操作性的协议、数据封装以及呈现时间过程，这些终端站使用所有 IEEE 802 网络提供的标准网络服务，并满足时间敏感应用的服务质量要求。

[***IEEE 1722 音频视频传输协议 (AVTP) 幻灯片（AVU 联盟）***](https://avnu.org/wp-content/uploads/2014/05/AVnu-AAA2C_Audio-Video-Transport-Protocol-AVTP_Dave-Olsen.pdf)

***IEEE 1733***：“桥接局域网中时间敏感应用的第三层传输协议。”
