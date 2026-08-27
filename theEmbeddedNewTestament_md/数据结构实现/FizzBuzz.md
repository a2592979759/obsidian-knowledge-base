---
tags: [并发, 线程, 同步, 嵌入式]
source: Data_Struct_Implementation/concurrency/FizzBuzz
created: 2026-08-27
---

# 多线程 FizzBuzz

### 描述
```来自 LeetCode 并发标签。```

编写一个程序，输出从 1 到 n 的数字的字符串表示，但：
- 若数字能被 3 整除，输出 "fizz"。
- 若数字能被 5 整除，输出 "buzz"。
- 若数字同时能被 3 和 5 整除，输出 "fizzbuzz"。

例如 n = 15 时，输出：1, 2, fizz, 4, buzz, fizz, 7, 8, fizz, buzz, 11, fizz, 13, 14, fizzbuzz。

假设给定如下代码：
```C++
class FizzBuzz {
  public FizzBuzz(int n) { ... }               // 构造函数
  public void fizz(printFizz) { ... }          // 只输出 "fizz"
  public void buzz(printBuzz) { ... }          // 只输出 "buzz"
  public void fizzbuzz(printFizzBuzz) { ... }  // 只输出 "fizzbuzz"
  public void number(printNumber) { ... }      // 只输出数字
}
```
用四个线程实现多线程版本的 FizzBuzz。同一个 FizzBuzz 实例会被传给四个不同的线程：

- 线程 A 调用 `fizz()` 检查能否被 3 整除并输出 fizz。
- 线程 B 调用 `buzz()` 检查能否被 5 整除并输出 buzz。
- 线程 C 调用 `fizzbuzz()` 检查能否被 3 和 5 整除并输出 fizzbuzz。
- 线程 D 调用 `number()`，只输出数字。

### 解法

***使用条件变量（condition_variable）+ mutex***

### 代码

```C++
class FizzBuzz {
private:
    int n;
    atomic<int> current;
    mutex mtx;
    condition_variable cv;

protected:
    void do_work(function<void(int)> printN, function<bool()> check) {
        while (current <= n) {
            std::unique_lock<mutex> lk(mtx);
            cv.wait(lk, [&]{return current > n || check();});
                    
            if (current > n) break;
            printN(current);
            current ++;
            cv.notify_all();
        }
    }
    
public:
    FizzBuzz(int n) {
        this->n = n;
        this->current = 1;
    }

    // printFizz() 输出 "fizz"。
    void fizz(function<void()> printFizz) {
        do_work([&](int i){printFizz();}, [&]{return (current%3 == 0) && (current%5 !=0);});
    }

    // printBuzz() 输出 "buzz"。
    void buzz(function<void()> printBuzz) {
        do_work([&](int i){printBuzz();}, [&]{return current%3 != 0 && current%5 ==0;});
    }

    // printFizzBuzz() 输出 "fizzbuzz"。
	void fizzbuzz(function<void()> printFizzBuzz) {
        do_work([&](int i){printFizzBuzz();}, [&]{return current%3 == 0 && current%5 ==0;});
    }

    // printNumber(x) 输出 "x"，其中 x 是整数。
    void number(function<void(int)> printNumber) {
        do_work([&](int i){printNumber(i);}, [&]{return current%3 != 0 && current%5 !=0;});
    }
};
```

## 分析

- 使用一个原子计数器 `current` 表示当前处理的数字。
- 每个线程通过 `do_work` 通用函数处理：用条件变量等待，直到 `current > n`（结束）或 `check()` 满足（即当前数字属于该线程负责的类别）。
- 打印后 `current++` 并 `notify_all()` 唤醒其它线程。
- `atomic` 保证计数器的读写原子性，`mutex` 保护对 `current` 的条件判断。

## 相关文档
- [[PrintInorder]] —— 线程按序执行
- [[BuildingH2O]] —— 生成水分子
