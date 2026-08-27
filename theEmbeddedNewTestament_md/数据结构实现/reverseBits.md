---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/reverseBits
created: 2026-08-27
---

# 反转比特位（Reverse Bits）

将 32 位整数的比特顺序完全反转。

## 代码

```c
#include<stdio.h>
#include<stdint.h>

typedef uint32_t (*generalTestFunct)(uint32_t target);

uint32_t reverseBits(uint32_t target) {
    target = ((target & 0xffff0000) >> 16) | ((target & 0x0000ffff) << 16);
    target = ((target & 0xff00ff00) >> 8) | ((target & 0x00ff00ff) << 8);
    target = ((target & 0xf0f0f0f0) >> 4) | ((target & 0x0f0f0f0f) << 4);
    target = ((target & 0xcccccccc) >> 2) | ((target & 0x33333333) << 2);
    target = ((target & 0xaaaaaaaa) >> 1) | ((target & 0x55555555) << 1);

    return target;
}

int main(void) {
    generalTestFunct test_func1 = reverseBits;
    
    // test 1:
    uint32_t test_num1 = 1;
    printf("%x\n", test_func1(test_num1));
    
    // test 2:
    test_num1 = 0xffff0000;
    printf("%x\n", test_func1(test_num1));
    
    // test 3:
    test_num1 = 0x11001100;
    printf("%x\n", test_func1(test_num1));

    // test 4:
    test_num1 = 0x00110011;
    printf("%x\n", test_func1(test_num1));
    
    return 0;
}
```

## 分析

这是**分治法（分而治之）**：每次交换一半的比特。

1. 交换高 16 位与低 16 位。
2. 在每个 16 位内交换高低 8 位。
3. 在每个 8 位内交换高低 4 位。
4. 在每个 4 位内交换高低 2 位。
5. 相邻两两交换。

复杂度固定为 O(1)，无循环、无分支，适合嵌入式。

## 相关文档
- [[endianessSwap]] —— 字节序转换用到类似的掩码移位
- [[rotateLeftRight]] —— 循环移位
- [[swapOddEvenBits]] —— 交换奇偶位
