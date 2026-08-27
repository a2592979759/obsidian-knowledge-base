---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/math.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

1. 不用 long long 反转整数（Reverse Integer without using long long）
2. 回文数（Palindrome Number）
3. 两个稀疏向量的点积（Dot Product of Two Sparse Vectors）
4. Pow(x, n)
5. 丑数（Ugly Number）


## 实现（Implementation）
### **两个稀疏向量的点积（Dot Product of Two Sparse Vectors）**

***Big O：*** O(k) 时间（k 为长度较短的数组大小），O(n) 空间
```
提示：

用哈希表（hashmap）存储 <index, value>。计算点积时，遍历更稀疏的那个向量的哈希表并求和即可。
```
```c++
class SparseVector {
public:
    unordered_map<int, int> umap;
    
    SparseVector(vector<int> &nums) {
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != 0)
                umap[i] = nums[i];
        }
    }
    
    // Return the dotProduct of two sparse vectors
    int dotProduct(SparseVector& vec) {
        int sum = 0;
        
        if (vec.umap.size() < this->umap.size()) {
            for (auto it : vec.umap) {
                if (umap.find(it.first) != umap.end())
                    sum += umap[it.first]*it.second;
            }
        } else {
             for (auto it : umap) {
                if (vec.umap.find(it.first) != vec.umap.end())
                    sum += vec.umap[it.first]*it.second;
            }
        }
        
        return sum;
    }
};

```

### **不用 long long 反转整数（Reverse Integer without using long long）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

通过除以边界值 10 来与边界值比较，从而检查溢出。

另一种方法是先把整数转成字符串，然后在字符串上进行比较。
```
```c++
class Solution {
public:
    int reverse(int x) {
        int y=0;
        while(x){
            if(y>INT_MAX/10 || y<INT_MIN/10){
                return 0;
            }else{
                y=y*10 +x%10;
                x=x/10;
            }
        }
        return y;
    }
};
```

## **回文数（Palindrome Number）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

反转整数并与原数比较。相等则返回 true。注意溢出。对反转后的数字变量使用 long long。
```
```c++
class Solution {
public:
    bool isPalindrome(int x) {
        if (x < 0)
            return false;
        
        long long temp = x, dummy = 0;
        
        while (temp) {
            dummy *= 10;
            dummy += temp%10;
            temp /= 10;
        }
        
        return (long long) x == dummy;
    }
};
```

## **Pow(x, n)**

***Big O：*** O(log(n)) 时间，O(1) 空间
```
提示：

快速幂算法（Fast Power Algorithm）迭代版
```
```c++
class Solution {
public:
    double myPow(double x, int n) {
        long long N = n;
        if (N < 0) {
            x = 1 / x;
            N = -N;
        }
        double ans = 1;
        double current_product = x;
        for (long long i = N; i ; i /= 2) {
            if ((i % 2) == 1) {
                ans = ans * current_product;
            }
            current_product = current_product * current_product;
        }
        return ans;
    }
};
```

## **丑数（Ugly Number）**

***Big O：*** O(log(n)) 时间，O(1) 空间
```
提示：

思路很简单；你在 primes 中方便地存放了 3 个质数，遍历它们，逐步缩减最初给定的数字，条件是该质数是否为它的因数。

循环结束时，如果剩下的是 == 1，那么它就是丑数，否则不是（注意这也会排除非正数，不过我倾向于先检查它以省去计算）。
```
```c++
class Solution {
public:
    vector<int> primes = {2, 3, 5};
    bool isUgly(int n) {
        if (n < 1) return false;
        for (int p: primes) while (n % p == 0) n /=p;
        return n == 1;
    }
};
```

## 相关页面
- [[Array]] —— 数组相关算法题
- [[linked_list]] —— 链表相关算法题
- [[string]] —— 字符串相关算法题
- [[matrix]] —— 矩阵相关算法题
- [[dataStructure]] —— 数据结构题（LRU 缓存等）

返回索引 [[00-索引]]
