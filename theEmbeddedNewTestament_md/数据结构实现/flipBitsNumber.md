---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/flipBitsNumber
created: 2026-08-27
---

# 翻转比特计数（Count number of bits to be flipped to convert A to B）

给定两个数 `a` 和 `b`，写一个程序统计需要翻转多少位才能把 `a` 变成 `b`。

## 代码

```c
#include<stdio.h>
#include<stdint.h>

typedef uint32_t (*generalTestFunct)(uint32_t num1, uint32_t num2);

uint32_t flipBitsNumber(uint32_t num1, uint32_t num2) {
    num1 ^= num2;

    int res = 0;
    while(num1) {
        res ++;
        num1 &= num1-1;
    }

    return res;
}

int main(void) {
    generalTestFunct test_func1 = flipBitsNumber;
    
    // test 1:
    uint32_t test_num1 = 1;
    uint32_t test_num2 = 1;
    printf("%d\n", test_func1(test_num1, test_num2));
    
    // test 2:
    test_num1 = 0xffff0000;
    test_num2 = 0x1;
    printf("%d\n", test_func1(test_num1, test_num2));
    
    // test 3:
    test_num1 = 0x11001100;
    test_num2 = 1;
    printf("%d\n", test_func1(test_num1, test_num2));

    // test 4:
    test_num1 = 0x00110011;
    test_num2 = 1;
    printf("%d\n", test_func1(test_num1, test_num2));
    
    return 0;
}
```

## 分析

1. `num1 ^= num2`：异或后，`num1` 中为 1 的位就是两个数**不同**的位，即需要翻转的位。
2. 循环用 `num1 &= num1-1` 每次清除最右侧的一个置位，同时 `res++` 计数。

复杂度 O(置位的个数) ≤ O(32)。

## 相关文档
- [[turnoffRightmostBit]] —— 关闭最右侧置位（`x & (x-1)`）
- [[countBitsLookUpTable]] —— 查表统计置位
