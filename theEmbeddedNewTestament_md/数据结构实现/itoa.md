---
tags: [字符串, 转换, 算法, 嵌入式]
source: Data_Struct_Implementation/itoa
created: 2026-08-27
---

# 整数转字符串 / itoa（多进制）

### 用法
```
make
./atoi
```

### 注意事项

这个函数实际上并不像看起来那么简单。我们需要考虑以下几点：

1. 当 base 为 10 且值为负时，需要加上负号 `-`。
2. 字符串顺序与数字顺序相反（因此需要反转）。
3. 记得用 `\0` 终结字符串。
4. 处理 `val = 0` 的情形。
5. 当 base 为 16 时，处理特殊字母 `ABCDEF`。
6. 记得在取模之前对值取绝对值，否则会得到负的余数。

### 改进点

以下实现可以从以下方面改进：

1. 处理 `INT_MIN`，当前版本无法处理。
2. 若缓冲区空间不足以容纳所有数字，会段错误。也许应该把缓冲区长度作为参数传给函数。

### 代码

源码文件：`itoa/itoa.c`

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <limits.h>

// 内联函数：交换两个数
inline static void my_swap(char *x, char *y) {
    char t = *x; *x = *y; *y = t;
}
 
// 反转 buffer[i..j]
char* reverse(char *buffer, int i, int j)
{
    while (i < j)
        my_swap(&buffer[i++], &buffer[j--]);
 
    return buffer;
}
 
// 迭代实现 C 版本的 itoa() 函数
char* itoa(int value, char* buffer, int base)
{
    // 非法输入
    if (base < 2 || base > 32)
        return buffer;
 
    // 考虑数字的绝对值
    int n = abs(value);
 
    int i = 0;
    while (n)
    {
        int r = n % base;
 
        if (r >= 10) 
            buffer[i++] = 'A' + (r - 10);
        else
            buffer[i++] = '0' + r;
 
        n = n / base;
    }
 
    // 若数字为 0
    if (i == 0)
        buffer[i++] = '0';
 
    // 若 base 为 10 且值为负，则字符串前面加负号 (-)
    // 对其它任何进制，值总是被视为无符号
    if (value < 0 && base == 10)
        buffer[i++] = '-';
 
    buffer[i] = '\0'; // 用空字符终结字符串
 
    // 反转字符串并返回
    return reverse(buffer, 0, i - 1);
}

int main(int argc, char **argv) {
    int i;
    char buffer[64];
    int buf_s = 64;

    int test_base10[6] = {0, -10, 12345, -12345, -32768, INT_MAX};
    memset(buffer, 0, buf_s);
    for (i=0; i<6; i++) 
        printf("%s ", itoa(test_base10[i], buffer, 10));
    printf("\n==============\n");  
    
    int test_base2[6] = {0, -16, 12345, -12345, -32768, INT_MAX};
    memset(buffer, 0, buf_s);
    for (i=0; i<6; i++) 
        printf("%s ", itoa(test_base2[i], buffer, 2));
    printf("\n==============\n"); 

    int test_base16[6] = {0, -10, 0x1234abcd, -12345, -32768, INT_MAX};
    memset(buffer, 0, buf_s);
    for (i=0; i<6; i++)
        printf("%s ", itoa(test_base16[i], buffer, 16)); 
    printf("\n==============\n");  
    return 0; 
}
```

## 分析
- 逐位取余得到数字，存入缓冲区（顺序是反的）。
- 结束后先补 `\0`，再反转整段字符串得到正常顺序。
- 十进制负数加负号；十六进制 `A-F` 对应余数 `10~15`。
- 与 [[atoi]] 互为逆操作。

## 相关文档
- [[atoi]] —— 字符串转整数
- [[strstr]] —— 字符串查找
