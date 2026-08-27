---
tags:
  - 网络
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Network/PTP.md
created: 2026-08-27
---

## 精密时间协议 (Precision Time Protocol, PTP IEEE1588)

精密时间协议 (Precision Time Protocol, PTP) 旨在将局域网 (local area network, LAN) 上的时钟同步到亚微秒级精度。其基本假设是：准确的时间源与正在同步时钟的系统位于同一个局域网内。这不同于 NTP，NTP 假定时间服务器是远程的。通过局域网同步的一个优势是延迟变得可预测得多。不存在因路由器而导致的排队延迟（回想一下，路由器在将完整报文转发给下一个路由器之前必须先存储它），并且物理距离往往小得多。当然，以太网交换机确实会产生一些延迟，并且如果目的地正在接收其他报文，数据包有可能被排队。然而，交换机延迟通常在几微秒量级，如果流量不大，排队甚至可以不存在。

为了进一步最小化延迟和抖动，高精度 PTP 实现会尝试在网络栈的最低层（例如 MAC 层，即以太网收发器）生成时间戳，就在数据包发出之前。

### 最佳主时钟 (Best master clock)
在运行 PTP 的计算机网络中，必须有一个系统被选为主时钟。该系统被认为拥有最准确的时钟，其他系统则是从主时钟同步的从时钟 (slaves)。

最佳主时钟选择过程用于确定哪个系统拥有最适合用于同步的时钟。这是通过一种选举完成的，系统在其中展示各自时钟的信息，然后从以下按优先级排序的属性中选出最佳时钟：

1. 优先级 1 (Priority 1)（管理员定义的提示，允许管理员强制使用某个系统作为主时钟）
2. 时钟等级 (Clock class)（时钟的类型）
3. 时钟精度 (Clock accuracy)
4. 时钟方差 (Clock variance)：基于先前同步的稳定性估计
5. 优先级 2 (Priority 2)（另一个管理员定义的提示，当所有其他值都相同时，允许偏好某个系统）
6. 唯一 ID (Unique ID)（当所有其他值在系统间都相同时，作为平局决胜）

### PTP 报文
1. PTP 通过交换三个报文来同步时钟：
2. 主时钟通过发送一个同步 (sync) 报文来发起同步。
3. 从时钟向主时钟发回一个延迟请求 (delay request) 报文。
4. 主时钟回应一个延迟响应 (delay response) 报文。

延迟请求报文并不完全像它听起来那样：它并不是向主时钟询问网络延迟是多少的查询——主时钟对此毫不知情。相反，它是使从时钟能够获得网络延迟准确估计的额外报文。

![[_assets/ptp.png]]

### **实现亚 100 纳秒同步的网络要求**
在局域网上获得亚 100 纳秒的定时需要一种完全符合 IEEE-1588 的架构。三个主要组件是：GPS 主时钟 (Grandmaster clock)、以太网交换机（透明时钟或边界时钟）以及 PTP 从时钟。所有组件都必须支持硬件时间戳。Grandmaster 和 Slave 在“PTP 实现”一节中讨论。下面讨论所需的以太网交换机。

***以太网交换机***

以太网交换机可以分为标准以太网交换机和支持 IEEE-1588 的以太网交换机。标准以太网交换机在发出数据包前会暂时存储它们。数据包的存储时间是非确定性的，并且取决于网络负载，从而导致数据包延迟变化。数据包延迟变化是在主时钟和从时钟都支持硬件时间戳的情况下，标准以太网交换机仍导致时间同步不佳的主要原因。支持 IEEE-1588 的交换机要么是透明时钟 (transparent clock)，要么是边界时钟 (boundary clock)。使用透明时钟或边界时钟可以改善主从之间的同步，并确保主从不受数据包延迟变化的影响。

***高速、低延迟交换机***

从定时的角度来看，高速低延迟交换机被归类为标准交换机。高速低延迟的存储转发交换机在轻网络负载下可以产生非常稳定和准确的同步；然而，它们仍会存储数据包，从而增加数据包延迟变化，对时间同步产生负面影响。

***透明交换机***

透明交换机是一种与标准交换机相比以不同方式处理 IEEE-1588 报文的以太网交换机。透明时钟测量数据包存储在交换机中的时间，然后将测得的时间加到后续 (follow-up) 报文的校正字段中。为了计入数据包延迟，从时钟使用原始时间戳和校正字段。

***边界时钟***

边界时钟是一种与标准交换机或透明交换机相比以不同方式处理 IEEE-1588 报文的以太网交换机。在安装边界时钟时，网络的子网必须隔离 PTP 报文。边界时钟在网络上的作用很像一个普通时钟，并成为隔离子网上的主时钟。边界时钟只处理 PTP 报文，而标准以太网交换机或路由器处理所有其他网络流量。隔离子网上的从时钟同步到边界时钟，就好像它是主时钟一样。

### 主时钟选择算法：BMS

## 广义精密时间协议 (Generalized Precision Time Protocol, gPTP IEEE 802.1AS)

```IEEE 802.1AS 是 PTP 的一种改编，用于音频视频桥接和时间敏感网络。```

## 参考

https://endruntechnologies.com/pdf/PTP-1588.pdf

[[_assets/PTP-1588.pdf|PTP-1588 论文]]

[PTP 入门视频](https://www.youtube.com/watch?v=kJcmPg-qIFA&ab_channel=alantalkstech)

[PTP 从时钟如何与 PTP 主时钟同步（视频）](https://www.youtube.com/watch?v=Forh3XfD_Ec&ab_channel=DavidGessner)

[基于 IEEE Std 802.1AS 的 AAA2C 时间同步教程](https://avnu.org/wp-content/uploads/2014/05/AVnu-AAA2C_Tutorial-on-time-synchronization-for-AAA2C-based-on-IEEE-Std-802.1AS%E2%84%A2-2011_Kevin-Stanton-5.pdf)

[AVB Wiki](https://en.wikipedia.org/wiki/Audio_Video_Bridging)

[TSN Wiki](https://en.wikipedia.org/wiki/Time-Sensitive_Networking)
