---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/matrix.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

1. 对角线遍历（Diagonal Traverse）


## 实现（Implementation）

### **对角线遍历（Diagonal Traverse）**

***Big O：*** O(n*m) 时间，O(min(n,m)) 空间
```
提示：

```
```c++
class Solution {
public:
    vector<int> findDiagonalOrder(vector<vector<int>>& matrix) {
        int rows = matrix.size();
        if (rows == 0)
            return vector<int>();
        
        int cols = matrix[0].size();
        if (cols == 0)
            return vector<int>();
        
        int dig_size = rows + cols - 1;
        
        vector<int> ret;
        int i = 0, j = 0;
        for (int k = 1; k <= dig_size; k++) {
            vector<int> diag{};
            i = k >= rows ? rows-1 : k-1;
            j = k >= rows ? k-rows : 0;
            while(i >= 0 && j < cols) {
                diag.push_back(matrix[i][j]);
                i--;
                j++;
            }
            if (k%2 == 0)
                reverse(diag.begin(), diag.end());
            ret.insert(ret.end(), diag.begin(), diag.end());
        }
        
        return ret;
    }
};
```

## 相关页面
- [[Array]] —— 数组相关算法题
- [[linked_list]] —— 链表相关算法题
- [[string]] —— 字符串相关算法题
- [[math]] —— 数学类算法题
- [[dataStructure]] —— 数据结构题（LRU 缓存等）

返回索引 [[00-索引]]
