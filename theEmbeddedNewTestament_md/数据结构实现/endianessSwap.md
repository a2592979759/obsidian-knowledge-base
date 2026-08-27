---
tags: [嵌入式, 硬件, 字节序]
source: Data_Struct_Implementation/endianessSwap
created: 2026-08-27
---

# 字节序转换（Endianness Conversion）

#### 用法
```
make
./endianess
```

## 代码

源码文件：`endianessSwap/endianess.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

uint32_t endianess_swap(uint32_t num) {
    num = ((num & 0xffff0000) >> 16) | ((num & 0x0000ffff) << 16);
    num = ((num & 0xff00ff00) >> 8) | ((num & 0x00ff00ff) << 8);
    return num;
}

int main (int argc, char *argv[]) {

	uint32_t num = 0x12345678;
    uint8_t *n = (uint8_t*) &num;
    int i;

    for (i = 0; i < sizeof(num); i++) {
        printf("%x ", *n++);
    }
    printf("\n");

    uint32_t swap_num = endianess_swap(num);
    n = (uint8_t*) &swap_num;

    for (i = 0; i < sizeof(swap_num); i++) {
        printf("%x ", *n++);
    }
    printf("\n");

	return 0;
}

// 另一种反转 32 位比特的写法（LeetCode）
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        n = (n << 16) | (n >> 16);
        n = ((n << 8) & 0xff00ff00) | ((n >> 8) & 0x00ff00ff);
        n = ((n << 4) & 0xf0f0f0f0) | ((n >> 4) & 0x0f0f0f0f);
        n = ((n << 2) & 0xcccccccc) | ((n >> 2) & 0x33333333);
        n = ((n << 1) & 0xaaaaaaaa) | ((n >> 1) & 0x55555555);
        
        return n;
    }
};
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        uint32_t ret{};
        int m = 32;
        
        while(m --) {
            ret <<= 1;
            ret |= (n & 0x1);
            n >>= 1;
        }
        
        return ret;
    }
};
```

## 分析

`endianess_swap` 通过分治交换字节序：
1. 交换高 16 位与低 16 位。
2. 再在每个 16 位内交换高 8 位与低 8 位。

从而把字节序大端 ↔ 小端互换，例如 `0x12345678` 变成 `0x78563412`。附加的两段 C++ 代码是 LeetCode 上反转 32 位比特的不同实现（分治式与逐位循环式）。

## 参考
https://www.geeksforgeeks.org/little-and-big-endian-mystery/

## 相关文档
- [[endianess]] —— 字节序判断
- [[reverseBits]] —— 反转比特位
- [[memoryMap]] —— 内存映射寄存器
