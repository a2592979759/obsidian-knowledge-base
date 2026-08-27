---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/SwapBitsInNumber
created: 2026-08-27
---

# 交换指定位（Swap bits in a given number）

给定一个数 x 以及它在二进制表示中的两个位置 p1、p2，编写一个函数交换这两个位置的 n 个比特并返回结果。题目保证两组比特互不重叠。

步骤：
1. 将第一组比特移到最右侧：
   `set1 = (x >> p1) & ((1U << n) - 1)`。
   这里 `(1U << n) - 1` 得到一个低 n 位全 1、其余为 0 的数，与之相与后，除低 n 位外的比特都变成 0。
2. 将第二组比特移到最右侧：
   `set2 = (x >> p2) & ((1U << n) - 1)`
3. 将两组比特异或：
   `xor = (set1 ^ set2)`
4. 将 xor 放回原始位置：
   `xor = (xor << p1) | (xor << p2)`
5. 最后将 xor 与原数异或，从而完成两组交换：
   `result = x ^ xor`

## 代码

```c
#include <stdio.h>
// C 程序：交换整数中的比特位
#include<stdio.h>
 
// 该函数交换整数 n 中位置 p1 和 p2 的比特(单比特版)
int swapBit(unsigned int n, unsigned int p1, unsigned int p2)
{
    /* 将 p1 位移到最右侧 */
    unsigned int bit1 =  (n >> p1) & 1;
 
    /* 将 p2 位移到最右侧 */
    unsigned int bit2 =  (n >> p2) & 1;
 
    /* 异或这两个比特 */
    unsigned int x = (bit1 ^ bit2);
 
    /* 把异或结果放回原始位置 */
    x = (x << p1) | (x << p2);
 
    /* 将 'x' 与原数异或，从而交换两组 */
    unsigned int result = n ^ x;
}
 
/* 驱动代码 */
int main()
{
    int res =  swapBits(28, 0, 3);
    printf("Result = %d ", res);
    return 0;
}

// 交换 n 个比特的版本
int swapBits(unsigned int x, unsigned int p1, unsigned int p2, unsigned int n)
{
    /* 将第一组比特全部移到最右侧 */
    unsigned int set1 = (x >> p1) & ((1U << n) - 1);
 
    /* 将第二组比特全部移到最右侧 */
    unsigned int set2 = (x >> p2) & ((1U << n) - 1);
 
    /* 异或两组 */
    unsigned int xor = (set1 ^ set2);
 
    /* 将异或结果放回原始位置 */
    xor = (xor << p1) | (xor << p2);
 
    /* 将 'xor' 与原数异或，从而交换两组 */
    unsigned int result = x ^ xor;
 
    return result;
}
 
/* 驱动代码 */
int main()
{
    int res = swapBits(28, 0, 3, 2);
    printf("\nResult = %d ", res);
    return 0;
}
```

## 分析

核心技巧：`x ^ xor` 能交换两组比特。因为两组的差异被提取到 `xor` 中，与原数异或时，bit 值不同的位置会翻转，从而完成交换。

复杂度 O(1)。

## 相关文档
- [[swapOddEvenBits]] —— 交换奇偶位（本方法的特殊情形）
- [[singleNumber]] —— 用异或找只出现一次的数
