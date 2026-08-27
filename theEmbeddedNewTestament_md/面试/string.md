---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/string.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

1. 验证回文串 II（Valid Palindrome II）
2. 验证回文串（Valid Palindrome）
3. 字符串解码（Decode String）
4. 找到字符串中所有字母异位词（Find All Anagrams in a String）

## 实现（Implementation）

### **验证回文串（Valid Palindrome）**

***Big O：*** O(n)，O(1) 空间
```
提示：

双指针。
```
```c++
class Solution {
public:
    bool isPalindrome(string s) {
        int st = 0;
        int end = s.size()-1;
        
        while (st < end) {
            if (!isalnum(s[st]) || !isalnum(s[end])) {
                while(st < end && !isalnum(s[st])) {
                    st ++;
                }
                while(st < end && !isalnum(s[end])) {
                    end --;
                }
            } else {
                if ((isalpha(s[st]) && isalpha(s[end]) && tolower(s[st]) == tolower(s[end])) || (s[st] - '0' == s[end] - '0')) {
                    st ++;
                    end --;
                }  else 
                    return false;
            } 
        }
        
        return true;
    }
};
```
### **验证回文串 II（Valid Palindrome II）**

***Big O：*** O(n)，O(1) 空间
```
提示：

先实现 is_palindrome。将两次 is_palindrome 的结果取或作为答案。
```
```c++
class Solution {

public:
    // Checks if string is a palindrome
    bool isPalin(string &s, int start, int end) {
        while(start < end) {
            if(s[start] != s[end])
                return false;
            ++start, --end;
        }
        return true;
    }
    
    // TC: O(N)
    // SC: O(1)
    bool validPalindrome(string s) {
        for(int i = 0, j = s.size()-1; i < j;  ++i, --j) {
            // mismatch found, only if it is the first time delete
            // a char and move on, else not possible
            if(s[i] != s[j]) {
                // s[0:i-1] and s[j+1, n-1] matched,
                // now we check if atleast s[i:j-1] or s[i+1:j] is a palindrome
                return (isPalin(s, i, j-1) || isPalin(s, i+1, j));
            }
        }
        return true;
    }
};
```

### **字符串解码（Decode String）**

***Big O：***

时间复杂度：O(maxK⋅n)，其中 maxK 是 kk 的最大值，nn 是给定字符串 ss 的长度。我们遍历长度为 nn 的字符串，并迭代 kk 次来解码每个形如 k[string] 的模式。这使最坏情况的时间复杂度为 O(maxK⋅n)。

空间复杂度：O(m+n)，其中 mm 是字符串 ss 中字母（a-z）的个数，nn 是数字（0-9）的个数。在最坏情况下，stringStack 和 countStack 的最大尺寸分别为 mm 和 nn。
```
提示：

使用栈来存储 pair<int, int>（frequency, string）（等价于使用两个栈）。
```
```c++
class Solution {
public:
    string decodeString(string s) {
        stack<pair<int, string>> decode_stk;
        string ret = "";
        
        int number = 0;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == '[') {
                string encode_str = "";
                decode_stk.push(make_pair(number, encode_str));
                number = 0;
            } else if (s[i] == ']') {
                pair<int, string> tmp = decode_stk.top(); 
                decode_stk.pop();
                
                string copy = tmp.second;
                while(--tmp.first) {
                    copy += tmp.second;
                }
                
                if (decode_stk.empty()) {
                    ret += copy;
                } else {
                    tmp = decode_stk.top();
                    decode_stk.pop();
                    tmp.second += copy;
                    decode_stk.push(tmp);
                }
            } else if (isalpha(s[i])) {
                if (decode_stk.empty()) {
                    ret += s[i];
                } else {
                    pair<int, string> tmp = decode_stk.top(); 
                    decode_stk.pop();
                    tmp.second += s[i];
                    decode_stk.push(tmp);
                }
            } else if (isdigit(s[i])) {
                number = number*10 + s[i] - '0';
            }
        }
        
        return ret;
    }
};
```
### **找到字符串中所有字母异位词（Find All Anagrams in a String）**

***Big O：*** O(Ns + Np)，O(1) 空间（不超过 26 个字符）
```
提示：

滑动窗口 + 哈希表（HashMap），或滑动窗口 + 含 26 个元素的数组
```
```c++
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        unordered_map<char, int> p_umap, s_umap;
        vector<int> ret;
        
        if (s.size() < p.size())
            return ret;
        
        for (auto c : p)
            p_umap[c] ++;
        
        int st = 0, end = 0;
        
        while (end < s.size()) {
            //cout << st << end << endl;
            if (end-st+1 < p.size()) {
                if (p_umap.find(s[end]) != p_umap.end()) {
                    s_umap[s[end]] ++;
                    end ++;
                } else {
                    s_umap.clear();
                    end ++;
                    st = end;
                }
            } else if ((end-st+1 > p.size())) {
                if (s_umap[s[st]] == 1)
                    s_umap.erase(s[st]);
                else
                    s_umap[s[st]] --;
                st ++;
            } else {
                s_umap[s[end]] ++;
                if (s_umap == p_umap)
                    ret.push_back(st);
                end ++;
            }
        }
        
        return ret;
    }
};
```

## 相关页面
- [[Array]] —— 数组相关算法题
- [[linked_list]] —— 链表相关算法题
- [[matrix]] —— 矩阵相关算法题
- [[math]] —— 数学类算法题
- [[dataStructure]] —— 数据结构题（LRU 缓存等）

返回索引 [[00-索引]]
