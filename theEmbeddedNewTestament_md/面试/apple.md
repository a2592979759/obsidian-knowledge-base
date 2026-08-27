---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Company/Apple/apple.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的嵌入式题库、带 AI 反馈的编码练习以及系统设计指南进行准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)**

---

# Apple 现场面试流程（Apple On-site interview process）

> 相关代码样例（位于 `_assets/`）：
> - `TakeHomeQ1.c`
> - `TakeHomeQ2.c`

```免责声明：所有信息都来自公开的在线资源！```

### 第 1 轮（1st round）
- 招聘经理轮（Hiring Manager round）
- 两名面试官
- 聊简历（resume）- 30 分钟
- 实现 atoi()

### 第 2 轮（2nd round）
- 两名面试官
- 给定一个一维数组（矩阵 matrix）以及圆心坐标和半径，然后画出圆（将圆形区域矩阵中所有位置标记为 1）。
  - 提示：二维数组搜索（Search 2D array）
  - 计算坐标，并与圆心计算距离

### 第 3 轮（3rd round）
- 两名面试官
- GPIO 传感器中断问题：
  - 给定车轮上的 8 个传感器，计算车轮行驶的距离
  - 前进/后退（forward/backward）？-> 8 位模式（bit pattern）

### 第 4 轮（4th round）
- 两名面试官
- 实现环形缓冲区（Ring Buffer）

### 第 5 轮（5th round）
- 两名面试官
- 长时间聊简历
- 怪异问题（WERID QUESTION）：
  - #def HIGH_THRES 100
  - #def DELTA_THRES 0.2
  - 给定 API call_alarm()，check_data(double temp, time_t timestamp)
  - 问题：如果 temp > HIGH_THRES 或 delta_val > DELTA_THRES，则调用 call_alarm()
  - 你有什么顾虑？如果数据读数非常嘈杂（noisy）会怎样？

## 相关页面
- [[apple]] —— Apple
- [[amazon]] —— Amazon
- [[tesla]] —— Tesla
- [[commonBehavior]] —— 常见行为面试题
- [[prepare]] —— 通用嵌入式面试准备清单

返回索引 [[00-索引]]
