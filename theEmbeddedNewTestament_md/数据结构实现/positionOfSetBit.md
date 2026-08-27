---
tags: [位操作, 算法, 嵌入式]
source: Data_Struct_Implementation/BitsManipulation/positionOfSetBit
created: 2026-08-27
---

# 唯一置位的位置（Position of the only set bit）

## 代码

```c
#include<stdio.h>
#include<stdint.h>

// 任务：找到唯一置位的比特位置
// 提示：一直右移直到数字变成 0。注意下标从 -1 开始而不是 0！
typedef int (*generalTestFunct)(uint32_t target);

int setBit_pos(uint32_t target) {
    int pos = -1;
    while (target) {
        pos ++;
        target >>= 1;
    }
    
    return pos;
}

int main(void) {
    generalTestFunct test_func = setBit_pos;
    
    // test 1:
    uint32_t test1 = 0x1;
    printf("%x\n", test_func(test1));
    
    // test 2:
    uint32_t test2 = 0x8;
    printf("%x\n", test_func(test2));
    
    // test 3:
    uint32_t test3 = 0x80;
    printf("%x\n", test_func(test3));
    
    // test 4:
    uint32_t test4 = 0x10000;
    printf("%x\n", test_func(test4));
    
    return 0;
}
```

## 分析

每次将 `target` 右移 1 位，`pos` 递增。当 `target` 变成 0 时，最后一次右移前的那一位就是唯一置位所在的位置。由于 `pos` 从 `-1` 开始，最后的值即为最高置位所在的位下标。

复杂度 O(位数)，即最多 O(32)。

## 相关文档
- [[countBitsLookUpTable]] —— 统计置位个数
- [[turnoffRightmostBit]] —— 关闭最右侧置位
- [[signessCheck]] —— 判断符号
