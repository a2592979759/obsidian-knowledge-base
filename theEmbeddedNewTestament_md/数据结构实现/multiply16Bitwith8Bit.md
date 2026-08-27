---
tags: [数学, 算法, 嵌入式, 乘法]
source: Data_Struct_Implementation/Math/Multiply16Bitwith8Bit
created: 2026-08-27
---

# 用 8 位乘法器实现 16 位乘法（Multiply 16-bit number with 8-bit multiplier）

乘法展开示意：

```
                                       [mHigh mLow]    X
                                       [nHigh nLow]
                                       —————————————
                        [mHigh * nLow] [mLow * nLow]
        [mHigh * nHigh] [mLow * nHigh]
    ——————————————————————————————————————————————————————————————
    [mHigh * nHigh] + [mLow * nHigh + mHigh * nLow] + [mLow * nLow]
    ——————————————————————————————————————————————————————————————
```

## 代码

```c
#include<stdio.h>
#include<stdint.h>

typedef uint32_t (*generalTestFunct)(uint16_t AB, uint16_t CD);

uint16_t mult_8_bit(uint8_t A, uint8_t B) {
    return (uint16_t)(A * B);
} 

uint32_t mult_16_bit2(uint16_t m, uint16_t n)
{
    uint8_t mLow = (m & 0x00FF);              // 取 `m` 的低 8 位
    uint8_t mHigh = (m & 0xFF00) >> 8;        // 取 `m` 的高 8 位
    uint8_t nLow = (n & 0x00FF);              // 取 `n` 的低 8 位
    uint8_t nHigh = (n & 0xFF00) >> 8;        // 取 `n` 的高 8 位
 
    uint16_t mLow_nLow = mult_8_bit(mLow, nLow);
    uint16_t mHigh_nLow = mult_8_bit(mHigh, nLow);
    uint16_t mLow_nHigh = mult_8_bit(mLow, nHigh);
    uint16_t mHigh_nHigh = mult_8_bit(mHigh, nHigh);
 
    // 返回 32 位结果（别忘了把 `mHigh_nLow` 和 `mLow_nHigh` 左移 1 字节，
    // 把 `mHigh_nHigh` 左移 2 字节）
    return mLow_nLow + ((uint32_t)(mHigh_nLow + mLow_nHigh) << 8) + ((uint32_t)mHigh_nHigh << 16);
}

int main(void) {
    generalTestFunct test_func1 = mult_16_bit2;
    
    // test 1:
    uint16_t test_num1 = 0xf;
    uint16_t test_num2 = 0x3;
    printf("Expect: %x Actual: %x \n", (uint32_t) test_num1*test_num2, test_func1(test_num1, test_num2));
    
    // test 2:
    test_num1 = 0x1ff;
    test_num2 = 0x1ff;
    printf("Expect: %x Actual: %x \n", (uint32_t) test_num1*test_num2, test_func1(test_num1, test_num2));
    
    // test 3:
    test_num1 = 0xffff;
    test_num2 = 0xffff;
    printf("Expect: %x Actual: %x \n", (uint32_t) test_num1*test_num2, test_func1(test_num1, test_num2));

    // test 4:
    test_num1 = 0x1fff;
    test_num2 = 0x3;
    printf("Expect: %x Actual: %x \n", (uint32_t) test_num1*test_num2, test_func1(test_num1, test_num2));
    
    return 0;
}
```

## 分析

把两个 16 位数分别拆成高 8 位 `mHigh/nHigh` 和低 8 位 `mLow/nLow`：

`m * n = (mHigh<<8 + mLow) * (nHigh<<8 + nLow)`

展开为四项 8 位 × 8 位乘法（结果最多 16 位）：

- `mLow * nLow`（权 2^0）
- `mHigh * nLow + mLow * nHigh`（权 2^8）
- `mHigh * nHigh`（权 2^16）

最后按权重移位相加，得到 32 位结果。这在某些只有 8 位乘法器的 MCU 上非常有用。

### 参考

[multiply-16-bit-integers-using-8-bit-multiplier](https://www.techiedelight.com/multiply-16-bit-integers-using-8-bit-multiplier/)

## 相关文档
- [[sizeof]] —— 位数/类型大小
- [[endianess]] —— 字节序（影响多字节拆分）
