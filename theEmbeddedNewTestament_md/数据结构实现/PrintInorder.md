---
tags: [并发, 线程, 同步, 嵌入式]
source: Data_Struct_Implementation/concurrency/PrintInorder
created: 2026-08-27
---

# 线程按序执行（Print In Order）

### 描述
```来自 LeetCode 并发标签。```

假设有一个类：
```C++
public class Foo {
  public void first() { print("first"); }
  public void second() { print("second"); }
  public void third() { print("third"); }
}
```
同一个 Foo 实例会传给三个不同的线程。线程 A 调用 `first()`，线程 B 调用 `second()`，线程 C 调用 `third()`。设计一种机制并修改程序，确保 `second()` 在 `first()` 之后执行，`third()` 在 `second()` 之后执行。

示例 1：

输入：[1,2,3]

输出："firstsecondthird"

解释：三个线程被异步触发。输入 [1,2,3] 表示线程 A 调用 `first()`，线程 B 调用 `second()`，线程 C 调用 `third()`。"firstsecondthird" 是正确的输出。

### 解法
***使用信号量（Semaphore）来控制每个线程的进入顺序。***

### 代码

```C++
class Foo {
    sem_t a, b;
public:
    Foo() {
        sem_init(&a, 0, 0);
        sem_init(&b, 0, 0);
    }

    void first(function<void()> printFirst) {
        
        // printFirst() 输出 "first"。不要改动或删除这一行。
        printFirst();
        sem_post(&a);
    }

    void second(function<void()> printSecond) {
        
        // printSecond() 输出 "second"。不要改动或删除这一行。
        sem_wait(&a);
        printSecond();
        sem_post(&b);
    }

    void third(function<void()> printThird) {
        
        // printThird() 输出 "third"。不要改动或删除这一行。
        sem_wait(&b);
        printThird();
    }
};
```

## 分析

- 两个信号量 `a`、`b` 初始为 0，用于传递“前一个步骤已完成”的信号。
- `first()` 完成后 `sem_post(&a)` 释放 `second()`。
- `second()` 用 `sem_wait(&a)` 等待，完成后再 `sem_post(&b)` 释放 `third()`。
- `third()` 用 `sem_wait(&b)` 等待，从而保证严格的先后顺序。

## 相关文档
- [[FizzBuzz]] —— 多线程 FizzBuzz
- [[BuildingH2O]] —— 生成水分子
