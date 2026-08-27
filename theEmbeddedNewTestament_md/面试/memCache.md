---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/SystemDesign/implementation/memCache.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 学习嵌入式系统设计方法论，并在网站上浏览由社区排名的面试题库。
>
> 👉 **[探索系统设计准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_systemdesign)**

---

## 描述（Description）
实现一个支持以下功能的 memcache：

get(curtTime, key)。获取 key 的值，如果 key 不存在则返回 2147483647。
set(curtTime, key, value, ttl)。在 memcache 中设置键值对，并带有生存时间（time to live，ttl）。该 key 从 curtTime 到 curtTime + ttl - 1 有效，并且在 ttl 秒之后过期。如果 ttl 为 0，则 key 永久有效，直到内存耗尽。
delete(curtTime, key)。删除该 key。
incr(curtTime, key, delta)。将该 key 的值增加 delta，并返回新值。如果 key 不存在则返回 2147483647。
decr(curtTime, key, delta)。将该 key 的值减少 delta，并返回新值。如果 key 不存在则返回 2147483647。
保证输入是按递增的 curtTime 给出的。

## 实现（Implementation）

```c++
class Resource {
public:
    int value, expired;
    Resource() {}
    Resource(int v, int e) {
        value = v;
        expired = e;
    }
};

class Memcache {
private:
    map<int, Resource> client;
public:
    Memcache() {
        // initialize your data structure here.
    }

    int get(int curtTime, int key) {
        // Write your code here
        if (client.find(key) == client.end())
            return INT_MAX;

        Resource res = client[key];
        if (res.expired >= curtTime || res.expired == -1)
            return res.value;
        else
            return INT_MAX;
    }

    void set(int curtTime, int key, int value, int ttl) {
        // Write your code here
        int expired;
        if (ttl == 0)
            expired = -1;
        else
            expired = curtTime + ttl - 1;
        client[key] = Resource(value, expired);
    }

    void del(int curtTime, int key) {
        // Write your code here
        if (client.find(key) == client.end())
            return;

        client.erase(client.find(key));
    }
    
    int incr(int curtTime, int key, int delta) {
        // Write your code here
        if (get(curtTime, key) == INT_MAX)
            return INT_MAX;
        client[key].value += delta;
        return client[key].value;
    }
    
    int decr(int curtTime, int key, int delta) {
        // Write your code here
        if (get(curtTime, key) == INT_MAX)
            return INT_MAX;
        client[key].value -= delta;
        return client[key].value;
    }
};
```

## 相关页面
- [[consistentHashing]] —— 一致性哈希
- [[caching]] —— 缓存
- [[consistentHashing_impl]] —— 一致性哈希实现
- [[cacheDesign]] —— 缓存设计示例
- [[systemDesign]] —— 系统设计总览

返回索引 [[00-索引]]
