---
tags: [并发, 线程, 阻塞队列, 嵌入式]
source: Data_Struct_Implementation/concurrency/BoundedQueue
created: 2026-08-27
---

# 有界阻塞队列（Bounded Blocking Queue）

### 描述
```来自 LeetCode 并发标签。```

实现一个线程安全的有界阻塞队列，包含以下方法：

- `BoundedBlockingQueue(int capacity)`：构造函数，以最大容量初始化队列。
- `void enqueue(int element)`：将元素添加到队首。若队列已满，调用线程被阻塞，直到队列不再满。
- `int dequeue()`：返回队尾元素并将其移除。若队列为空，调用线程被阻塞，直到队列不再空。
- `int size()`：返回当前队列中的元素个数。

实现会被多线程同时测试。每个线程要么是只调用 `enqueue` 的生产者线程，要么是只调用 `dequeue` 的消费者线程。每次测试后都会调用 `size` 方法。

请不要使用内置的有界阻塞队列实现，否则在面试中不会被接受。

### 解法

***使用 mutex + 信号量（Semaphore）。Mutex 保护队列操作，信号量保证不超过容量。典型的有界缓冲区、生产者-消费者问题。***

### 代码

```C++
#include <semaphore.h>

class BoundedBlockingQueue {
    int max_cap;
    queue<int> block_queue;
    mutex mtx;
    sem_t sem_de;
    sem_t sem_en;
public:
    BoundedBlockingQueue(int capacity) {
        max_cap = capacity;
        sem_init(&sem_de, 0, 0);
        sem_init(&sem_en, 0, max_cap);
        block_queue = {};
    }
    
    void enqueue(int element) {
        sem_wait(&sem_en);
        mtx.lock();
        block_queue.push(element);
        mtx.unlock();
        sem_post(&sem_de);
        return;
    }
    
    int dequeue() {
        sem_wait(&sem_de);
        mtx.lock();
        int ret = block_queue.front();
        block_queue.pop();
        mtx.unlock();
        sem_post(&sem_en);
        return ret;
    }
    
    int size() {
        mtx.lock();
        int ret = block_queue.size();
        mtx.unlock();
        return ret;
    }
};
```

## 分析

- `sem_en` 初始化为 `max_cap`，表示可用空位数量；`sem_de` 初始化为 0，表示当前元素数量。
- 生产者在写入前 `sem_wait(&sem_en)`，写入后 `sem_post(&sem_de)`。
- 消费者在读出前 `sem_wait(&sem_de)`，读出后 `sem_post(&sem_en)`。
- `mutex` 保护对底层 `queue` 的并发访问，信号量则负责在容量/空满间阻塞。

## 相关文档
- [[circularRingBuffer]] —— 环形缓冲区（生产者/消费者版本）
- [[queue]] —— 队列
