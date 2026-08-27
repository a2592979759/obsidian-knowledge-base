---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/rotateLeftRight
created: 2026-08-27
---

# 循环移位（Rotate bits of a number）

**位旋转（循环移位）**：一种类似移位、但一端掉落的比特会被放回另一端的操作。
- 左旋转：从左侧掉落的比特被放回右侧。
- 右旋转：从右侧掉落的比特被放回左侧。

## 代码

```c
#include<stdio.h>
#include<stdint.h>

// 任务：关闭最右侧置位（注释与代码不符，见实现）
typedef uint32_t (*generalTestFunct)(uint32_t target, int pos);

uint32_t leftRotate(uint32_t target, int pos) {
    return (target >> (32 - (pos%32))) | (target << (pos%32));
}

uint32_t rightRotate(uint32_t target, int pos) {
    return (target >> (pos%32)) | (target << (32-(pos%32)));
}

int main(void) {
    generalTestFunct test_func1 = leftRotate;
    generalTestFunct test_func2 = rightRotate;
    
    // test 1:
    uint32_t test_num1 = 1;
    int shift = 33;
    printf("%x\n", test_func1(test_num1, shift));
    printf("%x\n", test_func2(test_num1, shift));
    
    // test 2:
    test_num1 = 0x1000;
    shift = 1;
    printf("%x\n", test_func1(test_num1, shift));
    printf("%x\n", test_func2(test_num1, shift));
    
    // test 3:
    test_num1 = 0x1100;
    shift = 1;
    printf("%x\n", test_func1(test_num1, shift));
    printf("%x\n", test_func2(test_num1, shift));
    
    // test 4:
    test_num1 = 0x100010;
    shift = 1;
    printf("%x\n", test_func1(test_num1, shift));
    printf("%x\n", test_func2(test_num1, shift));
    
    return 0;
}
```

## 分析

- 左旋转 `n` 位：`(x >> (32 - n)) | (x << n)`。
- 右旋转 `n` 位：`(x >> n) | (x << (32 - n))`。
- 使用 `pos % 32` 处理移动位数 ≥ 32 的情况（例如 `shift = 33` 相当于移动 1 位）。

## 相关文档
- [[reverseBits]] —— 反转比特位
- [[swapOddEvenBits]] —— 交换奇偶位
