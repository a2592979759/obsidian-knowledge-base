---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/Array.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

1. 一维数组的连续求和（Running Sum of 1d Array）
2. 连续子数组和（Continuous Subarray Sum）
3. 带权重的随机选取（Random Pick with Weight）
4. 适龄朋友（Friends Of Appropriate Ages）
5. 接雨水（Trap Rain Water）
6. 打家劫舍（House Robber）
7. 数组的度（Degree of an array）
8. 解码方法（Decode Ways）
9. 买卖股票的最佳时机 II（Best Time to Sell Stocks II）
10. 单调数组（Monotonic Array）
11. 最长连续递增子序列（Longest Continuous Increasing Subsequence）
12. 最大乘积子数组（Maximum Product Subarray）
13. 缺失的第一个正数（First Missing Positive）
14. 最大交换（Maximum Swap）
15. 会议室 II（Meeting Room II）
16. 颜色分类（Sort Colors）
17. 三数之和（3 Sum）

## 实现（Implementation）

### **一维数组的连续求和（Running Sum of 1d Array）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

使用前一项之和。
```
```c++
class Solution {
public:
    vector<int> runningSum(vector<int>& nums) {
        vector<int> running_sum(nums.size(), 0);
        running_sum[0] = nums[0];
        
        for (int i = 1; i < nums.size(); i++) {
            running_sum[i] = nums[i] + running_sum[i-1];
        }
        
        return running_sum;
    }
};
```

### **连续子数组和（Continuous Subarray Sum）**

***Big O：*** O(n) 时间，O(n) 空间
```
提示：

a%k = x
b%k = x
(a - b) %k = x -x = 0
这里 a - b 表示 i 与 j 之间的和。
```
```c++
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        if(nums.size()<2)
            return false;
    
        unordered_map<int, int> mp;
    
        // <0,-1> can allow it to return true when the runningSum%k=0,
        mp[0]=-1;
        
        int runningSum=0;
        for(int i=0;i<nums.size();i++) {
            runningSum+=nums[i];
            
            if(k!=0) 
                runningSum = runningSum%k;
            
            //check if the runningsum already exists in the hashmap
            if(mp.find(runningSum)!=mp.end()) {
                //if it exists, then the current location minus the previous location must be greater than1
                if(i-mp[runningSum]>1)
                    return true;
            }
            else
                mp[runningSum]=i; 
        }
        return false;
    }
};
```

### **带权重的随机选取（Random Pick with Weight）**

***Big O：*** O(log(n)) 时间，O(n) 空间
```
提示：

前缀和（Prefix Sums）+ 二分查找（Binary Search）。
```
```c++
class Solution {
    vector<int> sums;
    
public:
    Solution(vector<int>& w) {
        sums = vector<int>(w.size(), 0);
        sums[0] = w[0];
        
        for (int i = 1; i < sums.size(); i++) {
            sums[i] = w[i] + sums[i-1];
        }
    }
    
    int pickIndex() {
        int w_index = rand()%(sums.back());
        
        int st = 0;
        int end = sums.size() - 1;
        int ret;
        
        while (st < end) {
            int mid = st + (end-st)/2;
            
            if (sums[mid] <= w_index) {
                st = mid + 1;
            } else{
                end = mid;
            }
        }
        
        return st;
    }
};

```

### **适龄朋友（Friends Of Appropriate Ages）**

***Big O：*** O(nlog(n)) 时间，O(n) 空间
```
提示：
方法一：

先计算目标值，然后进行二分查找。对于已经处理过的年龄，直接累加该年龄的请求数量即可。

方法二：

与其处理全部 20000 人，不如处理一组组 (age, count) 对，表示每个年龄有多少人。由于只有 120 个可能的年龄，这个循环要快得多。
```
```c++
// Approach 2
class Solution {
public:
    int numFriendRequests(vector<int>& ages) {
        int count[121] = {};
        for (int age: ages) count[age]++;

        int ans = 0;
        for (int ageA = 0; ageA <= 120; ageA++) {
            int countA = count[ageA];
            for (int ageB = 0; ageB <= 120; ageB++) {
                int countB = count[ageB];
                if (ageA/2 + 7 >= ageB) continue;
                if (ageA < ageB) continue;
                if (ageA < 100 && 100 < ageB) continue;
                ans += (countA * countB);
                if (ageA == ageB) ans -= countA;
            }
        }

        return ans;
    }
};

// Approach 1
class Solution {
public:
    unordered_map <int,int> map; // key: age, val: FriendRequestCount. 
    
