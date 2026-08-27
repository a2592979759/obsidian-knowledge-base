---
tags: [并发, 线程, 同步, 嵌入式]
source: Data_Struct_Implementation/concurrency/TheDiningPhilosophers
created: 2026-08-27
---

# 哲学家就餐问题（Dining Philosophers）

### 描述
```来自 LeetCode 并发。```
五位沉默的哲学家围坐在一张摆着意大利面碗的圆桌旁。每对相邻哲学家之间放着一把叉子。

每个哲学家必须交替地思考和进食。然而，只有在拥有左右两把叉子时，哲学家才能吃面。每把叉子只能被一位哲学家持有，因此一位哲学家只有在叉子未被别人使用时才能使用它。当某位哲学家吃完后，需要放下两把叉子，让它们对其他人可用。哲学家可以在叉子可用时取右边的或左边的叉子，但在拿到两把叉子之前不能开始进食。

进食不受剩余面量或胃容量的限制；假设供给无限、需求无限。

设计一种行为规范（并发算法），使得没有哲学家会饿死；即假设没有任何哲学家知道别人何时想要进食或思考，每个人都能永远在进食与思考之间交替。

![本题说明与上图取自 wikipedia.org](https://assets.leetcode.com/uploads/2019/09/24/an_illustration_of_the_dining_philosophers_problem.png)

哲学家的编号按顺时针从 0 到 4。实现函数 `void wantsToEat(philosopher, pickLeftFork, pickRightFork, eat, putLeftFork, putRightFork)`，其中：

- `philosopher` 是想进食的哲学家的 id。
- `pickLeftFork` 和 `pickRightFork` 是你可调用的函数，用于取该哲学家的相应叉子。
- `eat` 是你可调用的函数，让哲学家在拿到两把叉子后进食。
- `putLeftFork` 和 `putRightFork` 是你可调用的函数，用于放下该哲学家的相应叉子。
- 只要哲学家没有请求进食（该函数未被以其编号调用），就认为他们在思考。
- 代表每位哲学家的五个线程会同时使用你的类的一个对象来模拟该过程。该函数可能在同一个哲学家的上一次调用尚未结束时再次被调用。

### 解法

***我的解法是总是先取左叉再取右叉。然而，当所有五位哲学家同时取左叉时，这可能导致死锁。我的解决方法是添加一个信号量，将同时进食的人数限制为 n - 1。***

***另一种解法是确保进食前总是同时拿到两把叉子。***

***高级解法参见 [wiki 链接](https://en.wikipedia.org/wiki/Dining_philosophers_problem)。***

### 代码

```C++
#include <semaphore.h>

class DiningPhilosophers {
    sem_t sem_ph;
    mutex mutexes[5];
public:
    DiningPhilosophers() {
        sem_init(&sem_ph, 0, 4);
    }

    void wantsToEat(int philosopher,
                    function<void()> pickLeftFork,
                    function<void()> pickRightFork,
                    function<void()> eat,
                    function<void()> putLeftFork,
                    function<void()> putRightFork) {
        
         // 找到左右叉子的编号
        
        int left = philosopher;
        int right = (philosopher + 1) % 5;
        
        sem_wait(&sem_ph);
        {
            // 有序加锁以避免死锁：编号小的叉子先取
            std::lock_guard<std::mutex> first(mutexes[std::min(left, right)]);
            std::lock_guard<std::mutex> second(mutexes[std::max(left, right)]);

            pickLeftFork();
            pickRightFork();

            eat();

            putRightFork();
            putLeftFork();
        }
        sem_post(&sem_ph);
    }
};
```

## 分析

- **死锁原因**：若每位哲学家都先取左叉再取右叉（左叉即 `philosopher` 编号，右叉为 `(philosopher+1)%5`），当五人都拿到左叉时，无人能拿到右叉而全部阻塞，形成循环等待。
- **本解法对策一**：用信号量 `sem_ph` 限制最多 4 人能“试图用餐”，从源头避免 5 人同时争抢。
- **对策二**：对两把叉子按编号排序后加锁（先小后大），消除死锁所需的“循环等待”条件（资源全序）。

## 相关文档
- [[ReaderWritter]] —— 读者写者问题
- [[BuildingH2O]] —— 生成水分子
