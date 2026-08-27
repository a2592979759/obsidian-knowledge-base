---
tags: [并发, 线程, 同步, 嵌入式]
source: Data_Struct_Implementation/concurrency/BuildingH2O
created: 2026-08-27
---

# 生成水分子（Building H2O）

### 描述
```来自 LeetCode 并发标签。```

有两种线程：氧线程和氢线程。你的目标是将这些线程组合成水分子。有一个屏障（barrier），每个线程必须等待，直到能组成一个完整的水分子。氢和氧线程分别会被授予 `releaseHydrogen` 和 `releaseOxygen` 方法，允许它们通过屏障。这些线程必须以三个为一组通过屏障，并且必须能立即互相结合成一个水分子。你必须保证：来自一个水分子的所有线程结合完成后，下一个水分子的任何线程才能结合。

换句话说：

- 若氧线程到达屏障时没有氢线程，它必须等待两个氢线程。
- 若氢线程到达屏障时没有其它线程，它必须等待一个氧线程和另一个氢线程。

我们不必显式匹配线程；也就是说，线程不一定知道它们与哪些线程配对。关键在于线程必须以完整集合通过屏障；因此，若我们把结合的线程序列分成三个一组，每组应包含一个氧线程和两个氢线程。

为氧和氢分子编写同步代码，强制执行这些约束。

### 解法
***典型的读者-写者问题。使用 mutex + 信号量。***

### 代码

```C++
#include <semaphore.h>

class H2O {
    int d; // H - 2 * O
    std::mutex mtx;
    sem_t hy, ox;
    
public:
    H2O(): d(0) {
        sem_init(&hy, 0, 1);
        sem_init(&ox, 0, 0);
    }

    void hydrogen(function<void()> releaseHydrogen) {
        sem_wait(&hy);
        
        mtx.lock();
        d++;
        mtx.unlock();
        // releaseHydrogen() 输出 "H"。不要改动或删除这一行。
        releaseHydrogen();
        
        if (d >= 2)
            sem_post(&ox);
        else
            sem_post(&hy);
    }

    void oxygen(function<void()> releaseOxygen) {
        sem_wait(&ox);
        
        mtx.lock();
        d = 0;
        mtx.unlock();
        
        // releaseOxygen() 输出 "O"。不要改动或删除这一行。
        releaseOxygen();
        
        sem_post(&hy);
       
    }
};
```

## 分析

- `hy` 初始为 1（允许第一个氢线程进入），`ox` 初始为 0。
- 每次氢线程进入 `d++`：若 `d < 2`，`sem_post(&hy)` 放行下一个氢；若 `d >= 2`，`sem_post(&ox)` 放行氧。
- 氧线程用 `sem_wait(&ox)` 等待，进入后将 `d = 0` 重置，然后 `sem_post(&hy)` 放行下一个水分子。

## 相关文档
- [[PrintInorder]] —— 线程按序执行
- [[ReaderWritter]] —— 读者写者问题
