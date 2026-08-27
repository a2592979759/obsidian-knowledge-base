---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/algorithm_prepare.md"
created: 2026-08-27
---
> ## 🚀 在 EmbeddedInterviewLab 上练习与深度学习
>
> 在浏览器内使用 C/C++ 编辑器并配合 AI 评分评测来解这些题，并浏览按难度排名的题库。
>
> 👉 **[使用 AI 反馈练习编程题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## Facebook 高频 LeetCode 题目

## 数组（Array）
1. [连续子数组和（Continuous Subarray Sum）](#continuous-subarray-sum)          中等
2. [最长连续递增子序列（Longest Continous increasing subsequence）](#longest-continous-increasing-subsequence)      简单
3. [买卖股票的最佳时机（Best time to buy and sell stocks）](#best-time-to-buy-and-sell-stocks)     中等
4. [买卖股票的最佳时机 II（Best time to buy and sell stocks II）]      中等
5. [买卖股票的最佳时机 III（Best Time to Buy and Sell Stock III）]      中等
6. [含手续费买卖股票的最佳时机（Best time to buy and sell stock with transcation fee）](#Best-time-to-buy-and-sell-stock-with-transcation-fee)     中等
7. [插入删除获取随机 O(1)（Insert Delete GetRandom O(1)）](#Insert-Delete-GetRandom-O(1))     中等
8. [除自身以外数组的乘积（Product of Array Except Self）](#product-of-array-except-self)        中等
9. [最大交换（Maximum Swap）](#Maximum-Swap)        中等
10. [两个数组的交集（Intersection of Two Arrays）](#Intersection-of-Two-Arrays)      简单
11. [两个数组的交集 II（Intersection of Two Arrays II）](#Intersection-of-Two-Arrays-II)       简单
13. [矩形重叠（Rectangle Overlap）](#Rectangle-Overlap)       简单
14. [最小覆盖子串（Minimum Window Substring）](#Minimum-Window-Substring)        困难
15. [有效数字（Valid Number）](#Valid-Number)        困难
16. [分发糖果（Candy）](#Candy)       困难
17. [和为 K 的子数组（Subarray Sum Equals K）](#Subarray-Sum-Equals-K)     中等
18. [找第 K 大的元素（Find the Kth largest item）](#Find-the-Kth-largest-item)     中等
19. [统计重复项（Count Duplicates）](#Count-Duplicates)        中等
20. [三数之和（3 Sum）](#3-Ssum)        中等
21. [多数元素（Majority Item）](#Majority-Item)      中等

## 矩阵（Matrix）
1. [对角线遍历（Diagnose Tranverse）](#Diagnose-Tranverse)       中等
2. [有序矩阵中的第 K 小元素（Kth Smallest Element in a Sorted Matrix）](#Kth-Smallest-Element-in-a-Sorted-Matrix)     中等
3. [旋转图像（Rotate Image）](#Rotate-Image)        中等
4. [稀疏矩阵乘法（Sparse Matrix Multiplication）](#Sparse-Matrix-Multiplication)        中等

## 二分查找（Binary Search）
1. [搜索二维矩阵（Search a 2D Matrix）](#Search-a-2D-Matrix)      简单
2. [搜索二维矩阵 II（Search a 2D Matrix II）](#Search-a-2D-Matrix-II)   中等
3. [第一个错误的版本（First Bad Version）](#First-Bad-Version)        中等
4. [旋转排序数组中的最小值（Find Minimum in Rotated Sorted Array）](#Find-Minimum-in-Rotated-Sorted-Array)     中等
5. [两数相除（Divide Two Integers）](#Divide-Two-Integers)      中等
6. [在排序数组中查找元素的第一个和最后一个位置（Find First and Last Position of Element in Sorted Array）](#Find-First-and-Last-Position-of-Element-in-Sorted-Array)      中等

## 链表（Linked List）
1. [反转链表 II（Reverse Linked List II）](#Reverse-Linked-List-II)       中等
2. [合并 K 个有序链表（Merge K sorted List）](Merge-K-sorted-List)      中等
3. [重排链表（Reorder List）](#Reorder-List)     中等
4. [两数相加（Add two Number）](#Add-two-Number)  中等
5. [两数相加 II（Add two number II）](#Add-two-number-II)        中等
6. [链表插入排序（Linked List Insertion Sort）](#Linked-List-Insertion-Sort)       中等
7. [两两交换链表中的节点（Swap Nodes in Pairs）](#Swap-Nodes-in-Pairs)      中等
8. [奇偶链表（Odd Even Linked List）](#Odd-Even-Linked-List)     中等
9. [插入到循环有序链表（Insert into a Cyclic Sorted List）](#Insert-into-a-Cyclic-Sorted-List)     中等
10. [旋转链表（Rotate List）](#Rotate-List)     中等
11. [链表加一（Plus One Linked List）](#Plus-One-Linked-List)        中等
12. [相交链表（Intersection of Two Linked Lists）](#Intersection-of-Two-Linked-Lists)        中等
13. [删除排序链表中的重复元素 II（Remove Duplicates from Sorted List II）](#Remove-Duplicates-from-Sorted-List-II)     中等
14. [删除链表倒数第 N 个节点（Remove Nth Node From End of List）](#Remove-Nth-node-from-the-end-of-list)       中等

## 字符串（String）
1. [有效的字母异位词（Valid Anagram）](#Valid-Anagram)        简单
2. [整数转英文单词（Integer to English Words）](#Integer-to-English-Words)     困难
3. [插入五（Insert five）](#Insert-five)      简单
4. [划分字母区间（Partition Labels）](#Partition-Labels)        中等
5. [最长回文子串（Longest Palindromic Substring）](#Longest-Palindromic-Substring)      中等
6. [单词拆分（Word Break）](#Word-Break)        中等

## 位操作（Bits Manipulation）
1. [是否是 4 的幂（Is Power of Four）](#Is-Power-of-Four)     简单
2. [范围按位与（Range Bit And）](#Range-Bit-And)     中等
3. [数字补数（Number Complement）](#Number-Complement)      中等
4. [只出现一次的数字 II（Single Number II）](#Single-Number-II)        中等

## 数据结构（Data Structure）
1. [LRU 缓存（LRU cache）](#LRU-cache)      困难
2. [LFU 缓存（LFU cache）](#LFU-cache)      困难
3. [最小栈（Min Stack）](#Min-Stack)      简单

## 操作系统相关（OS flavor）
1. [任务调度器（Task Scheduler）](#task-scheduler)       中等
2. [用 Read4 读取 N 个字符（Read N Characters Given Read4）](#Read-N-Characters-Given-Read4)         简单
3. [用 Read4 读取 N 个字符 II（Read N Characters Given Read4 II）](#Read-N-Characters-Given-Read4-II)    困难
4. [函数的独占时间（Exclusive Time of Functions）](#Exclusive-Time-of-Functions)  中等

## 其他高频（Other High Frequency）
1. [接雨水（Trap Rain Water）](#Trap-Rain-Water)      困难
2. [编辑距离为 1（One edit distance）](#One-edit-distance)      中等
3. [Pow(x,n)](#Pow(x,n))        中等
4. [二叉树的序列化与反序列化（Serialize and Deserialize Binary Tree）](#Serialize-and-Deserialize-Binary-Tree)      困难
5. [字母大小写全排列（Letter Case Permutation）](#Letter-Case-Permutation)      简单
6. [Excel 表列名称（Excel Sheet Column Title）](#Excel-Sheet-Column-Title)    简单
7. [有效的括号（Valid Parentheses）](#Valid-Parentheses)      简单
8. [外观数列（Count and Say）](#Count-and-say)      简单
9. [合并区间（Merge Intervals）](#Merge-intervals)      简单
10. [子集（Subsets）](#Subsets)     中等
11. [子集 II（SubsetsII）](#SubsetsII)     中等
12. [验证外星语字典（Verifying a Alien Dictionary）](#Verify-a-Alien-Dictionary)        中等
13. [盛最多水的容器（Container With Most Water）](#Container-With-Most-Water)     中等
14. [移动零（Move Zeroes）](#Move-Zeroes)     中等
15. [对称二叉树（Symmetric Tree）](#Symmetric-Tree)       简单
16. [另一棵树的子树（Subtree of Another Tree）](#Subtree-of-Another-Tree)     中等

## 实现（Implementation）

### **连续子数组和（Continuous Subarray Sum）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

遍历并检查。
```
```c++
class Solution {
public:
    /*
     * @param A: An integer array
     * @return: A list of integers includes the index of the first number and the index of the last number
     */
    vector<int> continuousSubarraySum(vector<int> &A) {
        // write your code here
        int start = 0;
        int end = start;
        int sum = 0;
        int ans = INT_MIN;
        vector<int> ret(2, 0);

        if (A.size() < 2)
            return {0, 0};
        
        for (int i = 0; i < A.size(); i++) {
            if (sum < 0) {
                sum = A[i];
                start = end = i;
            } else {
                sum += A[i];
                end = i;
            }

            if (sum > ans) {
                ret[0] = start;
                ret[1] = end;
                ans = sum;
            }
        }

        return ret;
    }
};
```
### **最长连续递增子序列（Longest Continous increasing subsequence）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

从左到右求出最长递增和最长递减。
```
```c++
class Solution {
public:
    /**
     * @param A an array of Integer
     * @return  an integer
     */
    int longestIncreasingContinuousSubsequence(vector<int>& A) {
        // Write your code here
        int max = 1, s = 1, l = 1;
        int len = A.size();
        if (len == 0)
            return 0;
        for (int i = 1; i < len; ++i) {
            if (A[i] > A[i-1])
                s += 1;
            else {
                if (s > max) max = s;
                s = 1;
            } 
            
            if (A[i] < A[i-1])
                l += 1;
            else {
                if (l > max) max = l;
                l = 1;
            } 
        }
        if (s > max) max = s;
        if (l > max) max = l;
        return max;
    }
};
```
### **买卖股票的最佳时机（Best time to buy and sell stock，仅一次交易）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

记录最低价（min_price），并在遍历过程中计算最大利润。
```
```c++
class Solution {
public:
    /**
     * @param prices: Given an integer array
     * @return: Maximum profit
     */
    int maxProfit(vector<int> &prices) {
        // write your code here
        int min_price = INT_MAX; 
        int profit = 0;

        for (int i = 0; i < prices.size(); i++) {
            min_price = min(prices[i], min_price);
            profit = max(prices[i] - min_price, profit);
        }

        return profit;
    }
};
```


### **买卖股票的最佳时机 II（Best time to buy and sell stock II）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

低买高卖！！！！YOLO WSB!!
```
```c++
class Solution {
public:
    /**
     * @param prices: Given an integer array
     * @return: Maximum profit
     */
    int maxProfit(vector<int> &prices) {
        // write your code here
        int profit = 0;
        int min_price = INT_MAX;

        for (auto p : prices) {
            min_price = min(min_price, p);
            if (p > min_price) {
                profit += (p - min_price);
                min_price = p;
            }
        }

        return profit;
    }
};
```

### **买卖股票的最佳时机 III（Best time to buy and sell stock III）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

```
```c++
class Solution {
public:
    /**
     * @param prices: Given an integer array
     * @return: Maximum profit
     */
    int maxProfit(vector<int> &prices) {
        if (prices.size() == 0) {
            return 0;
        }
        
        vector<int> profit(prices.size());
        int buy = 0;
        buy = prices[0];
        profit[0] = 0;
        for (int i = 1; i < prices.size(); i++) {
            profit[i] = max(profit[i - 1], prices[i] - buy);
            buy = min(buy, prices[i]);
        }
        
        int sell = prices[prices.size() - 1];
        int best = 0;
        for (int i = prices.size() - 2; i >= 0; i--) {
            best = max(best, sell - prices[i] + profit[i]);
            sell = max(sell, prices[i]);
        }
        
        return best;
    }
};
```

### **含手续费买卖股票的最佳时机（Best time to buy and sell stock with transcation fee）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

```
```c++
class Solution {
public:
    /**
     * @param prices: a list of integers
     * @param fee: a integer
     * @return: return a integer
     */
    int maxProfit(vector<int> &prices, int fee) {
        // write your code here
        int cash = 0, hold = -prices[0];

        for (int i = 0; i < prices.size(); i++) {
            cash = max(cash, hold + prices[i] - fee);
            hold = max(hold, cash-prices[i]);
        }
        
        return cash;
    }
};
```
### **对角线遍历（Diagnose Tranverse）**

***复杂度（Big O）：*** O(n*m) 时间，O(max(n, m)) 空间
```
提示（Tips）： 

```
```c++
class Solution {
public:
    /**
     * @param matrix: a 2D array
     * @return: return a list of integers
     */
    vector<int> findDiagonalOrder(vector<vector<int>> &matrix) {
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

### **任务调度器（Task Scheduler）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 
任务种类最多为 26 种。我们分配一个长度为 26 的频率数组 frequencies 来记录每种任务的频率。

遍历输入数组，把任务 A 的频率存在下标 0，任务 B 的频率存在下标 1，以此类推。

找出最大频率：f_max = max(frequencies)。

找出具有最大频率的任务数量：n_max = frequencies.count(f_max)。

如果所使用的槽位数由频率最高的任务决定，则等于 (f_max - 1) * (n + 1) + n_max。

否则，所使用的槽位数由总任务数决定：len(tasks)。

返回这两者的最大值：max(len(tasks), (f_max - 1) * (n + 1) + n_max)。
```
```c++
class Solution {
public:
    int cnt[26], maxcnt = 0, e = 0;
    int leastInterval(vector<char>& tasks, int n) {
        for (char c : tasks) cnt[c-'A']++;
        for (int i = 0; i < 26; i++) maxcnt = max(maxcnt, cnt[i]);
        for (int i = 0; i < 26; i++) 
            if (cnt[i] == maxcnt) e++;
        return max(tasks.size(), (maxcnt-1)*(n+1) + e);
    }
};

// approach 2
class Solution {
public:
    int leastInterval(string &tasks, int n) {
        // write your code here
        unordered_map<char,int>mp;
        int count = 0;
        for(auto e : tasks) {
            mp[e]++;
            count = max(count, mp[e]);
        }
        
        int ans = (count - 1) * (n + 1);
        for(auto e : mp) {
            if(e.second == count) {
                ans ++;
            }
        }
        return max((int)tasks.size(), ans);        
    }
};
```

### **插入删除获取随机 O(1)（Insert Delete GetRandom O(1)）**
**

***复杂度（Big O）：*** O(n) 时间，O(n) 空间
```
提示（Tips）： 

删除时，把目标位置的值与 vector 最后一个元素交换，并更新下标映射。
```
```c++
class RandomizedSet {
public:
    unordered_map<int, int >mp;
    vector<int>vc;
    int n;
    /** Initialize your data structure here. */
    RandomizedSet() {
        srand(time(NULL));
        n = 0;
    }
    
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    bool insert(int val) {
       if (mp.find(val) == mp.end()) {
            if (n < vc.size()) {
                vc[n] = val;
            } else {
                vc.push_back(val);
            }
            mp[val] = n;
            n++;
            return true;
        }
        return false;
    }
    
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    bool remove(int val) {
           
        if (mp.find(val) != mp.end()) {
            vc[mp[val]] = vc[n-1];
            mp[vc[n-1]] = mp[val];
            n--;
            mp.erase(val);
            return true;
        }
        return false;
    }
    
    /** Get a random element from the set. */
    int getRandom() {
        if (n < 1)  
            return 0;
        return vc[rand()%n];
    }
};
```

### **除自身以外数组的乘积（Product of Array Except Self）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

维护一个乘积数组。类似双指针（two pointer）的做法。
```
```c++
vector<int> productExceptSelf(vector<int> &nums) {
    // write your code here
    int n = nums.size();
    vector<int>res = vector<int>(n, 0);
    res[0] = 1;
    for (int i = 1; i < n; i++) {
        res[i] = res[i - 1] * nums[i - 1];
    }
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        res[i] *= right;
        right *= nums[i];
    }
    return res;
}
};
```
### **搜索二维矩阵（Search a 2D Matrix）**

***复杂度（Big O）：*** O(log(m*n)) 时间，O(1) 空间
```
提示（Tips）： 

对所有元素进行二分查找。计算矩阵坐标。
```
```c++
class Solution {
public:
    bool searchMatrix(vector<vector<int>> &matrix, int target) {
        // write your code here
        int row = matrix.size();
        int col = matrix[0].size();

        if (row == 0 && col == 0) {
            return false;
        }

        int st, end;
        st = 0;
        end = row * col-1;
        while (st <= end) {
            int mid = st + (end-st)/2;
            int x = mid/col;
            int y = mid%col;

            if (matrix[x][y] == target) {
                return true;
            } else if (matrix[x][y] < target) {
                st = mid + 1;
            } else {
                end = mid - 1;
            }
        }

        return false;
    }
};
```
### **搜索二维矩阵 II（Search a 2D Matrix II）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

从左下角开始，向右上方搜索。
```
```c++
class Solution {
public:
    int searchMatrix(vector<vector<int>> &matrix, int target) {
        // write your code here
        int occur = 0;
        int row = matrix.size();
        int col = matrix[0].size();
        
        if (row == 0)
            return occur;

        int x = row-1;
        int y = 0;

        while (x >= 0 && y < col) {
            int val = matrix[x][y];

            if (val == target) {
                occur ++;
                x --;
            } else if (val > target) {
                x--;
            } else {
                y ++;
            }
        }

        return occur;
    }
};
```
### **第一个错误的版本（First Bad version）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）： 

从末端进行二分查找。
```
```c++
class Solution {
public:
    int findFirstBadVersion(int n) {
        // write your code here
        int st = 1;
        int end = n;

        while (st < end) {
            int mid = st + (end - st)/2;
            if (SVNRepo::isBadVersion(mid))
                end = mid;
            else {
                st = mid + 1;
            }
        }

        return end;
    }
};
```
### **旋转排序数组中的最小值（Find Minimum in Rotated Sorted Array）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）： 

二分查找（Binary Search）
```

```c++
class Solution {
public:
    int findMin(vector<int> &nums) {
        // write your code here
        int st = 0;
        int end = nums.size()-1;

        if (nums.size() < 1)
            return -1;

        while (st < end) {
            int mid = st + (end - st)/2;

            if (nums[end] > nums[st]) {
                end = mid;
            } else {
                if (nums[mid] < nums[st])
                    end = mid;
                else 
                    st = mid + 1;
            }
        }

        return nums[end];
    }
};
```
### **两数相除（Divide Two Integers）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）： 

二分查找，但要注意溢出和特殊情况。
```
```c++
class Solution {
public:
    int divide(int dividend, int divisor) {
        // write your code here
        if (dividend == INT_MIN && divisor == -1)
            return INT_MAX;
        if (abs(divisor) == 1)
            return divisor == 1 ? dividend : -dividend;

        long dividend_long = dividend;
        long divisor_long = divisor;
        if (dividend_long == 0 || (abs(dividend_long) < abs(divisor_long)))
            return 0;

        int sign = (dividend_long < 0 && divisor_long > 0) || (dividend_long > 0 && divisor_long < 0);
        dividend_long = abs(dividend_long);
        divisor_long = abs(divisor_long);

        int quo = 0;
        int mul = 0;
        while (dividend_long > 0) {
            long divi = divisor_long << mul;
            if (dividend_long >= divi) {
                dividend_long -= divi;
                quo += 1<<mul;
                mul++;
                if (dividend_long < divisor_long)
                    break;
            } else {
                mul = 0;
            }
        }

        return sign ? -quo : quo;
    }
};
```
### **有序矩阵中的第 K 小元素（Kth Smallest Element in a Sorted Matrix）**

***复杂度（Big O）：*** O(n*n) 时间，O(1) 空间
```
提示（Tips）： 

最小堆（min heap）优先队列解法。
```
```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int k) {
        priority_queue<int> mxheap;
        for(auto x:matrix)
        {
            for(auto y:x)
            {
                if(mxheap.size()<k) mxheap.push(y);
                else
                {
                    if(y<mxheap.top())
                    {
                        mxheap.pop();
                        mxheap.push(y);
                    }
                }
            }
        }
        return mxheap.top();
    }
};
```

### **在排序数组中查找元素的第一个和最后一个位置（Find First and Last Position of Element in Sorted Array）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）： 

做两次二分查找，第一次找头部，第二次找尾部。
```
```c++
class Solution {
public:
    vector<int> searchRange(vector<int> &nums, int target) {
        // Write your code here.
        int st = 0;
        int end = nums.size() - 1;

        vector<int> ans = {-1, -1};
        while(st < end) {
            int mid = st + (end-st)/2;
            if (nums[mid] >= target)
                end = mid;
            else
                st = mid + 1;
        }

        if (nums[end] != target)
            return ans;
        
        ans[0] = st;

        st = 0;
        end = nums.size() - 1;
        while(st < end-1) {
            int mid = st + (end-st)/2;
            if (nums[mid] <= target)
                st = mid;
            else
                end = mid;
        }
        ans[1] = nums[end] == target ? end : st;
        
        return ans;
    }
};
```
### **反转链表 II（Reverse Linked List II）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）： 

反转链表（Reverse Linked list）。
```
```c++
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int left, int right) {
        ListNode dummy;
        dummy.next = head;
        
        ListNode *pre_left;
        ListNode *left_tail;
        ListNode *tmp = &dummy, *pre, *cur;
        int count = 0;
        while(tmp) {
            if (count == left - 1) {
                pre_left = tmp;
                left_tail = tmp->next;
                cur = left_tail;
                pre = nullptr;
                tmp = tmp->next;
            } else if (count >= left && count <= right) {
                ListNode *nxt = cur->next;
                cur->next = pre;
                pre = cur;
                cur = nxt;
                if (count == right) {
                    left_tail->next = cur;
                    pre_left->next = pre;
                }
                tmp = cur;
            } else {
                tmp = tmp->next;
            }
            count ++;
        }
        
        return dummy.next;
    }
};
```

### **合并 K 个有序链表（Merge K sorted List）**

***复杂度（Big O）：*** O(nlog(k)) 时间，O(n) 空间
```
提示（Tips）： 

使用优先队列（priority queue）记录队列中最小的元素。弹出它，再把这个链表中的下一个元素入队。每次弹出和入队到优先队列的比较开销降到 O(logk)。但找到最小值的节点只需 O(1) 时间。

方法 2：分治合并（Merge with Divide And Conquer）

时间复杂度：O(Nlogk)，其中 k 是链表个数。

我们可以在 O(n) 时间内合并两个有序链表，其中 n 是两个链表的总节点数。

空间复杂度：O(1)

我们可以在 O(1) 空间内合并两个有序链表。
```
![Merge with DC](https://leetcode.com/problems/merge-k-sorted-lists/Figures/23/23_divide_and_conquer_new.png)
```c++
class Solution {
public:
    struct compare {
        bool operator()(ListNode* A, ListNode* B) {
            return A->val > B->val;
        }
    };

    ListNode *mergeKLists(vector<ListNode *> &lists) {
        // write your code here
        priority_queue<ListNode*, vector<ListNode*>, compare> nodes_heap;
        int n = lists.size();
        ListNode* ans, dummy;
        dummy.next = nullptr;

        for (int i = 0; i < n; i++) {
            if (lists[i] != nullptr)
                nodes_heap.push(lists[i]);
        }

        ans = &dummy;
        while(!nodes_heap.empty()) {
            ListNode* tmp = nodes_heap.top();
            nodes_heap.pop();
            ans->next = tmp;
            ans = ans->next;
            tmp = tmp->next;
            if (tmp != nullptr)
                nodes_heap.push(tmp);
        }

        return dummy.next;
    }
};

// Approach 2 (Merge Lists one by one. O(kn) speed.)
class Solution {
public:
    ListNode* merge2Lists(ListNode* l1, ListNode* l2) {
        if (!l1) return l2;
        if (!l2) return l1;
        ListNode* head = l1->val <= l2->val? l1 : l2;
        head->next = l1->val <= l2->val ? merge2Lists(l1->next, l2) : merge2Lists(l1, l2->next);
        return head;
    }
    
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if (lists.size() == 0) return NULL;
        
        ListNode* head = lists[0];
        
        for (int i = 1; i < lists.size(); i++)
            head = merge2Lists(head, lists[i]);
        
        return head;
    }
};
```

### **丑数（Ugly number）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）： 
迭代并检查，或使用递归。
```
```c++
class Solution {
public:
    /**
     * @param num an integer
     * @return true if num is an ugly number or false
     */
    bool isUgly(int num) {
        if (num <= 0) return false;  
        if (num == 1) return true;  
          
        while (num >= 2 && num % 2 == 0) num /= 2;  
        while (num >= 3 && num % 3 == 0) num /= 3;  
        while (num >= 5 && num % 5 == 0) num /= 5;  
          
        return num == 1;  
    }
};
```

### **旋转图像（Rotate Image）**

***复杂度（Big O）：*** O(M) 时间，O(1) 空间
```
提示（Tips）： 
设 M 为网格中的单元格数量。

时间复杂度：O(M)。我们执行两步：转置矩阵，然后翻转每一行。转置矩阵的开销是 O(M)，因为我们移动每个单元格的值一次。翻转每一行的开销也是 O(M)，因为同样每个单元格的值移动一次。

空间复杂度：O(1)，因为我们没有使用任何额外的数据结构。
```
```c++
class Solution {
public:
    
    void swap(int *a, int *b) {
        int temp = *a;
        *a = *b;
        *b = temp;
    }
    
    void rotate(vector<vector<int>>& matrix) {
        int rows = matrix.size();
        int cols = matrix[0].size();
        
        // transpose 
        for (int x = 0; x < rows; x++) {
            for (int y = x + 1; y < cols; y++) {
                swap(&matrix[x][y], &matrix[y][x]);
            }
        }
        
        // reflect
        for (int x = 0; x < rows; x++) {
            int st = 0, end = cols - 1;
            while (st < end) {
                swap(&matrix[x][st++], &matrix[x][end--]);
            }
        }
    }
};
```

### **有效的井字棋状态（Valid Tic-Tac-Toe State）**

***复杂度（Big O）：*** O(M) 时间，O(1) 空间
```
提示（Tips）： 
我们把 'X' 和 'O' 的数量记为 xCount 和 oCount。

我们还会用一个辅助函数 win(player)，用来检查该玩家是否获胜。它会检查棋盘的 8 条水平、垂直或对角线上是否有三个连续相等的该玩家棋子。
```
```c++
class Solution {
public:
    bool is_winner(vector<string>& board, char let) {
        for (int i = 0; i < 3; i++) {
            if (board[i][0] == let && board[i][1] == let && board[i][2] == let)
                return true;
        }
        
        for (int i = 0; i < 3; i++) {
            if (board[0][i] == let && board[1][i] == let && board[2][i] == let)
                return true;
        }
        
        if (board[0][0] == let && board[1][1] == let && board[2][2] == let)
                return true;
        if (board[0][2] == let && board[1][1] == let && board[2][0] == let)
                return true;
        
        return false;
    }
    
    bool validTicTacToe(vector<string>& board) {
        // 0 for 'X', 1 for 'O'
        int xCount = 0, oCount = 0;
        
        for (auto vec : board) {
            for (auto c : vec) {
                if (c == 'X') xCount ++;
                if (c == 'O') oCount ++;
            }
        }
        
        if (xCount == 0 && oCount == 0) return true;
        if (xCount != oCount && xCount - oCount != 1) return false;
        if (is_winner(board, 'X') && (oCount != xCount-1)) return false;
        if (is_winner(board, 'O') && (xCount != oCount)) return false;
        return true;
    }
};
```

### **最大交换（Maximum Swap）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：（贪心）

直觉

对输入数字的每一位按顺序考察：如果后面出现了更大的数字，我们知道最佳交换一定发生在当前正在考察的这一位上。

算法

我们计算 last[d] = i，即数字 d 最后一次出现的下标 i（如果存在）。

之后，从左到右扫描这个数字，如果后面有更大的数字，我们就把它与最大的这类数字交换；如果有多个这样的数字，则与出现最晚的那个交换。
```
```c++
class Solution {
public:
    /**
     * @param num: a non-negative intege
     * @return: the maximum valued number
     */
    int maximumSwap(int num) {
        vector<int> pos(10, -1);
        string number = to_string(num);
        int n = number.size();
        for (int i = 0; i < n; i++) {
            pos[number[i] - '0'] = i;
        }
        
        for (int i = 0; i < n; i++) {
            for (char j = '9'; j > number[i]; j--) {
                if (pos[j - '0'] > i) {
                    swap(number[i], number[pos[j - '0']]);
                    return stoi(number);
                }
            }
        }
        
        return num;
    }
};

class Solution {
public:
    int maximumSwap(int num) {
        string num_str = to_string(num);
        int n = num_str.size();
        int max_val = -1, max_i = -1;
        int left = -1, right = -1;
        
        for (int i = n - 1; i >= 0; i--) {
            if (num_str[i] > max_val) {
                max_val = num_str[i];
                max_i = i;
            } else if (num_str[i] < max_val) {
                left = i;
                right = max_i;
            }
        }
       
        if (left == -1) return num;
        char c = num_str[left];
        num_str[left] = num_str[right];
        num_str[right] = c;
        return stoi(num_str);
    }
};
```

### **两个数组的交集（Intersection of Two Arrays）**

***复杂度（Big O）：*** O(nlog(n) + mlog(m)) 时间，O(1) 空间
```
提示（Tips）： 
先排序 O(nlogn)，然后遍历。
或者使用两个哈希集合（hash set）。
```
```c++
class Solution {
public:
    vector<int> intersection(vector<int> &nums1, vector<int> &nums2) {
        // write your code here
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());
        vector<int> ans;

        int i = 0, j = 0;
        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] < nums2[j]) {
                i++;
            } else if (nums1[i] > nums2[j]) {
                j++;
            } else {
                ans.push_back(nums1[i]);
                i++;
                j++;
            }
        }

        return ans;
    }
};
```

### **稀疏矩阵乘法（Sparse Matrix Multiplication）**

***复杂度（Big O）：*** O(k*n^2) 时间，O(1) 空间
```
提示（Tips）： 
先用哈希（hash）记录非零元素，只对这些非零项做运算。
```
```c++
class Solution {
public:
    vector<vector<int>> multiply(vector<vector<int>> &A, vector<vector<int>> &B) {
        // write your code here        
        vector<vector<int>> ans(A.size(), vector<int>(B[0].size(), 0));
        
        for (int i = 0; i < A.size(); i++) {
            unordered_map<int, int> umap;
            for (int j = 0; j < A[0].size(); j++) {
                umap[j] = A[i][j];
            }

            for (int k = 0; k < B[0].size(); k++) {
                int sum = 0;
                for (auto item : umap) {
                    if (B[item.first][k] != 0)
                        sum += item.second * B[item.first][k];
                }
                ans[i][k] = sum;
            }
        }

        return ans;
    }
};
```
### **矩形重叠（Rectangle Overlap）**

***复杂度（Big O）：*** O(1) 时间，O(1) 空间
```
直觉

如果两个矩形重叠，它们就有正面积。这个面积一定是一个两个维度都为正的矩形，因为交集的边界是轴向对齐的。

因此，我们可以把问题简化为判断两条线段是否重叠的一维问题。

算法

设交集面积为 width * height，其中 width 是矩形在 x 轴上的投影交集，height 是 y 轴上的投影交集。我们希望两者都为正。

当 min(rec1[2], rec2[2]) > max(rec1[0], rec2[0]) 时，即（最大的 x 坐标中较小者）大于（最小的 x 坐标中较大者）时，width 为正。height 同理。
```

```c++
class Solution {
public:
    /**
     * @param l1: top-left coordinate of first rectangle
     * @param r1: bottom-right coordinate of first rectangle
     * @param l2: top-left coordinate of second rectangle
     * @param r2: bottom-right coordinate of second rectangle
     * @return: true if they are overlap or false
     */
    bool doOverlap(Point &l1, Point &r1, Point &l2, Point &r2) {
        return (min(r1.x, r2.x) > max(l1.x, l2.x)) && (min(l1.y, l2.y) > max(r1.y, r2.y));
    }
};
```

### **最小覆盖子串（Minimum Window Substring）**

***复杂度（Big O）：*** O(K + T) 时间，O(1) 空间，K 为 s 的大小，T 为 t 的大小
```
提示（Tips）：

算法

我们用两个指针，left 和 right 初始都指向字符串 S 的第一个元素。

我们用 right 指针向右扩展窗口，直到得到一个想要的窗口，即一个包含 T 中所有字符的窗口。

一旦得到包含所有字符的窗口，就可以让 left 指针一步步向前移动。如果窗口仍然是想要的，我们就持续更新最小窗口大小。

如果窗口不再满足条件，就重复步骤 2 及之后的操作。
```
```c++

class Solution {

 public:
  static string minWindow(const string &str, const string &pattern) {
  // TODO: Write your code here
    string ret_str = "";
    unsigned int min_str = -1, win_s, win_e, match = 0;
    unordered_map <char, int> umap;

    for (auto chr : pattern)
      umap[chr] ++;

    for (win_e = win_s = 0; win_e < str.size(); win_e ++) {
      if (umap.find(str[win_e]) != umap.end()) {
        if (--umap[str[win_e]] == 0) {
          match ++;
        }
      }
     
      while (match == umap.size()) {
        if ((win_e - win_s + 1) < min_str) {
          min_str = min(min_str, win_e - win_s + 1);
          ret_str = str.substr(win_s, win_e - win_s + 1);
        }
        
        if (umap.find(str[win_s]) != umap.end() && ++umap[str[win_s]] > 0) {
          match --;
        }

        win_s++;     
      }
    }

    return ret_str;
  }
};
```
### **有效数字（Valid Number）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

分治。以 e/E 符号为中心分成两部分，分别检查两边。
```
```c++
class Solution {
public:
	bool ojbk(string s, int k){
		for(int i=0;i<s.size();i++){
			if((s[i]=='+' || s[i]=='-') && i!=0) return false;
		}
		if(s[0]=='+' || s[0]=='-') s.erase(s.begin());
		int countDot=0;
		for(int i=0;i<s.size();i++){
			if(s[i]=='.') countDot++;
			else if(!isdigit(s[i])) return false;
		}
		if(s=="" || s==".") return false;

		return countDot<=k;
	}
	bool isNumber(string s) {
		while(s.size()>0 && s.back()==' ') s.pop_back();
		while(s.size()>0 && s[0]==' ') s.erase(s.begin());
		int countE=0;
		int pos;
		for(int i=0;i<s.size();i++){
			if(s[i]=='e'){
				countE++;
				pos=i;
			}
		}
		if(countE>=2) return false;
		if(countE==0) return ojbk(s,1);
		return ojbk(s.substr(0,pos),1) && ojbk(s.substr(pos+1),0);
	}
};
```

### **有效的字母异位词（Valid Anagram）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

哈希映射（Hash map）。
```
```c++
class Solution {
public:
    bool isAnagram(string s, string t) {
        int umap[26] = {};
        
        if (s.size() != t.size())
            return false;
    
        for (auto &c : s)
            umap[c-'a'] ++;
        
        int count = 0;
        for (auto &c : t) {
            umap[c-'a'] --;
            if (umap[c-'a'] < 0)
                return false;
        }
        
        return true;
    }
};
```

### **整数转英文单词（Integer to English Words）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

分治。分成千位逐段处理，逐个攻克。
```
```c++
class Solution {
public:
    string thousandToWords(int num) {
        string ret = "";
        if (num >= 100) {
            ret += digits[num/100] + "Hundred ";
            num %= 100;
        }
        if (num == 0)  return ret;
        
        ret += (num < 20) ? digits[num] : tens[num/10] + digits[num%10];
        return ret;
    }
    
    string numberToWords(int num) {
        string ans = "";
        
        if (num == 0)
            return "Zero";
        
        for (int i = 0; i < 4; i++) {
            if (num >= units[i].first) {
                ans += thousandToWords(num/units[i].first) + units[i].second;
                num %= units[i].first;
            }
        }
        ans += thousandToWords(num);
        if (ans.size() > 0)
            ans.pop_back();
       
        return ans;
    }
    
private:
    string digits[20] = {"", "One ", "Two ", "Three ", "Four ", "Five ", "Six ", "Seven ", "Eight ", "Nine ", "Ten ", "Eleven ", "Twelve ", "Thirteen ", "Fourteen ", "Fifteen ", "Sixteen ", "Seventeen ", "Eighteen ", "Nineteen "};
    
    string tens[10] = {"", "Ten ", "Twenty ", "Thirty ", "Forty ", "Fifty ", "Sixty ", "Seventy ", "Eighty ", "Ninety "};

    pair<int, string> units[4] = { {1000*1000*1000, "Billion "}, {1000*1000, "Million "}, {1000, "Thousand "}, {100, "Hundred "}};
};
```
### **重排链表（Reorder list）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

找中点节点，反转，再合并。
```
```c++
class Solution {
public:
    ListNode *reverseNode (ListNode* head) {
        ListNode *pre, *cur;
        
        pre = nullptr;
        cur = head;
        while (cur) {
            ListNode *nxt = cur->next;
            cur->next = pre;
            pre = cur;
            cur = nxt;
        }
        
        return pre;
    }
    
    void reorderList(ListNode* head) {
        if (head == nullptr || head->next == nullptr)
            return;
        
        ListNode *slow, *fast;
        slow = fast = head;
        
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next; 
        }
        ListNode *tail = reverseNode(slow);
        ListNode *first = head;
        
        while(tail->next != nullptr) {
            ListNode *tmp = first->next;
            first->next = tail;
            first = tmp;
            
            tmp = tail->next;
            tail->next = first;
            tail = tmp;
        }
        
        return;
    }
};
```

### **两数相加（Add two number）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

进位行波加法器（Carry ripple adder）
```
```c++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int carry = 0;
        ListNode dummy, *tmp;
        
        tmp = &dummy;
        while (l2 || l1 || carry) {
            int sum = 0;
            
            if (l1 && l2) 
                sum = (carry + l1->val + l2->val);
            else if (l1 != nullptr || l2 != nullptr)
                sum = (carry + ((l1) ? l1->val : l2->val));
            else 
                sum = carry;
            
            carry = sum/10;
            
            ListNode *new_node = new ListNode(sum%10, nullptr);
            tmp -> next = new_node;
            tmp = tmp->next;
            l1 = l1 ? l1->next : nullptr;
            l2 = l2 ? l2->next : nullptr;
        }
        
        return dummy.next;
    }
};
```

### **两数相加 II（Add two number II）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

方法 1：

反转每个链表，然后做两数相加。

方法 2：

把数转成字符串，然后从最后一位开始计算。

或者用两个栈（stack）。
```
```c++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        string a, b;
        ListNode *result = nullptr;
        while(l1) { a.push_back(l1->val+'0'); l1 = l1->next;}
        while(l2) { b.push_back(l2->val+'0'); l2 = l2->next;}
        int l = a.size()-1, r = b.size()-1, carry = 0;
        while(l >= 0 || r >= 0 || carry == 1) {
            int c = (l >= 0 ? a[l--]-'0' : 0) + ( r >= 0 ? b[r--]-'0' : 0) + carry;
            ListNode *temp = new ListNode(c%10);
            temp->next = result;
            result = temp;
            carry = c/10;
        }        
        return result;
    }
};
```

### **链表插入排序（Linked List Insertion Sort）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

创建一个哨兵节点（sentinel node）来维护一个独立的有序链表。比较并将节点插入到那个独立的链表中。

```
```c++
class Solution {
public:
    ListNode * insertionSortList(ListNode * head) {
        // write your code here
        ListNode sentinal, *tmp, *tmp_s;
        sentinal.next = nullptr;

        tmp = head;
        while (tmp) {
            ListNode* next = tmp->next;
            tmp_s = &sentinal;

            while (tmp_s->next != nullptr && tmp_s->next->val < tmp->val) {
                tmp_s = tmp_s->next;
            }

            tmp->next = tmp_s->next;
            tmp_s->next = tmp;
            tmp = next;
        }

        return sentinal.next;
    }
};
```

### **两两交换链表中的节点（Swap nodes in pair）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

我们在遍历过程中交换节点。交换一对节点（例如 A 和 B）之后，需要把节点 B 链接到 A 之前的那个节点。为了建立这个链接，我们在 prevNode 中保存 A 的前一个节点。

```
```c++
class Solution {
public:
    ListNode * swapPairs(ListNode * head) {
        // write your code here
        if (!head || !head->next)
            return head;
        
        ListNode *l1, *l2, *pre, sentinal;
    
        l1 = head;
        pre = &sentinal;

        while (l1 && l1->next) {
            l2 = l1->next;
            ListNode *l2_next = l2->next;
            l1->next = l2_next;
            l2->next = l1;
            pre->next = l2;
            pre = l1;
            l1 = l2_next;
        }        

        return sentinal.next;
    }
};
```
### **奇偶链表（Odd Even Linked List）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

先构造偶数链表和奇数链表。然后把偶数链表接到奇数链表的尾部。
```
```c++
class Solution {
public:
    ListNode * oddEvenList(ListNode * head) {
        // write your code here
        if (!head || !head->next)
            return head;

        ListNode odd, even, *iterator, *odd_it, *even_it;
        iterator = head;
        odd_it = &odd;
        even_it = &even;
        
        int count = 1;
        while (iterator) {
            if (count % 2 != 0) {
                odd_it->next = iterator;
                odd_it = odd_it->next;
            }
            else {
                even_it->next = iterator;
                even_it = even_it->next;
            }
            count ++;
            iterator = iterator->next;
        }
        even_it->next = nullptr;
        odd_it->next = even.next;

        return odd.next;
    }
};
```

### **插入到循环有序链表（Insert into a Cyclic Sorted List）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

对单链表使用 pre 和 cur 节点。注意边界情况。
```
```c++
class Solution {
public:
    ListNode * insert(ListNode * head, int x) {
        // write your code here
        if (!head) {
            ListNode *new_node = new ListNode(x, nullptr);
            new_node->next = new_node;
            return new_node;
        }

        ListNode *pre = nullptr, *cur = head;
        do {
            pre = cur;
            cur = cur->next;

            if (pre->val <= x && cur->val >= x)
                break;
            if (pre->val > cur->val && (x > pre->val || x < cur->val))
                break;
        } while (cur != head);

        ListNode *new_node = new ListNode(x, cur);
        pre->next = new_node;

        return head;
    }
};
```
### **旋转链表（Rotate List）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

统计节点数，并计算有效的移动步数。然后从最后一个节点开始旋转。注意 step = 1 的情况。
```
```c++
class Solution {
public:
    ListNode * rotateRight(ListNode * head, int k) {
        // write your code here
        if (!head || !head->next)
            return head;

        int nodes_num = 1;
        ListNode* iterator = head;
        while (iterator->next) {
            nodes_num ++;
            iterator = iterator->next;
        }
        int effective_move = k%nodes_num;
        if (effective_move == 0) return head;

        iterator->next = head;
        int step = nodes_num - effective_move;
        while (step--) {
            iterator = iterator->next;
        }

        ListNode *new_head = iterator->next;
        iterator->next = nullptr;

        return new_head;
    }
};
```
### **相交链表（Intersection of Two Linked Lists）**

***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

```
```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode *A, *B;
        int lenA{}, lenB{}, diff{};
        
        if (!headA || !headB)
            return nullptr;
        
        A = headA;
        B = headB;
        
        while(A) {
            lenA++;
            A = A->next;
        }
        
        while(B) {
            lenB++;
            B = B->next;
        }
        
        diff = abs(lenA-lenB);
        ListNode *tmp = lenA > lenB ? headA : headB;
        
        while(diff--) tmp = tmp->next;
        
        headA = lenA > lenB ? tmp : headA;
        headB = lenB > lenA ? tmp : headB;
        
        while(headA && headB) {
            if (headA == headB) return headA;
            headA = headA->next;
            headB = headB->next;
        }
        
        return nullptr;
    }
};
```
### **LRU 缓存（LRU cache）**

***复杂度（Big O）：*** O(1) 时间，O(n) 空间
```
提示（Tips）：

双向链表 + 哈希映射（hashmap）

用 stl 的 list 存储数据，用哈希映射记录节点的位置。
```
```c++
#include <list>

class LRUCache {
public:
    struct LRUNode {
        int value;
        int key;
        
        LRUNode(int k, int v) : key(k), value(v) {}
    };
    
    LRUCache(int capacity) : cap(capacity) {}

    int get(int key) {
        if (!kv.count(key)) {
            return -1;
        }
        move_front(key);
        return values.front().value;
    }

    void set(int key, int value) {
        if (!kv.count(key)) {
            values.emplace_front(key, value);
            kv[key] = values.begin();
            if (values.size() > cap) {
                kv.erase(values.back().key);
                values.pop_back();
            }
        } else {
            kv[key]->value = value;
            move_front(key);
        }
    }
    
private:
    int move_front(int key) {
        LRUNode node = *kv[key];
        values.erase(kv[key]);
        values.push_front(node);
        kv[key] = values.begin();
    }
    
private:
    int cap;
    std::list<LRUNode> values;
    std::unordered_map<int, list<LRUNode>::iterator> kv;
};
```

### **链表加一（Plus One Linked list）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

跟踪最后一个非 9 的位置。如果非 9 的位置从未被进位，就创建新节点。然后再做一次遍历，把所有 >= 非 9 位置的位加 1 并对 10 取模。
```
```c++
class Solution {
public:
    ListNode* plusOne(ListNode* head) {
        int posNoneNine = 0;
        
        ListNode *iterator = head, sentinal;
        sentinal.next = head;
        int counter = 0;
        while (iterator) {
            counter ++;
            if (iterator->val != 9) {
                posNoneNine = counter;
            }
            iterator = iterator->next;
        }
        
        if (posNoneNine == 0) {
            ListNode *new_node = new ListNode(0, head);
            sentinal.next = new_node;
        }
        
        iterator = sentinal.next;
        counter = 0;
        while (iterator) {
            counter ++;
            if (counter >= posNoneNine)
                iterator->val = (iterator->val + 1)%10;
            iterator = iterator->next;
        }
        
        return sentinal.next;
    }
};
```

### **是否是 4 的幂（Is Power of four）**

***复杂度（Big O）：*** O(1) 时间，O(1) 空间
```
提示（Tips）：

如果一个数是 2 的幂，且置位位（set bit）在奇数位位置，那么它就是 4 的幂。
```
```c++
class Solution {
public:
    /**
     * @param num: an integer
     * @return: whether the integer is a power of 4
     */
    bool isPowerOfFour(int num) {
        // Write your code here
        if (num > 0 && ((num & (num-1)) == 0) && (0x55555555 & num))
            return true;
        return false;
    }
};
```

### **范围按位与（Range Bit And）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

本题中，我们需要得到[m,n]所有元素按位与的结果。举个例子，当m=26，n=30时，它们的二进制表示为为：
11010　　11011　　11100　　11101　　11110
这个样例的答案是11000，易见我们发现我们只需要找到m和n最左边的公共部分即可。

每次都将n与n-1按位与，当n的二进制为1010时，1010 & 1001 = 1000，相当于把二进制位的最后一个1去掉了。因此我们不断的做n^n-1的操作，直到n小于m相等即可。
```
```c++
public class Solution {
    /**
     * @param m: an Integer
     * @param n: an Integer
     * @return: the bitwise AND of all numbers in [m,n]
     */
    public int rangeBitwiseAnd(int m, int n) {
        // Write your code here
        while(m < n) {
            n = n & (n-1);
        }
        return n;
    }
}
```

### **子数组异或查询（Bitwise XOR in a subarray）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：


```
```c++
class Solution {
public:
    vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
        vector<int> res;
        
        for (int i = 1; i < arr.size(); i++) {
            arr[i] ^= arr[i-1];
        }
        
        for (auto &v : queries) {
            if (v[0] == 0)
                res.push_back(arr[v[1]]);
            else {
                res.push_back(arr[v[1]]^arr[v[0]-1]);
            }
            
        }
        
        return res;
    }
}
```

### **分发糖果（Candy）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

先按正常顺序遍历数组，再按相反顺序遍历，过程中更新糖果数组。最后把糖果数组求和得到结果。
```
```c++
class Solution {
public:
    int candy(vector<int>& ratings) {
        int n = ratings.size();
        
        if (n < 2)
            return n;
        
        vector<int>candy(n,1);
        
        for (int i = n-2; i >= 0; i--) {
            if (ratings[i] > ratings[i+1])
                candy[i] = candy[i+1]+1;
        }
        
        for (int i = 1; i < n; i++) {
            if (ratings[i] > ratings[i-1] && candy[i] <= candy[i-1])
                candy[i] = candy[i-1]+1;
        }
        
        int res = 0;
        for (int i = 0; i < n; i++) {
            res += candy[i];
        }
        
        return res;
    }
};
```

### **范围按位与（Candy / Range Bit And）**

***复杂度（Big O）：*** O(1) 时间，O(1) 空间
```
提示（Tips）：

只需要统计 n 和 m 的前缀部分。
```
```c++
class Solution {
public:
    int rangeBitwiseAnd(int m, int n) {
        while (m < n) {
            n &= n - 1;
        }
        
        return m & n;
    }
};
```

### **用 Read4 读取 N 个字符（Read N Characters Given Read4）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）：

用一个临时字符串（temporary string）存储 read4 的结果。注意 read4 返回少于 4 个字节的情况。
```
```c++
class Solution {
public:
 
    int read(char *buf, int n) {
        int read_bytes = 0, read4_bytes = 4;
        char tmp_buf[4];
        
        while (read_bytes < n && read4_bytes == 4) {
            read4_bytes = read4(tmp_buf);
            
            for (int i = 0; i < read4_bytes; i++) {
                if (read_bytes >= n)
                    return read_bytes;
                *buf++ = tmp_buf[i];
                read_bytes ++;
            }
        }
        
        return read_bytes;
    }
};
```

### **用 Read4 读取 N 个字符 II（多次调用，Read N Characters Given Read4 II）**

***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）：

使用私有数据变量记录上一次调用遗留的位置。
```
```c++
class Solution {
    int pos = 0, read4_bytes = 0;
    char tmp_buf[4];

public:
    int read(char *buf, int n) {
        int read_bytes = 0;
        
        // if there is leftover
        while (pos < read4_bytes && read_bytes < n) {
            *buf++ = tmp_buf[pos++];
            read_bytes ++;
        }
        
        while (read_bytes < n) {
            read4_bytes = read4(tmp_buf);

            for (pos = 0; pos < read4_bytes;) {
                if (read_bytes >= n)
                    return read_bytes;

                *buf++ = tmp_buf[pos++];
                read_bytes ++;
            }
                
            if (read4_bytes != 4)
                break;
        }
        
        return read_bytes;
    }
};
```

### **函数的独占时间（Exclusive Time of Functions）**

***复杂度（Big O）：*** O(n) 时间，O(n) 空间
```
提示（Tips）：

使用栈（stack）。更新时间时，记得减去栈顶元素中重复的时间部分。
```
```c++
class Solution {
    struct Log {
        int id;
        bool isStart;
        int time;
    };
    
    Log getLog(string& s) {
        string id, isStart, time;
        stringstream ss(s);
        getline(ss, id, ':');
        getline(ss, isStart, ':');
        getline(ss, time, ':');

        return {stoi(id), isStart == "start", stoi(time)};
    }
    
public:
    vector<int> exclusiveTime(int n, vector<string>& logs) {
        vector<int> exclusive(n, 0);
        stack<Log> s;
        
        for(auto& log: logs) {
            Log l = getLog(log);
            if(l.isStart)
                s.push(l);
            else {
                int time = l.time - s.top().time + 1;
                exclusive[l.id] += time;
                
                s.pop();
                if(!s.empty())
                    exclusive[s.top().id] -= time;
            }
        }
        
        return exclusive;
    }
};
```
### **插入五（Insert Five）**

***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

把整数转成字符串，然后在字符串上操作。
```
```c++
class Solution {
public:
    /**
     * @param a: A number
     * @return: Returns the maximum number after insertion
     */
    int InsertFive(int a) {
        // write your code here
        int sign = a < 0;
        string num = to_string(abs(a));
        int n = num.size();

        int i;
        for (i = 0; i < n; i++) {
            if (sign) {
                if (num[i] - '0' > 5) {
                    num.insert(i, "5");
                    break;
                }
            } else {
                if (num[i] - '0' < 5) {
                    num.insert(i, "5");
                    break;
                }
            }
        }
        if (i == n)
            num += "5";

        return sign ? -(stoi(num)) : stoi(num);
    }
};
```

### Excel 表列名称（Excel Sheet Column Title）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

注意下标
```
```c++
class Solution {
public:
    string convertToTitle(int n) {
        // write your code here
        string res = "";

        while(n) {
            int rem = (n-1)%26;
            char c = rem + 'A';
            res = c + res;
            n = (n-1)/26;
        }

        return res;
    }
};
```
### LFU 缓存（LFU cache）
![LFU cache design](https://assets.leetcode.com/users/images/4cb3255c-f77a-495d-a3ff-804583a7d5b8_1605533049.7540417.png)
```c++
//Just for better readability
typedef int Key_t;
typedef int Count_t;

struct Node
{
    int value;
    list<Key_t>::iterator itr;
};

class LFUCache
{
    unordered_map<Key_t, Node> m_values;
    unordered_map<Key_t, Count_t> m_counts;
    unordered_map<Count_t, list<Key_t>> m_countKeyMap;
    int m_lowestFrequency;
    int m_maxCapacity;

public:
    LFUCache(int capacity)
    {
        m_maxCapacity = capacity;
        m_lowestFrequency = 0;
    }

    int get(int key)
    {
        if (m_values.find(key) == m_values.end() || m_maxCapacity <= 0)
        {
            return -1;
        }
        //update frequency, & return value
        put(key, m_values[key].value);
        return m_values[key].value;
    }

    void put(int key, int value)
    {
        if (m_maxCapacity <= 0)
        {
            return;
        }

        //If key is not present and capacity has exceeded,
        //then remove the key entry with least frequency
        //else just make the new key entry
        if (m_values.find(key) == m_values.end())
        {
            if (m_values.size() == m_maxCapacity)
            {
                int keyToDelete = m_countKeyMap[m_lowestFrequency].back(); 
                m_countKeyMap[m_lowestFrequency].pop_back();
                if (m_countKeyMap[m_lowestFrequency].empty())
                {
                    m_countKeyMap.erase(m_lowestFrequency);
                }
                m_values.erase(keyToDelete);
                m_counts.erase(keyToDelete);
            }
            m_values[key].value = value;
            m_counts[key] = 0;
            m_lowestFrequency = 0;
            m_countKeyMap[m_counts[key]].push_front(key);
            m_values[key].itr = m_countKeyMap[0].begin();
        }
        //Just update value and frequency
        else
        {
            m_countKeyMap[m_counts[key]].erase(m_values[key].itr);
            if (m_countKeyMap[m_counts[key]].empty())
            {
                if (m_lowestFrequency == m_counts[key])
                    m_lowestFrequency++;
                m_countKeyMap.erase(m_counts[key]);
            }
            m_values[key].value = value;
            m_counts[key]++;
            m_countKeyMap[m_counts[key]].push_front(key);
            m_values[key].itr = m_countKeyMap[m_counts[key]].begin();
        }
    }
};
```

### 接雨水（Trap Rain Water）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

双指针（Two pointer）
```
```c++
class Solution {
public:
    int trap(vector<int>& height)
    {
        int ans = 0;
        int left = 0, right = height.size()-1;
        int left_max, right_max;
        left_max = right_max = 0;
        
        while (left < right) {
            right_max = max(right_max, height[right]);
            
            if (height[left] < height[right]) {
                if (left_max > height[left]) {
                    ans += (left_max - height[left]);
                } else {
                    left_max = height[left];
                }
                left ++; 
            } else {
                if (right_max > height[right]) {
                    ans += (right_max - height[right]);
                } else {
                    right_max = height[right];
                }
                right --; 
            }
        }
        
        return ans;
    }
};
```

### 编辑距离为 1（One edit distance）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

若两个字符串长度相差大于1或相等，直接返回false。
反之，顺序判断每一位是否相等，若不相等，执行修改操作。
最后判断一下即可。
```
```c++
class Solution {
public:
    /**
     * @param s a string
     * @param t a string
     * @return true if they are both one edit distance apart or false
     */
    bool isOneEditDistance(string& s, string& t) {
        // Write your code here
        int m = s.size();
        int n = t.size();
        if (m > n)
            return isOneEditDistance(t, s);
        if (n - m > 1 || s == t)
            return false;
        for (int i = 0; i < m; i++) {
            if (s[i] != t[i]) {
                if (m == n) {
                    s[i] = t[i];
                } else {
                    s.insert(i, 1, t[i]);
                }
                break;
            }
        }
        return s == t || s + t[n - 1] == t;
    }
};
```

### Pow(x,n)
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

注意 n 可能是负数, 需要求一下倒数, 可以在一开始把x转换成倒数, 也可以到最后再把结果转换为倒数.

那么现在我们实现 pow(x, n), n 是正整数的版本:

二分即可: 有 x^n = x{n/2} * x{n/2}x
​n
​​ =xn/2∗xn/2, 因此可以在 O(logn) 的时间复杂度内实现.

又叫快速幂. 有递归实现和循环实现的版本.
```
```c++
// Loop Version
class Solution {
private:
    double _myPow(double x, long long n) {
        double res = 1.0;
        for (double i = x; n; n /= 2, i *= i) {
            if (n % 2)
                res *= i;
        }
        return res;
    }
public:
    /**
     * @param x: the base number
     * @param n: the power number
     * @return: the result
     */
    double myPow(double x, int n) {
        return n < 0 ? 1.0 / _myPow(x, -n) : _myPow(x, n);
    }
};

// Recursion version
class Solution {
private:
    double _myPow(double x, long long n) {
        // 使用long long以避免 -2147483648 边界数据出错
        if (n == 0)
            return 1.0;
        double half = _myPow(x, n / 2);
        if (n % 2)
            return half * half * x;
        else
            return half * half;
    }
public:
    /**
     * @param x: the base number
     * @param n: the power number
     * @return: the result
     */
    double myPow(double x, int n) {
        return n < 0 ? 1.0 / _myPow(x, -n) : _myPow(x, n);
    }
};
```

### 二叉树的序列化与反序列化（Serialize and Deserialize Binary Tree）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

serialize()采用bfs，对当前二叉树搜索，遍历vector，将当前节点左右儿子依次存入vector，空节点需要删去。

deserialize()首先切割字符串，然后用isLeftChild标记是当前是左右儿子，数字转化为字符串，存为队列首节点的左右儿子。
```
```c++
class Solution {
public:
    /**
     * This method will be invoked first, you should design your own algorithm 
     * to serialize a binary tree which denote by a root node to a string which
     * can be easily deserialized by your own "deserialize" method later.
     */
    vector<string> split(const string &str, string delim) {
        vector<string> results;
        int lastIndex = 0, index;
        while ((index = str.find(delim, lastIndex)) != string::npos) {
            results.push_back(str.substr(lastIndex, index - lastIndex));
            lastIndex = index + delim.length();
        }
        if (lastIndex != str.length()) {
            results.push_back(str.substr(lastIndex, str.length() - lastIndex));
        }
        return results;
    }
    string serialize(TreeNode *root) {
        if (root == NULL) {
            return "{}";
        }
        vector<TreeNode *> q;
        q.push_back(root);
        for(int  i = 0; i < q.size(); i++) {
            TreeNode * node = q[i];
            if (node == NULL) {
                continue;
            }
            q.push_back(node->left);
            q.push_back(node->right);
        }
        while (q[q.size() - 1] == NULL) {
                q.pop_back();
        }
        string sb="";
        sb += "{";
        sb += to_string(q[0]->val);
        for (int i = 1; i < q.size(); i++) {
            if (q[i] == NULL) {
                sb += (",#");
            } 
            else {
                sb += ",";
                sb += to_string(q[i]->val);
            }
        }
        sb += "}";
        return sb;
    }
    /**
     * This method will be invoked second, the argument data is what exactly
     * you serialized at method "serialize", that means the data is not given by
     * system, it's given by your own serialize method. So the format of data is
     * designed by yourself, and deserialize it here as you serialize it in 
     * "serialize" method.
     */
    TreeNode * deserialize(string &data) {
        // write your code here
        if (data == "{}") return NULL;
        vector<string> vals = split(data.substr(1, data.size() - 2), ",");
        TreeNode *root = new TreeNode(atoi(vals[0].c_str()));
        queue<TreeNode *> Q;
        Q.push(root);
        bool isLeftChild= true;
        for (int i = 1; i < vals.size(); i++) {
            if (vals[i] != "#") {
                TreeNode *node = new TreeNode(atoi(vals[i].c_str()));
                if (isLeftChild) Q.front()->left = node;
                else Q.front()->right = node;
                Q.push(node);
            }
            if (!isLeftChild) {
                Q.pop();
            }
            isLeftChild = !isLeftChild;
        }
        return root;
    }
};
```

### 字母大小写全排列（Letter case permutation）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

递归（Recursion）
```
```c++
class Solution {
public:
    vector<string> letterCasePermutation(string S) {
        vector<string> res;
        helper(S, res, {}, 0);
        return res;
    }
    // 利用回溯法找到所有的字符串
    void helper(const string S, vector<string>& res, string path, int start) {
        if (start == S.size()) {
            res.push_back(path);
            return;
        }
        if (S[start] >= '0' && S[start] <= '9') {
            helper(S, res, path + S[start], start + 1);
        } else {
            helper(S, res, path + (char)toupper(S[start]), start + 1);
            helper(S, res, path + (char)tolower(S[start]), start + 1);
        }
    }
};
```

### 有效的括号（Valid Parentheses）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

栈（Stack）
```
```c++
class Solution {
public:
    bool isValid(string s) {
        stack<char> str;
        
        for (auto c : s) {
            if (c == '(' || c == '[' || c == '{')
                str.push(c);
            else if (c == ')' || c == ']' || c == '}') {
                if (str.empty()) return false;
                switch (c) {
                    case ')':
                        if (str.top() != '(') return false;
                        break;
                    case ']':
                        if (str.top() != '[') return false;
                        break;
                    case '}':
                        if (str.top() != '{') return false;
                        break;
                    default:
                        break;
                }
                str.pop();
            }
        }
        
        return str.empty();
    }
};
```

### 外观数列（Count and Say）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

滑动窗口（Sliding window）
```
```c++
class Solution {
public:
    string countAndSay(int n) {
        string res = "1";
        for (int i = 2; i <= n; i++) {
            string tmp = "";
            char c = res[0];
            int count = 1;
            for (int j = 1; j < res.size(); j++) {
                if (res[j] != c) {
                    tmp.push_back('0' + count);
                    tmp.push_back(c);
                    count = 0;
                    c = res[j];
                }
                count ++;
            }
            tmp.push_back('0' + count);
            tmp.push_back(c);
            res = tmp;
        }
        
        return res;
    }
};
```

### 合并区间（Merge Intervals）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

逐个比较相邻区间
```
```c++
class Solution {
public:
    /**
     * @param intervals: interval list.
     * @return: A new interval list.
     */
    vector<Interval> merge(vector<Interval> &intervals) {
        // write your code here
        if (intervals.size() < 2) return intervals;
        
        sort(intervals.begin(), intervals.end(), [](Interval &a, Interval &b){ return a.start < b.start;});
        vector<Interval> ans;
        
        for (auto &inte : intervals) {
            if (ans.empty())
                ans.push_back(inte);
            else {
                int sz = ans.size();
                if (ans[sz-1].end < inte.start)
                    ans.push_back(inte);
                else {
                    ans[sz-1].end = ans[sz-1].end > inte.end ? ans[sz-1].end : inte.end;
                }
            }
        }
        
        return ans;
        
    }
};
```

### 子集（Subsets）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

从输出列表中的空子集开始。每一步考虑一个新的整数，并从现有子集生成新的子集。
```
![Cascading](https://leetcode.com/problems/subsets/Figures/78/recursion.png)
```c++
class Solution {
 public:
  static vector<vector<int>> subsets(const vector<int>& nums) {
    vector<vector<int>> subsets;
    
    subsets.push_back(vector<int>());
    for (auto num : nums) {
      int n = subsets.size();
      for (int i = 0; i < n; i++) {
        vector<int> new_set(subsets[i]);
        new_set.push_back(num);
        subsets.push_back(new_set);
      }
    }

    return subsets;
  }
};
```

### 子集 II（Subsets II）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

从输出列表中的空子集开始。每一步考虑一个新的整数，并从现有子集生成新的子集。
```



### 验证外星语字典（Verify a Alien Dictionary）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

比较相邻单词（Compare Adjacent Words）。
```
```c++
class Solution {
public:
    bool isAlienSorted(vector<string>& words, string order) {
        for (int i = 0; i < words.size() - 1; i++) {
        string word1 = words[i];
        string word2 = words[i + 1];
        int i1 = 0, i2 = 0;
        while (word1[i1] == word2[i2]) {
            i1++, i2++;
        }
        int r = order.find(word1[i1]);   
        int s = order.find(word2[i2]);
        if (r > s) return false;
    }
    return true;
    }
};
```

### 盛最多水的容器（Container With Most Water）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

双指针（Two pointers）。
```
```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int max_a = 0; 
        
        int st = 0, end = height.size()-1;
        while (st < end) {
            int water = min(height[st], height[end]) * (end-st);
            max_a = max(max_a, water);
            if (height[st] < height[end])
                st ++;
            else
                end --;
        }
        
        return max_a;
    }
};
```

### 移动零（Move Zeroes）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

维护一个 lastzero 下标记录。
```
```c++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int lastNonZeroFoundAt = 0;
        // If the current element is not 0, then we need to
        // append it just in front of last non 0 element we found. 
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != 0) {
                nums[lastNonZeroFoundAt++] = nums[i];
            }
        }
        // After we have finished processing new elements,
        // all the non-zero elements are already at beginning of array.
        // We just need to fill remaining array with 0's.
        for (int i = lastNonZeroFoundAt; i < nums.size(); i++) {
            nums[i] = 0;
        }
    }
};
```

### 数字补数（Number Complement）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

使用异或（Xor）
```
```c++
class Solution {
public:
    int findComplement(int num) {
        int bit = 1, tp = num;
        while(tp) {
            num ^= bit;
            bit <<= 1;
            tp >>= 1;
        }
        return num;
    }
};
```

### 和为 K 的子数组（Subarray Sum Equals K）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

使用前缀和（Cumulative Sum）
```
```c++
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int count = 0, sum = 0;
        unordered_map<int, int> m;
        m[0] = 1;
        for (int i=0; i<nums.size(); i++) {
            sum += nums[i];
            if (m.find(sum - k) != m.end())
                count += m[sum - k];
            m[sum]++;
        }
        return count;
    }
};
```


### 最小覆盖子串（Minimum Window Substring）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：
我们将使用双指针（two pointer）方法来解决这个问题。

这里的思路是：
1. 我们把 t 的字符存储在一个映射 mapt 里。
2. 我们有两个指针 l 和 r。
3. 当我们遍历 s 时，检查字符是否在 mapt 中找到，如果是，就把该字符存入另一个哈希映射 maps 里。
4. 如果 s 中映射字符的频率小于或等于 mapt，我们递增一个 letter counter 变量，它让我们知道何时达到 t 字符串的大小。
5. 我们用一个 while 循环来寻找包含 t 中所有字符的最小覆盖子串。

S = "ADOBECODEBANC" 且 T = "ABC"
```
```c++
string minWindow(string s, string t) {
    unordered_map<char,int> map1;
    unordered_map<char,int> map2;
    int check=INT_MAX;
    string result;
    for(char &ch:t)map1[ch]++;
    int slow=0,fast=0,lettercounter=0;
    for(;fast<s.length();fast++)
    {
        char ch=s[fast];
        if(map1.find(ch)!=map1.end())
        {
            map2[ch]++;
        if(map2[ch]<=map1[ch])
            lettercounter++;
        }
        if(lettercounter>=t.length())
        {
            while(map1.find(s[slow])==map1.end()||map2[s[slow]]>map1[s[slow]])
            {
                map2[s[slow]]--;
                slow++;
            }
            if(fast-slow+1<check)
            {
                check=fast-slow+1;
                result=s.substr(slow,check);
            }
        }
    }
    return result;
}
```

### 找第 K 大的元素（Find the Kth largest item）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

快速选择（Quick Select）
```
```c++
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
	//partition rule: >=pivot   pivot   <=pivot
	int left=0,right=nums.size()-1,idx=0;
	while(1){
		idx = partition(nums,left,right);
		if(idx==k-1) break;
		else if(idx < k-1) left=idx+1;
		else right= idx-1;
	}
	return nums[idx];
    }
    int partition(vector<int>& nums,int left,int right){//hoare partition
        int pivot = nums[left], l=left+1, r = right;
        while(l<=r){
            if(nums[l]<pivot && nums[r]>pivot) swap(nums[l++],nums[r--]);
            if(nums[l]>=pivot) ++l;
            if(nums[r]<=pivot) --r;
        }
        swap(nums[left], nums[r]);
        return r;
    }
};
```

### 统计重复项（Count Duplicates）
***复杂度（Big O）：*** O(n) 时间，O(n) 空间
```
提示（Tips）：

哈希映射（Hashmap）
```
```c++
class Solution {
public:
    vector<int> countduplicates(vector<int> &nums) {
        // write your code here
        unordered_map<int, bool> dup;
        vector<int> ans;

        for (auto n : nums) {
            if ((dup.find(n) != dup.end()) && !dup[n]) {
                ans.push_back(n);
                dup[n] = true;
            }
            else if (dup.find(n) == dup.end())
                dup[n] = false;
        }

        return ans;
    }
};
```

### 三数之和（3 Sum）
***复杂度（Big O）：*** O(n^2) 时间，O(1) 空间
```
提示（Tips）：

排序 + 两数之和 II（双指针）。

注意重复项。
```
```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(begin(nums), end(nums));
        vector<vector<int>> res;
        for (int i = 0; i < nums.size() && nums[i] <= 0; ++i)
            if (i == 0 || nums[i - 1] != nums[i]) {
                twoSumII(nums, i, res);
            }
        return res;
    }
    void twoSumII(vector<int>& nums, int i, vector<vector<int>> &res) {
        int lo = i + 1, hi = nums.size() - 1;
        while (lo < hi) {
            int sum = nums[i] + nums[lo] + nums[hi];
            if (sum < 0) {
                ++lo;
            } else if (sum > 0) {
                --hi;
            } else {
                res.push_back({ nums[i], nums[lo++], nums[hi--] });
                while (lo < hi && nums[lo] == nums[lo - 1])
                    ++lo;
            }
        }
    }
};
```

### 删除排序链表中的重复元素 II（Remove Duplicates from Sorted List II）
***复杂度（Big O）：*** O(n) 时间，O(1) 空间
```
提示（Tips）：

双指针 pre 和 cur。
```
```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode sentinal, *pre, *cur, *nxt;
        
        sentinal.next = head;
        sentinal.val = INT_MIN;
        pre = &sentinal;
        cur = pre;
        
        while (cur && cur->next) {
            while (cur->next && cur->val == cur->next->val) {
                cur = cur->next;
            }
            cur = cur->next;
            pre->next = cur;
            
            if (cur && cur->next && cur->val != cur->next->val)
                pre = pre->next;
        }
        
        return sentinal.next;
    }
};
```

### 删除链表倒数第 N 个节点（Remove Nth node from the end of list）
***复杂度（Big O）：*** O(L) 时间，O(1) 空间
```
提示（Tips）：

一次遍历算法：

第一个指针从头部开始向前移动 n+1 步，而第二个指针从链表头部开始。现在两个指针恰好相距 n 个节点。我们通过同时推进两个指针来保持这个固定间距，直到第一个指针越过最后一个节点。第二个指针将指向从末尾数的第 n 个节点。我们把第二个指针所引用节点的 next 指针重新链接，使其指向该节点的下一个节点。
```
```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if (head == nullptr || (head->next == nullptr && n > 0))
            return nullptr;
        
        ListNode sentinal, *first, *second;
        
        sentinal.next = head;
        first = &sentinal;
        second = &sentinal;
        
        while(n--) {
            if (second == nullptr)
                return nullptr;
            second = second->next;
        }
        
        while(second->next) {
            first = first->next;
            second = second->next;
        }
        
        first->next = first->next->next;
        
        return sentinal.next;
    }
};
```

### 对称二叉树（Symmetric Tree）
***复杂度（Big O）：*** O(log(n)) 时间，O(1) 空间
```
提示（Tips）：

递归（Recursion）。
```
```c++
class Solution {
public:
    /**
     * @param root: root of the given tree
     * @return: whether it is a mirror of itself
     */
    bool isSymmetric (TreeNode* root) {
        // Write your code here
        return root == nullptr || isSymmetricHelp (root->left, root->right);
    }
    bool isSymmetricHelp (TreeNode* left, TreeNode* right) {
        if (left == nullptr || right == nullptr) {
            return left == nullptr && right == nullptr;
        }
        if (left->val != right->val) {
            return false;
        }
        return isSymmetricHelp (left->left, right->right) && isSymmetricHelp (left->right, right->left);
    }
};
```

### 只出现一次的数字 II（Single Number II）
***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

统计重复位。如果对 3 取模为 1，那么该数一定设置了这一位。
```
```c++
class Solution {
public:
    void count_bits(int arr[], int val) {
        for (int i = 0; i < 32; i++) {
            if (val & (0x1 << i))
                arr[i] ++;
        }
    }
    
    int singleNumber(vector<int>& nums) {
        int arr[32] = {};
        int ret = 0;
        
        for (auto num : nums) {
            count_bits(arr, num);
        }
        
        for (int i = 0; i < 32; i++) {
            arr[i] = arr[i]%3;
            ret |= (arr[i] << i);
        }
        
        return ret;
    }
};
```

### 多数元素（Majority Item）
***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

统计重复位。如果某一位出现超过 n/2 次，那它一定在该数中。
```
```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int n = nums.size();
        int ans = 0, bits[32] = {0};
        
        for (auto num : nums) {
            for (int i = 0; i < 32; i++) {
                bits[i] += ((num >> i) & 0x1);
            }
        }
        
        for (int i = 0; i < 32; i++) {
            if (bits[i] > n/2)
                ans |= (1U << i);
        }
        
        return ans;
    }
};
```

### 划分字母区间（Partition Labels）
***复杂度（Big O）：*** O(N) 时间，O(1) 空间
```
提示（Tips）：

先记录每个字母最后一次出现的下标。然后再遍历一遍数组进行划分。
```
```c++
class Solution {
public:
    vector<int> partitionLabels(string S) {
        int letters[26] = {0};
        int n = S.size();
        vector<int> ans;
        
        if (n == 0)
            return ans;
        
        for (int i = 0; i < n; i++) {
            // record the last occurance index of the letter
            letters[S[i]-'a'] = i;
        }
        
        int p_size = letters[S[0]-'a'];
        for (int i = 0; i < n; i++) {
            // record the last occurance index of the letter
            if (i <= p_size) {
                p_size = max(letters[S[i]-'a'], p_size);
            } else {
                ans.push_back(p_size);
                p_size = letters[S[i]-'a'];
            }
        }
        ans.push_back(p_size);
        
        for (int i = ans.size()-1; i >= 1; i--) {
            ans[i] = ans[i] - ans[i-1];
        }
        ans[0] += 1;
        
        return ans;
    }
};
```

### 最长回文子串（Longest Palindromic Substring）
***复杂度（Big O）：*** O(N^2 或 N^3) 时间，O(1) 空间
```
提示（Tips）：

DP 方法：

为了改进暴力解法，我们首先观察如何避免验证回文时的冗余重算。考虑 "ababa" 这个例子。如果我们已知 "bab" 是回文，那么显然 "ababa" 也必须是回文，因为左右两端字母相同。

p(i, j) 表示子串 Si...Sj 是否是回文。

因此：

p(i, j) = (p(i+1, j-1) && Si == Sj)

基础情况：

p(i, i) = true
p(i, i+1) = (Si == S_(i+1))
```
```c++
//Brutal Force
class Solution {
public:
    bool isPanlidrome(string &s, int st, int end) {
        while (st < end) {
            if (s[st++] != s[end--])
                return false;
        }
        
        return true;
    }
    
    string longestPalindrome(string s) {
        int n = s.size();
        string ans = "";
     
        for (int i = 0; i < n; i++) {
            int st = i, end = n-1; 
            while (st <= end) {
                // if it is guanranteed that ans has bigger size, go to the next index
                if ((end - st + 1) <= ans.size()) {
                    break;
                } else if (s[st] == s[end]){
                    if (isPanlidrome(s, st, end)) {
                        ans = s.substr(st, end-st+1);
                    }
                } else {
                    end --;
                }
            }
        }
        
        return ans;
    }
};

// DP solution
string longestPalindrome(string s) {
	const int n = s.size();
	if(n==0) return "";
	int dp[n][n], maxlen =1 , left=0;// maxlen = 1 when only one word
	memset(dp,0,n*n*sizeof(int));
	for(int i=0;i<n;++i){
		dp[i][i] = 1;
		for(int j=0;j<i;++j){
			dp[j][i] = (s[j]==s[i]  && (i-j< 2 || dp[j+1][i-1]));
			if(dp[j][i] && maxlen < i-j+1){
				left = j;
				maxlen = i-j+1;
			}
		}
	}
	return s.substr(left,maxlen);
}
```

### 另一棵树的子树（Subtree of Another Tree）
***复杂度（Big O）：*** O(M*N) 时间，O(1) 空间
```
提示（Tips）：

递归（Recursion）
```
```c++
class Solution {
public:
    bool helper(TreeNode* s, TreeNode* t) {
        if (s == nullptr && t == nullptr)
            return true;
        
        if ((s == nullptr && t != nullptr) || (s != nullptr && t == nullptr))
            return false;
            
        return s->val == t->val && helper(s->left, t->left) && helper(s->right, t->right);
    }
    
    bool isSubtree(TreeNode* s, TreeNode* t) {
        if ((s == nullptr && t != nullptr) || (s != nullptr && t == nullptr))
            return false;
        
        if (helper(s, t))
            return true;
        
        return isSubtree(s->left, t) || isSubtree(s->right, t);
    }
};
```

### 最小栈（Min Stack）
***复杂度（Big O）：*** O(M*N) 时间，O(1) 空间
```
提示（Tips）：

在栈中存储 <new_val, current_min> 这样的对（pair）。
```
```c++
class MinStack {
    stack<pair<int, int>> data;
    
public:
    /** initialize your data structure here. */
    MinStack() {
        
    }
    
    void push(int val) {
        int min_val = (val < getMin()) ? val : getMin();
        data.push(make_pair(val, min_val));
    }
    
    void pop() {
        if (!data.empty())
            data.pop();
    }
    
    int top() {
        pair<int, int> item = data.top();
        return item.first;
    }
    
    int getMin() {
        if (data.empty())
            return INT_MAX;
        
        pair<int, int> item = data.top();
        return item.second;
    }
};
```

### 单词拆分（Word Break）
***复杂度（Big O）：*** O(2^n 或 n^3) 时间，O(1) 空间
```
提示（Tips）：

为了优化暴力法，我们加入记忆化：

在上一个方法中我们能看到许多子问题是冗余的，即我们对同一个字符串多次调用了递归函数。为了避免这一点，我们可以使用记忆化方法，即用一个数组 memo 存储子问题的结果。这样，当某个特定字符串再次被调用时，如果它的值已经计算过，就直接从 memo 数组中取值并返回。

加入记忆化后，许多冗余的子问题被避免，递归树被剪枝，从而大幅降低了时间复杂度。
```
```c++
// Brutal Force
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        set<string> word_set(wordDict.begin(), wordDict.end());
        return wordBreakRecur(s, word_set, 0);
    }

    bool wordBreakRecur(string& s, set<string>& word_set, int start) {
        if (start == s.length()) {
            return true;
        }
        for (int end = start + 1; end <= s.length(); end++) {
            if (word_set.find(s.substr(start, end - start)) != word_set.end() and
                wordBreakRecur(s, word_set, end)) {
                return true;
            }
        }
        return false;
    }
};

// Recursion with memo
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        set<string> word_set(wordDict.begin(), wordDict.end());
        // In the memo table, -1 stands for the state not yet reached,
        // 0 for false and 1 for true
        vector<int> memo(s.length(), -1);
        return wordBreakMemo(s, word_set, 0, memo);
    }

    bool wordBreakMemo(string& s, set<string>& word_set, int start, vector<int>& memo) {
        if (start == s.length()) {
            return true;
        }
        if (memo[start] != -1) {
            return memo[start];
        }
        for (int end = start + 1; end <= s.length(); end++) {
            if (word_set.find(s.substr(start, end - start)) != word_set.end() and
                wordBreakMemo(s, word_set, end, memo)) {
                return memo[start] = true;
            }
        }
        return memo[start] = false;
    }
};
```

## 相关页面

- [[LeetCode_for_Embedded_Developer]]
- [[LeetCode_for_embedded_advanced]]
- [[00-索引]]

返回索引 [[00-索引]]。
