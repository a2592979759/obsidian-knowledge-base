---
tags: [内存, 嵌入式, 结构体对齐]
source: Data_Struct_Implementation/sizeof
created: 2026-08-27
---

# 自定义 sizeof 实现与结构体填充

#### 用法
```
make
./stack
```

## 代码

源码文件：`sizeof/sizeof.c`

```C
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>

#define my_sizeof(type) (char*)(&type+1)-(char*)(&type) 

/* 假设这是 32 位机器，因此要求字（4 字节）对齐。 */

// 结构体大小 12 (3*sizeof(int))，p 表示填充字节
typedef struct Random_item {
    char a; // 1
            // 3p
    int b;  // 4 -> 主导项
    char c; // 1
            // 3p 以满足处理器字对齐要求
} Random_item;

// 结构体大小 12
typedef struct Random_item2 {
    char a;    // 1
               // 3
    int b;     // 4 -> 主导项
    char c;    // 1
    char d;    // 1
    uint16_t e; // 2
} Random_item2;

// 结构体大小 24 (3*sizeof(long int))
typedef struct Random_item3 {
    int b;      // 4
    char d;     // 1
                // 3p
    long int a; // 8 -> 若 long int 为 8 字节，则是 64 位机器
    int c;      // 4
    uint16_t f; // 2
                // 2p 
} Random_item3;

// 结构体大小 32 (4*sizeof(long int))
typedef struct Random_item4 {
    int b;        // 4
    char d;       // 1
                  // 3p
    long int a;   // 8 -> 若 long int 为 8 字节，则是 64 位机器
    int c;        // 4
    uint16_t f;   // 2
    char e;       // 1
    char g;       // 1
    char h;       // 1
                  // 7p -> 与更早的 8 字节 long int 对齐
} Random_item4;

// 结构体大小 56
typedef struct Random_item5 {
    int b;        // 4
    char d;       // 1
                  // 3p
    long int a;   // 8 
    int c;        // 4
    uint16_t f;   // 2
    char e;       // 1
                  // 1p
    Random_item3 random; // 24  <--- 需要把结构体拆开，否则很难判断中间填充
    char g; // 1
    char h; // 1
            // 6p -> 与更早的 8 字节 long int 对齐
} Random_item5;

// 应输出 4 或 -4
void argument_alignment_check( char c1, char c2 ) 
{ 
   // 考虑向下增长的栈
   // (在向上增长的栈上，输出为负)
   printf("Displacement %ld\n", (char*)&c2 - (char*)&c1); 
} 

int main() {
    Random_item tmp;
    Random_item2 tmp2;
    Random_item3 tmp3;

    printf("my_sizeof the item is %ld sizeof: %ld\n", my_sizeof(tmp), sizeof(tmp));
    printf("my_sizeof the item is %ld sizeof: %ld\n", my_sizeof(tmp2), sizeof(tmp2));
    printf("my_sizeof the item is %ld sizeof: %ld\n", my_sizeof(tmp3), sizeof(tmp3));

    argument_alignment_check('a', 'b');

    return 0;
}
```

## 练习：预测以下程序的输出

```C
#include <stdio.h> 
  
// 对齐要求
// (典型 32 位机器)
  
// char         1 字节
// short int    2 字节
// int          4 字节
// double       8 字节
  
// 结构体 A 
typedef struct structa_tag 
{ 
   char        c; 
   short int   s; 
} structa_t; 
  
// 结构体 B 
typedef struct structb_tag 
{ 
   short int   s; 
   char        c; 
   int         i; 
} structb_t; 
  
// 结构体 C 
typedef struct structc_tag 
{ 
   char        c; 
   double      d; 
   int         s; 
} structc_t; 
  
// 结构体 D 
typedef struct structd_tag 
{ 
   double      d; 
   int         s; 
   char        c; 
} structd_t; 
  
int main() 
{ 
   printf("sizeof(structa_t) = %d\n", sizeof(structa_t)); 
   printf("sizeof(structb_t) = %d\n", sizeof(structb_t)); 
   printf("sizeof(structc_t) = %d\n", sizeof(structc_t)); 
   printf("sizeof(structd_t) = %d\n", sizeof(structd_t)); 
  
   return 0; 
} 
```

答案：
```C
sizeof(structa_t) = 4
sizeof(structb_t) = 8
sizeof(structc_t) = 24
sizeof(structd_t) = 16
```

## 分析

- `my_sizeof(type)`：`(char*)(&type+1) - (char*)(&type)`，取 `type` 相邻两个同类型对象地址之差，即占用字节数。
- 结构体对齐规则：成员的偏移量必须是其自身大小的整数倍；结构体整体大小必须是最大成员对齐值的整数倍。
- 通常将较大的成员（如 `double`、`long int`）放在前面能减少填充字节、缩小结构体总大小。

## 参考
https://www.geeksforgeeks.org/implement-your-own-sizeof/

https://www.geeksforgeeks.org/structure-member-alignment-padding-and-data-packing/

## 相关文档
- [[alignedMalloc]] —— 对齐内存分配
- [[memoryMap]] —— 内存映射寄存器（考虑对齐的实践）