    int findRequests (vector<int> & ages, int index) {
        
        if (map.find(ages[index]) != map.end()) {
            return map[ages[index]];
        }
        
        int left = 0;
        int right = index-1;
        double target = (double) (0.5*ages[index]) + 7; // find ages >= target.
        
        while (left <= right) {
            int mid = left + (right-left)/2;
            if (ages[mid] <= target) {
                left = mid+1;
            } else {
                right = mid-1;
            }
        }
        
        map[ages[index]] = index-left;
        return index-left; // len between index-1 and left. 
    }
    
    int numFriendRequests(vector<int>& ages) {
        
        sort (ages.begin(), ages.end());
        
        int count = 0;
        
        for (int i = ages.size()-1; i >= 0 ; i--) {
            count += findRequests (ages, i);
        }
        return count;
    }
};
```
### **接雨水（Trap Rain Water）**

***Big O：*** O(nlog(n)) 时间，O(n) 空间
```
提示：
方法一：

使用栈。

方法二：

使用双指针。
```

```c++
// Approach 1: stacks
class Solution {
public:
    int trap(vector<int>& height)
    {
        int ans = 0, current = 0;
        stack<int> st;
        while (current < height.size()) {
            while (!st.empty() && height[current] > height[st.top()]) {
                int top = st.top();
                st.pop();
                if (st.empty())
                    break;
                int distance = current - st.top() - 1;
                int bounded_height = min(height[current], height[st.top()]) - height[top];
                ans += distance * bounded_height;
            }
            st.push(current++);
        }
        return ans;
    }
};

// Approach 2: two pointers 
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
### **打家劫舍（House Robber）**

***Big O：*** O(n) 时间，O(n) 空间
```
提示：

带记忆化的动态规划（Dynamic Programming）。
```
```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        if(nums.size() <= 1) return nums.size() == 0 ? 0 : nums[0];
        
        vector<int> dp(nums.size(), 0);
        
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        
        for (int i = 2; i < nums.size(); i++) {
            dp[i] = max(dp[i-2] + nums[i], dp[i-1]);
        }
        
        return dp[nums.size()-1];
    }
};
```

### **数组的度（Degree of an array）**

***Big O：*** O(n) 时间，O(n) 空间
```
提示：

用一个哈希表（Hash map）记录每个数字的出现频率，再用另一个映射记录每个数字的位置。当遇到具有相同度的数字时，计算其长度并取其中的最小值。
```
```c++
class Solution {
public:
    int findShortestSubArray(vector<int>& nums) {
        
    map<int, int> freq;
    map<int, vector<int>> pos;
    int mx = INT_MIN;
    
    /*
    get frequency of each number in array
    get highest degree
    note the positions of frequencies
    */    
    for(int i = 0; i < nums.size(); i++)
    {
        mx = max(mx, ++freq[nums[i]]);        
        pos[nums[i]].push_back(i);
    }
       
    //get shortest distance
    int dist = INT_MAX;
    for(auto num : nums)
    {
        if(freq[num] == mx)            
            dist = min(dist, pos[num].back() - pos[num].front());         
    }
    
    return dist + 1;
        
    }
};
```

### **解码方法（Decode Ways）**

***Big O：*** O(n) 时间，O(n) 空间
```
提示：

动态规划（Dynamic Programming）。
```
```c++
class Solution {
public:
    int numDecodings(string s) {
        // edge cases out - leading zero and single character string
        if (s[0] == '0') return 0;
        if (s.size() == 1) return 1;
        // support variables
        int len = s.size(), dp[len];
        // preparing dp
        dp[0] = 1;
        dp[1] = (s[0] == '1' || s[0] == '2' && s[1] < '7' ? 1 : 0) + (s[1] != '0');
        for (int i = 2; i < len; i++) {
            // edge case: we quit for 2 consecutive zeros
            if (s[i] == '0' && (s[i - 1] > '2' || s[i - 1] == '0')) return 0;
            // base case: we always keep the previous set of combinations, unless we met a 0
            dp[i] = s[i] != '0' ? dp[i - 1] : 0;
            // we go and look 2 positions behind if we can make a digit in the 10-26 range
            if (s[i - 1] == '1' || s[i - 1] == '2' && s[i] < '7') dp[i] += dp[i - 2];
        }
        return dp[len - 1];
    }
};
```
### **买卖股票的最佳时机 II（Best Time to Sell Stocks II）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

贪心法（Greedy）。低价买入，高价卖出。
```
```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int prev = prices[0], res = 0;
        for (int curr: prices) {
            if (prev < curr) res += curr - prev;
            prev = curr;
        }
        return res;
    }
};
```

### **单调数组（Monotonic Array）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

显而易见。
```
```c++
class Solution {
public:
    bool isMonotonic(vector<int>& A) {
        bool increase = true;
        bool decrease = true;
        for(int i = 0; i < A.size() - 1; i++) {
            if(A[i] > A[i+1]) increase = false;
            if(A[i] < A[i+1]) decrease = false;
            if(increase == false && decrease == false) return false;
        }
        return true;
    }
};
```

