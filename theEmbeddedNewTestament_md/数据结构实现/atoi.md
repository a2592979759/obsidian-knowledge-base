---
tags: [字符串, 转换, 算法, 嵌入式]
source: Data_Struct_Implementation/atoi
created: 2026-08-27
---

# 支持多进制的 atoi

### 用法
```
make
./atoi
```

## 代码

源码文件：`atoi/atoi.c`

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <limits.h>

int my_atoi(char* num, int size, int base) {
    int i, ret = 0;
    int neg = 0;
    
    if (!num || size == 0)
        return ret;
    
    // 若有负号，处理负数
    if (num[0] == '-') {
        neg = 1;
        i = 1;
    } else {
        i = 0;
    }

    // 处理不同的进制
    for (i; i < size; i++) {
        ret *= base;
        if (base != 16)
            ret += num[i] - '0';
        else { // 十六进制的特殊处理
            if (num[i] >= 'a' && num[i] <= 'f')
                ret += num[i] - 'a' + 10;
            else if (num[i] >= 'A' && num[i] <= 'F')
                ret += num[i] - 'A' + 10;
            else if (num[i] >= '0' && num[i] <= '9')
                ret += num[i] - '0';
        } 
    }

    return neg ? -ret : ret;
}

int main(int argc, char **argv) {
    int ret;

    char* test_base10 = "123456";
    printf("my: %d expect: %d", my_atoi(test_base10, strlen(test_base10), 10), atoi(test_base10));
    printf("\n==============\n");  
    
    test_base10 = "-123456";
    printf("my: %d expect: %d ", my_atoi(test_base10, strlen(test_base10), 10), atoi(test_base10));
    printf("\n==============\n");  

    char *test_base2 = "1000";
    printf("%d ", my_atoi(test_base2, strlen(test_base2), 2));
    printf("\n==============\n");

    test_base2 = "111";
    printf("%d ", my_atoi(test_base2, strlen(test_base2), 2));
    printf("\n==============\n");

    char *test_base16 = "1a";
    printf("%d ", my_atoi(test_base16, strlen(test_base16), 16));
    printf("\n==============\n");

    test_base16 = "100";
    printf("%d ", my_atoi(test_base16, strlen(test_base16), 16));
    printf("\n==============\n");

    test_base16 = "ff";
    printf("%d ", my_atoi(test_base16, strlen(test_base16), 16));
    printf("\n==============\n");

    test_base16 = "1A";
    printf("%d ", my_atoi(test_base16, strlen(test_base16), 16));
    printf("\n==============\n");

    test_base16 = "FF";
    printf("%d ", my_atoi(test_base16, strlen(test_base16), 16));
    printf("\n==============\n");
    return 0; 
}
```

## 分析

- 标准 `atoi` 只支持十进制。本实现通过 `base` 参数支持任意进制（2、10、16 等）。
- 处理负号：若首字符为 `-`，置 `neg` 标志并跳过。
- 十六进制需要区分 `a-f`/`A-F` 与数字字符。
- 结果由 `ret = ret * base + digit` 逐位累加得到。

## 相关文档
- [[itoa]] —— 整数转字符串（atoi 的逆操作）
- [[strstr]] —— 字符串查找
