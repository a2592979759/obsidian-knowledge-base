---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/dataStructure.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

1. 插入删除获取随机 O(1)（Insert Delete GetRandom O(1)）
2. LRU 缓存（LRU Cache）
3. 设计添加与搜索单词的数据结构（Design Add and Search Words Data Structure）
4. 数据流的中位数（Find Median from Data Stream）


## 实现（Implementation）

### **数据流的中位数（Find Median from Data Stream）**

***Big O：*** O(log(n)) 时间，O(n) 空间
```
提示：

两个堆。
```
```c++
class MedianFinder {
    priority_queue<int> lo;
    priority_queue<int> hi;
    
public:
    // Adds a number into the data structure.
    void addNum(int num)
    {
        lo.push(num);                                    // Add to max heap

        hi.push(-lo.top());                               // balancing step
        lo.pop();

        if (lo.size() < hi.size()) {                     // maintain size property
            lo.push(-hi.top());
            hi.pop();
        }
    }

    // Returns the median of current data stream
    double findMedian()
    {
        return lo.size() > hi.size() ? lo.top() : ((double) lo.top() - hi.top()) * 0.5;
    }
};
```

## 相关页面
- [[Array]] —— 数组相关算法题
- [[linked_list]] —— 链表相关算法题
- [[string]] —— 字符串相关算法题
- [[math]] —— 数学类算法题
- [[leetcodeOSflavor]] —— 带操作系统风格的 LeetCode 题（含 LRU/LFU 缓存）

返回索引 [[00-索引]]
