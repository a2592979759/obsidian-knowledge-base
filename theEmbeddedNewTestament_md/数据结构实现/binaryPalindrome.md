---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/binaryPanlidrom
created: 2026-08-27
---

# 判断二进制表示是否为回文（Check if binary representation of a number is palindrome）

给定一个整数 `x`，编写 C 函数，若 `x` 的二进制表示是回文则返回 true，否则返回 false。
例如，二进制表示为 `10..01` 的数字是回文，而 `10..00` 不是回文。

思路与判断字符串是否为回文类似：我们从最左和最右的比特开始，一位一位比较。若发现不匹配，则返回 false。

## 代码

```c
#include<stdio.h>
#include<stdint.h>

typedef uint32_t (*generalTestFunct)(uint32_t target);

uint32_t isBinaryPanlidrome(uint32_t target) {
    int i = 0;
    for (i; i < 32; i++) {
        if (((target>>i) & 0x1) != ((target>>(32-1-i)) & 0x1))
            return 0;
    }

    return 1;
}

int main(void) {
    generalTestFunct test_func1 = isBinaryPanlidrome;
    
    // test 1:
    uint32_t test_num1 = 0;
    printf("%d\n", test_func1(test_num1));
    
    // test 2:
    test_num1 = 0xf000000f;
    printf("%d\n", test_func1(test_num1));
    
    // test 3:
    test_num1 = 0x11001100;
    printf("%d\n", test_func1(test_num1));

    // test 4:
    test_num1 = 0xa0000005;
    printf("%d\n", test_func1(test_num1));
    
    return 0;
}
```

## 分析

- 遍历 32 位，比较第 `i` 位与第 `(31-i)` 位是否相等。
- 全部相等则为回文，返回 1；否则返回 0。

复杂度 O(32) = O(1)。

## 相关文档
- [[reverseBits]] —— 反转比特位
- [[signessCheck]] —— 判断符号
