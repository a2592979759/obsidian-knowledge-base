---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/implementation/consistentHashing.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 描述（Description）

执行水平分片（horizontal shard）的一般数据库方法是取 id 对数据库服务器总数 n 取模，然后找出它位于哪台机器上。这种方法的缺点是，随着数据不断增加，我们需要增加数据库服务器。当 n 变为 n+1 时，几乎所有的数据都必须被移动，这并不一致。为了减少这种朴素的哈希方法（%n）所带来的缺陷，一种新的哈希算法出现了：一致性哈希（Consistent Hashing，一致性哈希）。实现该算法的方式有很多种。在这里，我们实现一个简单的一致性哈希。

取 id 到 360。如果一开始有 3 台机器，那么让 3 台机器分别负责 0~119、120~239、240~359 三个部分。然后，根据模（取余）判断你落在哪个区间，就去哪台机器。
当机器从 n 变为 n+1 时，我们从 n 个区间中找出最大的一个，然后把它分成两半，把一半给第 n+1 台机器。
例如，当从 3 台变为 4 台时，我们找到第三个区间 0~119 是当前最大的区间，然后我们把 0~119 分成 0~59 和 60~119。0~59 仍然给第一台机器，60~119 给第四台机器。
然后从 4 台变为 5 台时，我们找到最大的区间是第三个区间 120~239，拆分成两个后，变成 120~179、180~239。
假设一开始所有数据都在一台机器上。当增加到第 n 台机器时，区间的分布和对应的机器号是怎样的？

## 实现（Implementation）
一致性哈希 I（Consistent Hashing I）
```c++
class Solution {
public:
    /**
     * @param n a positive integer
     * @return n x 3 matrix
     */
    vector<vector<int>> consistentHashing(int n) {
        // Write your code here
        vector<vector<int>> results;
        vector<int> machine = {0, 359, 1};
        results.push_back(machine);

        for (int i = 1; i < n; ++i) {
            int index = 0;
            for (int j = 1; j < i; ++j) {
                if (results[j][1] - results[j][0] + 1 >
                    results[index][1] - results[index][0] + 1)
                    index = j;
            }

            int x = results[index][0];
            int y = results[index][1];
            results[index][1] = (x + y) / 2;
            
            machine[0] = (x + y) / 2 + 1;
            machine[1] = y;
            machine[2] = i + 1;
            results.push_back(machine);
        }

        return results;
    }
};
```

一致性哈希 II（Consistent Hashing II）
```c++
class Solution
{
  public:
    int n, k;
    map<int, int> shards;
    set<int> ids;
    // @param n a positive integer
    // @param k a positive integer
    // @return a Solution object
    static Solution create(int n, int k)
    {
        // Write your code here
        Solution solution = Solution();
        solution.n = n;
        solution.k = k;
        return solution;
    }

    // @param machine_id an integer
    // @return a list of shard ids
    vector<int> addMachine(int machine_id)
    {
        // Write your code here
        vector<int> random_nums;
        for (int i = 0; i < k; ++i)
        {
            int index;
            do
            {
                index = rand() % n;
            } while (ids.find(index) != ids.end());
            ids.insert(index);
            random_nums.push_back(index);
            shards[index] = machine_id;
        }

        sort(random_nums.begin(), random_nums.end());
        return random_nums;
    }

    // @param hashcode an integer
    // @return a machine id
    int getMachineIdByHashCode(int hashcode)
    {
        // Write your code here
        map<int, int>::iterator it = shards.lower_bound(hashcode);
        if (it == shards.end())
            return shards.begin()->second;
        else
            return it->second;
    }
};
```

## 相关页面
- [[consistentHashing]] —— 一致性哈希（概念）
- [[memCache]] —— memCache 实现
- [[caching]] —— 缓存
- [[dataPartitioning]] —— 数据分区
- [[systemDesign]] —— 系统设计总览

返回索引 [[00-索引]]
