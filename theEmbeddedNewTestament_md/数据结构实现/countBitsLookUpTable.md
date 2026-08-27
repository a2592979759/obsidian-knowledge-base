---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/countBitsLookUpTable
created: 2026-08-27
---

# 用查表法统计置位个数（Count Set Bits with lookup table）

使用查表法是一种**非常快速**地检查值的方式。这特别适用于类似“[统计（大）数组中设置位的数量](https://www.geeksforgeeks.org/program-to-count-number-of-set-bits-in-an-big-array/)”这类问题。

基本思路：我们有一个 256 项的数组，记录 0~255（覆盖 8 位表示）的置位个数。然后我们可以把 32 位数字拆成 4 段，用该表检查每 8 位并把结果累加。

## 代码

```c++
#include <bits/stdc++.h>
using namespace std;
 
int BitsSetTable256[256];
 
// 初始化查表函数
void initialize() 
{ 
 
    // 以算法方式初始生成表
    BitsSetTable256[0] = 0; 
    for (int i = 0; i < 256; i++)
    { 
        BitsSetTable256[i] = (i & 1) + 
            BitsSetTable256[i / 2]; 
    } 
} 
 
// 返回 n 中置位个数的函数
int countSetBits(int n) 
{ 
    unsigned char *num = (unsigned char *) &n;
    return (BitsSetTable256[num[0]] + 
            BitsSetTable256[num[1]] + 
            BitsSetTable256[num[2]] + 
            BitsSetTable256[num[3]]);

    /*return (BitsSetTable256[n & 0xff] + 
            BitsSetTable256[(n >> 8) & 0xff] + 
            BitsSetTable256[(n >> 16) & 0xff] + 
            BitsSetTable256[n >> 24]);*/
} 
 
// 驱动代码
int main() 
{ 
    // 初始化查表
    initialize(); 
    int n = 9; 
    cout << countSetBits(n);
} 
```

## 分析

- 表项 `BitsSetTable256[i] = (i & 1) + BitsSetTable256[i/2]`，利用递推：`i` 的最低比特加上 `i/2` 的置位个数。
- `countSetBits` 通过一个 `unsigned char*` 分别取 32 位数的最低 4 个字节，查表得到各自置位个数再求和。也可用移位+掩码的等价写法（见被注释掉的部分）。

复杂度：预处理 O(256)，每次查询 O(1)。比逐位循环（O(32)）快得多。

## 相关文档
- [[bitsArray]] —— 位数组
- [[positionOfSetBit]] —— 唯一置位的位置
- [[flipBitsNumber]] —— 翻转比特计数
