---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/swapOddEvenBIts
created: 2026-08-27
---

# 交换奇偶位（Swap all odd and even bits）

## 代码

```c
#include<stdio.h>
#include<stdint.h>

// 任务：交换所有奇数和偶数位
// 提示：先取出所有奇数位和所有偶数位的数，再把它们或（OR）在一起。
uint32_t swapOddEven(uint32_t target) {
    return ((target & 0xaaaaaaaa) >> 1) | ((target & 0x55555555) << 1);
}

int main(void) {
    // test 1:
    uint32_t test1 = 0xa;
    printf("%x\n", swapOddEven(test1));
    
    // test 2:
    uint32_t test2 = 0x5;
    printf("%x\n", swapOddEven(test2));
    
    // test 3:
    uint32_t test3 = 0xff;
    printf("%x\n", swapOddEven(test3));
    
    // test 4:
    uint32_t test4 = 0xaa;
    printf("%x\n", swapOddEven(test4));
    
    return 0;
}
```

## 分析

- `0xaaaaaaaa`（二进制 `1010...1010`）掩码取出所有**偶数位**（位 0、2、4…），右移 1 位把它们挪到奇数位。
- `0x55555555`（二进制 `0101...0101`）掩码取出所有**奇数位**（位 1、3、5…），左移 1 位把它们挪到偶数位。
- 二者按位或，即完成奇偶位的交换。

复杂度 O(1)，无分支。

## 相关文档
- [[reverseBits]] —— 反转比特位
- [[rotateLeftRight]] —— 循环移位
