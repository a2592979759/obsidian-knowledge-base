---
tags: [字符串, 算法, 嵌入式]
source: Data_Struct_Implementation/strstr
created: 2026-08-27
---

# StrStr（子串查找）

#### 用法
```
make
./strstr
```

## 代码

源码文件：`strstr/strstr.c`

```c
#include <stdio.h>
 
// 判断 X 和 Y 是否相同
int compare(const char *X, const char *Y)
{
    while (*X && *Y)
    {
        if (*X != *Y)
            return 0;
 
        X++;
        Y++;
    }
 
    return (*Y == '\0');
}
 
// 实现 strstr() 函数
const char* strstr(const char* X, const char* Y)
{
    while (*X != '\0')
    {
        if ((*X == *Y) && compare(X, Y))
            return X;
        X++;
    }
 
    return NULL;
}
 
// 在 C 中实现 strstr 函数
int main()
{
    char *X = "Techie Delight - Coding made easy";
    char *Y = "Coding";
 
    printf("%s\n", strstr(X, Y));
 
    return 0;
}
```

## 分析

- 在文本串 `X` 中逐个字符扫描，当遇到与模式串 `Y` 首字符相同的字符时，调用 `compare` 完整比较后续字符。
- 若完全匹配，返回该位置的指针；否则继续扫描，返回 NULL。

复杂度最坏 O(|X|·|Y|)。

## 相关文档
- [[atoi]] —— 字符串转整数
- [[itoa]] —— 整数转字符串
- 参考：https://www.techiedelight.com/implement-strstr-function-c-iterative-recursive/
