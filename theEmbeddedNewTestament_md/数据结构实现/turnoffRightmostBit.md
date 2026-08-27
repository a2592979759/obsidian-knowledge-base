---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/turnoffRightmostBit
created: 2026-08-27
---

# 关闭最右侧置位（Turn off the rightmost set bit）

编写程序，取消设置一个整数**最右侧的那个置位**。

## 代码

```c
#include<stdio.h>
#include<stdint.h>

// 任务：关闭最右侧置位
typedef uint32_t (*generalTestFunct)(uint32_t target);

uint32_t rightmostOff(uint32_t target) {
    return target & (target - 1);
}

int main(void) {
    generalTestFunct test_func = rightmostOff;
    
    // test 1:
    uint32_t test_num1 = 1;
    printf("%x\n", test_func(test_num1));
    
    // test 2:
    test_num1 = 0x1000;
    printf("%x\n", test_func(test_num1));
    
    // test 3:
    test_num1 = 0x1100;
    printf("%x\n", test_func(test_num1));
    
    // test 4:
    test_num1 = 0x100010;
    printf("%x\n", test_func(test_num1));
    
    return 0;
}
```

## 分析

表达式 `x & (x - 1)` 是位操作中的经典技巧：
- `x - 1` 会把最右侧的 `1` 变成 `0`，并把该位更低位全部变为 `1`。
- 与 `x` 相与后，最右侧的置位被清除，其余位保持不变。

复杂度 O(1)。该技巧也常用于判断一个数是否为 2 的幂（结果为 0 即是），以及统计置位个数（`flipBitsNumber`）。

## 相关文档
- [[flipBitsNumber]] —— 统计需要翻转的位（利用 `x & (x-1)` 循环）
- [[positionOfSetBit]] —— 唯一置位的位置
