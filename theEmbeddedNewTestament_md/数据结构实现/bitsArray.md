---
tags: [位操作, 算法, 嵌入式, 位图]
source: Data_Struct_Implementation/bitsArray
created: 2026-08-27
---

# 位数组（Bits Array）

#### 用法
```
make
./bitsArray
```

## 代码

源码文件：`bitsArray/bitsArray.c`

```c
#include <stdio.h>

#define SetBit(A, k) (A[k/32] |= (1 << k%32))
#define ClearBit(A, k) (A[k/32] &= ~(1 << k%32))
#define TestBit(A, k) ((A[k/32] & (1 << k%32)))

int main( int argc, char* argv[] )
{
   int A[10] = {0};
   int i;

   for ( i = 0; i < 10; i++ )
      A[i] = 0;                    // 清除位数组

   printf("Set bit poistions 100, 200 and 300\n");
   SetBit( A, 100 );               // 设置 3 个位
   SetBit( A, 200 );
   SetBit( A, 300 );

   // 检查 SetBit 是否生效：
   for ( i = 0; i < 320; i++ )
      if ( TestBit(A, i) )
         printf("Bit %d was set !\n", i);

   printf("\nClear bit poistions 200 \n");
   ClearBit( A, 200 );

   // 检查 ClearBit 是否生效：
   for ( i = 0; i < 320; i++ )
      if ( TestBit(A, i) )
         printf("Bit %d was set !\n", i);
}
```

## 分析

位数组（也叫位图/bitmap）用一个 32 位 `int` 数组来存储比特。三个核心宏：

- `SetBit(A, k)`：将第 `k` 位置 1。`k/32` 定位到哪个 int，`k%32` 定位到该 int 内的比特位。
- `ClearBit(A, k)`：将第 `k` 位清 0。
- `TestBit(A, k)`：读取第 `k` 位的值。

示例中 `A[10]` 共 `10 * 32 = 320` 个比特位，所以 `i` 可以取到 0~319。

## 参考
[Array of bits introduction](http://www.mathcs.emory.edu/~cheung/Courses/255/Syllabus/1-C-intro/bit-array.html)

## 相关文档
- [[arrayOfBits]] —— 位数组概念
- [[countBitsLookUpTable]] —— 查表法统计置位个数
- [[reverseBits]] —— 反转比特位
