---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/BrainTeaser/BranTeaser_questions.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的题库与结构化的面试指南，完善你的备考。
>
> 👉 **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_brainteaser)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_brainteaser)**

---

# 常见脑筋急转弯题目（List of Common Brain Teaser Questions）

1. 数组游戏（Array Game）
2. 前缀层级（Prefix Hierarchy）
3. 最大字符串（The Largest String）

## 题目与解答（Questions and Answer）

### **数组游戏（Array Game）**
***题目（Questions）：***
```
给定一个整数数组，确定使所有元素相等的移动次数。每次移动选择除 1 个元素之外的所有元素，并将其值加 1。

示例

numbers = [3, 4, 6, 6, 3]
```

***解答（Solutions）：***
```
涉及概念：特设算法（Ad-hoc algorithm）、数组遍历（array traversal）

最优解：

由于我们的目标是使所有元素相同，与其选择除 1 个元素以外的所有元素并将它们加 1，不如选择 1 个元素并将其值减 1，同时保持其余元素不变。在这种条件下（只有被选中的元素被减小），唯一不应被减小的元素是数组的最小元素。其余所有元素都必须被减小，直到它们都等于最小元素。因此，答案可以计算为数组中每个元素与数组最小元素之差的累加和。
```

### **前缀层级（Prefix Hierarchy）**

***题目（Questions）：***
```
给定一个名称列表，计算该列表中有多少个名称以给定的查询字符串为前缀。前缀必须比整个名称字符串少至少 1 个字符。

示例

names = ['jackson', 'jacques', 'jack']

query = ['jack']

完整的查询字符串 'jack' 是 jackson 的前缀，但不是 jacques 或 jack 的前缀。前缀不能包含整个名称字符串，因此 'jack' 不符合条件。

函数描述

在下方编辑器里补全函数 findCompletePrefixes。该函数必须返回一个整数数组，每个整数表示有多少个名称字符串以某个查询字符串为前缀。

findCompletePrefixes 具有以下参数：

    string names[n]:  名称字符串数组

    string query[q]:  查询字符串数组
```

***解答（Solutions）：***
```
涉及概念：特设算法（Ad-hoc algorithm）、数组遍历（array traversal）

最优解：

由于我们的目标是使所有元素相同，与其选择除 1 个元素以外的所有元素并将它们加 1，不如选择 1 个元素并将其值减 1，同时保持其余元素不变。在这种条件下（只有被选中的元素被减小），唯一不应被减小的元素是数组的最小元素。其余所有元素都必须被减小，直到它们都等于最小元素。因此，答案可以计算为数组中每个元素与数组最小元素之差的累加和。
```

## 相关页面
- [[BranTeaser_questions]] —— 脑筋急转弯
- [[commonBehavior]] —— 常见行为面试题
- [[prepare]] —— 通用嵌入式面试准备清单
- [[onSite_prepare]] —— 现场面试准备
- [[topics_prepare]] —— Facebook 面试主题准备

返回索引 [[00-索引]]
