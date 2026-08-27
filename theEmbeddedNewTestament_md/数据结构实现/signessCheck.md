---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/signessCheck
created: 2026-08-27
---

# 判断两个整数符号是否相反（Detect if two integers have opposite signs）

给定两个有符号整数，编写一个函数：若给定整数的符号不同则返回 true，否则返回 false。例如，-1 和 +100 应返回 true，-100 和 -200 应返回 false。函数**不能使用任何算术运算符**。

## 代码

```c++
#include<stdio.h>
#include<stdint.h>

// 任务：找到唯一置位的位置（注释与实现不符，见下方分析）
typedef int (*generalTestFunct)(int target, int target2);

int checkSignDiff(int num1, int num2) {
    return (num1 ^ num2) < 0;
}

int main(void) {
    generalTestFunct test_func = checkSignDiff;
    
    // test 1:
    int test_num1 = 1;
    int test_num2 = -1;
    printf("%x\n", test_func(test_num1, test_num2));
    
    // test 2:
    test_num1 = 1;
    test_num2 = 1;
    printf("%x\n", test_func(test_num1, test_num2));
    
    // test 3:
    test_num1 = 2121312;
    test_num2 = 2121;
    printf("%x\n", test_func(test_num1, test_num2));
    
    // test 4:
    test_num1 = -213;
    test_num2 = 21443212;
    printf("%x\n", test_func(test_num1, test_num2));
    
    return 0;
}
```

## 分析

- 判断两个数符号是否相反，只需看它们的**符号位（最高位）是否不同**。
- `num1 ^ num2`：若二者符号位不同，结果的最高位为 1，作为有符号数即小于 0；若符号位相同，结果为 0/正。
- 因此 `(num1 ^ num2) < 0` 即为答案。

复杂度 O(1)，只用位运算，不用算术运算符。

## 相关文档
- [[singleNumber]] —— 异或位操作
- [[reverseBits]] —— 位操作技巧
