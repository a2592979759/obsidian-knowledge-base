---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/leetcodeOSflavor.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 带操作系统风格的 LeetCode 题（Leetcode problems with OS flavor）

1. 读取 4 字节（Read 4 bytes）    简单（Easy）
2. 读取 4 字节 II（Read 4 bytes II，多次调用）    困难（Hard）
3. LRU 缓存（LRU cache）    困难（Hard）
4. LFU 缓存（LFU cache）    困难（Hard）
5. 函数独占时间（Exclusive Time of Functions）    中等（Medium）
6. 任务调度器（Task Scheduler）    中等（Medium）



## 实现（Implementation）

### 读取 4 字节（Read 4 bytes）

### 读取 4 字节 II（Read 4 bytes II）

### LRU 缓存（LRU cache）
提示：哈希表（Hashmap）+ 双向链表（DoubleLinkedList）

该问题可以用一个哈希表来解决，哈希表记录了键以及它们在双向链表中的值。这使 put 和 get 操作的时间复杂度均为 \mathcal{O}(1)O(1)，并且也能在 \mathcal{O}(1)O(1) 时间内移除最早添加的节点。

```c++
class LRUCache {
    unordered_map<int, int> cache{};
    list<int> table{};
    int cap;

    void move_to_first(int item) {
        table.remove(item);
        table.push_back(item);
    }
    
public:
    LRUCache(int capacity) {
        this->cap = capacity;
    }
    
    int get(int key) {
        if (cache.find(key) == cache.end()) {
            return -1;
        }
        move_to_first(key);
        return cache[key];
    }
    
    void put(int key, int value) {
        if (cache.find(key) != cache.end()) {
            cache[key] = value;
            move_to_first(key);
        } else {
            // if full, evict the first item first
            if (cache.size() == cap) {
                cache.erase(table.front());
                table.erase(table.begin());
            }
            
            move_to_first(key);
            cache[key] = value;
        }
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
### 函数独占时间（Exclusive Time of Functions）

### 任务调度器（Task Scheduler）
两种情况：
- 最频繁的任务不够频繁，不足以强制出现空闲槽位。
- 最频繁的任务足够频繁，会强制出现一些空闲槽位。

第一种情况很直接，因为槽位总数由任务数量决定：len(tasks)

第二种情况稍微棘手，需要知道最频繁任务的数量 n_max 和频率 f_max。

如果要使用的槽位数由最频繁任务决定，那么它等于

    (f_max - 1) * (n + 1) + n_max。

```c++
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        vector<int> count(26, 0);
        int tasks_size = tasks.size();
        int max_freq = 0;
        for (auto &it : tasks) {
            count[it - 'A']++; 
            max_freq = max(max_freq, count[it - 'A']);
        }

        int result = (max_freq - 1) * (n+1);
        for (auto &it : count) {
            if (max_freq == it) {
                result++;
            }
        }

        return max(result, tasks_size);
    }
};
```

## 相关页面
- [[dataStructure]] —— 数据结构题（LRU/LFU 缓存）
- [[Array]] —— 数组相关算法题
- [[linked_list]] —— 链表相关算法题（含 LRU 缓存）
- [[string]] —— 字符串相关算法题
- [[math]] —— 数学类算法题

返回索引 [[00-索引]]
