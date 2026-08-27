---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/examples/airControlSystem.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

我不太确定上面这些，但我可能会从上到下进行，并尝试组织尽可能多的信息，不过在本质上细化一个设计时，时间总是有限。

## 参与者（Players） 

1. 天空中的飞机（airplane in the sky）
2. 地面的空中交通管制单元（air traffic control unit on the ground）

### 飞机（Airplane） -
它有许多组件，但这里重要的是两个

- 飞行员（pilot）—— 正在移动飞机或改变其位置的人
- 空中交通管制单元（air traffic control unit）—— 与地面上的空中交通管制系统通信的那个

***飞行员（Pilot） -***
它有几个动作（actions）

- 注册（register）—— 向地面新的空中交通管制发送初始消息

- 注销（deregister）—— 向地面旧的空中交通管制发送最终消息

- 转向（steer）—— 移动飞机（但我猜这与本设计不太相关）

***空中交通管制通信（Air Traffic Control Communication） -***
- 更新其位置 —— 这要么是在循环中持续发生
- 向地面的空中交通管制单元发送消息 —— 这也是周期性地在循环中发生
- 接收来自空中交通管制单元的消息，并将这些消息回传给飞行员 —— 周期性地在循环中发生（消息要么是简单的 ping，要么是某些消息）

### 在地面上（On the ground） -

***控制器（Controller）***

- 在全局队列（global queue）上订阅注册 / 注销 / 紧急消息（urgent messages）
- 接收注册，发送注册确认（registration ack）
- 接收注销，发送重新注册确认
- 作为对注册的响应，生成飞机专属的线程处理对象
（可以讨论线程池、用于维护飞机位置和其他信息的数据库/数据结构）

***工作者（Worker）：***

- 订阅飞机队列上的消息
- 订阅全局队列上的消息
- 响应周期性位置更新，更新位置数据库
- 通知注册消息并退出

***通信（Communication）：***
- 无线电（radio）的后端。实现消息传递（message passing）
- 接收并收集所有消息并分发它们
- 消息要么是注册、注销、紧急，要么是位置更新
- 消息被放入不同的队列，每个工作者线程在注册消息时创建自己的队列并订阅它

***飞机上的控制单元（Control Unit on Plane）***
- 订阅来自控制器的紧急消息
- 周期性发送位置更新
- 紧急消息被定向到飞行员类（Pilot class）
- 从控制器取消订阅

***飞行员（Pilot）***

- 发送注册，作为对确认的响应，触发飞机上的控制单元订阅新的控制器
- 发送注销，作为对确认的响应，触发飞机上的控制单元取消订阅旧的控制器
- 可以讨论消息队列、IPC 的订阅机制等、数据库、到 GUI 的接口（如果算的话）

## 相关页面
- [[systemDesign]] —— 系统设计总览
- [[crossMCUComm]] —— 跨 MCU 通信
- [[memory_management]] —— 内存管理
- [[cacheDesign]] —— 缓存设计示例
- [[embedded_interview_questions]] —— 嵌入式综合面试题

返回索引 [[00-索引]]
