---
tags: [位操作, 算法, 嵌入式, 位图]
source: Data_Struct_Implementation/ArrayOfBits
created: 2026-08-27
---

# 位数组概念（Array of Bits）

#### 用法
```
make
./aob
```

## 概述

位数组（Bit Array / BitSet）是一种用一个比特位来表示一个元素存在与否/状态的数据结构。相比于用整个字节存储一个布尔值，位数组能节省大量内存，非常适合嵌入式等内存紧张的场景。

支持的常见操作：
- 设置某位（Set）
- 清除某位（Clear）
- 测试某位（Test）
- 统计置位的个数（Count）

## 与系统库/其它实现的关系
- [[bitsArray]] 提供了 `SetBit`、`ClearBit`、`TestBit` 三个宏的具体实现，是位数组在实际编码中的典型写法。
- 位数组也是[[taskScheduler]] 中 RTOS 就绪集合（`OS_readySet` / `OS_delayedSet`）采用的核心结构。

## 参考
http://www.mathcs.emory.edu/~cheung/Courses/255/Syllabus/1-C-intro/bit-array.html

## 相关文档
- [[bitsArray]] —— 位数组的具体宏实现
- [[countBitsLookUpTable]] —— 一次统计 8 位/32 位置位个数