### **最长连续递增子序列（Longest Continuous Increasing Subsequence）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

显而易见。
```
```c++
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if(nums.size()<=1)return nums.size();
        int answer=1,count=1;
        for(int i=0;i<nums.size()-1;i++){
            if(nums[i]<nums[i+1]){
                count++;
                answer=max(answer,count);
            }
            else{
                count=1;
            }
        }
        return answer;
    }
};
```
### **最大乘积子数组（Maximum Product Subarray）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

动态规划（Dynamic Programming）。在遍历 nums 中的每个数字时，我们需要跟踪到该数字为止的最大乘积（记作 max_so_far）以及到该数字为止的最小乘积（记作 min_so_far）。跟踪 max_so_far 是为了记录正数的累计乘积；跟踪 min_so_far 是为了正确处理负数。
```
```c++
class Solution {
public:
    
    int maxProduct(vector<int>& nums) {
         if (nums.size() == 0) return 0;

        int max_so_far = nums[0];
        int min_so_far = nums[0];
        int result = max_so_far;

        for (int i = 1; i < nums.size(); i++) {
            int curr = nums[i];
            int temp_max = max(curr, max(max_so_far * curr, min_so_far * curr));
            min_so_far = min(curr, min(max_so_far * curr, min_so_far * curr));

            max_so_far = temp_max;

            result = max(max_so_far, result);
        }

        return result;
    }
};
```

### **缺失的第一个正数（First Missing Positive）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

哈希表（Hashmap）。
```
```c++
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        unordered_map<int, int> umap;
        int max_v = 0;
        
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] > 0) {
                umap[nums[i]] = 1;
                max_v = max(max_v, nums[i]);
            }
        }
        
        for (int i = 1; i < max_v; i++) {
            if (umap.find(i) == umap.end())
                return i;
        }
        
        return max_v+1;
    }
};
```

### **最大交换（Maximum Swap）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

贪心法（Greedy）。
```
```c++
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

### **会议室 II（Meeting Rooms II）**

***Big O：*** O(nlog(n)) 时间，O(n) 空间
```
提示：

排序 + 优先队列（Priority Queue）。如果已有会议结束，就把对应的人“踢”出去。
```
```c++
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        
        priority_queue<int, vector<int>, greater<int>> pq;
        
        for (auto v : intervals) {
            // check if the last meetings need to be completed
            // before this meeting starts.
            if (!pq.empty() && pq.top() <= v[0]) {
                pq.pop();
            }
            
            // start this meeting and enter the end time.
            pq.push(v[1]);
        }
        
        return pq.size();
    }
};
```

### **颜色分类（Sort Colors）**

***Big O：*** O(n) 时间（一趟遍历），O(1) 空间
```
提示：

解法的核心思想是让 curr 指针沿数组移动：如果 nums[curr] = 0，就与 nums[p0] 交换；如果 nums[curr] = 2，就与 nums[p2] 交换。
```
```c++
class Solution {

public: 
    void sortColors(vector<int>& nums) {
        int lo = 0, hi = nums.size() - 1, i = 0;

        while (i <= hi) {
            if      (nums[i] == 0) swap(nums, lo++, i++);
            else if (nums[i] == 2) swap(nums, i, hi--);
            else if (nums[i] == 1) i++;
        }
    }

    void swap(vector<int>& nums, int i, int j) {
        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }

};
```

### **三数之和（3 SUM）**

***Big O：*** O(n^2)，O(1) 空间
```
提示：

排序 + 双指针。
```
```c++
class Solution {
public:
	vector<vector<int>> threeSum(vector<int>& nums) {
		vector<vector<int>> ans;
		sort(nums.begin(), nums.end());
		int length = nums.size() - 1, left, right;
		for ( int index = 0; index <= length; ++index )
		{
			if ( index > 0 && nums[index - 1] == nums[index] ) continue; 
			left = index + 1;
			right = length;
			while ( left < right )
			{
				if ( nums[index] + nums[left] + nums[right] < 0 ) ++left;
				else if ( nums[index] + nums[left] + nums[right] > 0 ) --right;
				else
				{
					vector<int> anotherAnswer { nums[index], nums[left], nums[right] };
					ans.push_back(anotherAnswer);
					++left;
					while ( left < right && nums[left] == nums[left - 1] ) ++left;
				}   
			}    
		}     
		return ans;
	}
};
```

## 相关页面
- [[linked_list]] —— 链表相关算法题
- [[dataStructure]] —— 数据结构题（LRU 缓存等）
- [[string]] —— 字符串相关算法题
- [[matrix]] —— 矩阵相关算法题
- [[math]] —— 数学类算法题
- [[leetcodeOSflavor]] —— 带操作系统风格的 LeetCode 题

返回索引 [[00-索引]]
