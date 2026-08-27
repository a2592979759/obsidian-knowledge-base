---
tags: [内存, 字符串, 算法, 嵌入式]
source: Data_Struct_Implementation/memcpy_memmove
created: 2026-08-27
---

# 处理重叠源与目标的安全 memcpy（Memmove）

#### 用法
```
make
./memcpy
```

## 代码

##### memcpy.h
```c
#define CPY_DIR_LOWER_TO_HIGHER 0
#define CPY_DIR_HIGHER_TO_LOWER 1
```

##### memcpy.c
```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

#include "memcpy.h"
// 标准 memmove：能安全处理源与目标重叠
void *my_memmove(void *dest, const void *src, unsigned int n)
{
    char *pcSource =(char *)src;
    char *pcDstn =(char *)dest;

    // 若 pcDstn 或 pcSource 为 NULL，直接返回
    if ((pcSource == NULL)  || (pcDstn == NULL))
    {
        return NULL;
    }

    // 判断是否重叠：源在目标之前且目标落在源区间内
    if((pcSource < pcDstn) && (pcDstn < pcSource + n))
    {
        // 从尾部向前拷贝，避免覆盖
        for (pcDstn += n, pcSource += n; n--;)
        {
            *--pcDstn = *--pcSource;
        }
    }
    else
    {
        // 不重叠，从前向后拷贝
        while(n--)
        {
            *pcDstn++ = *pcSource++;
        }
    }
    return dest;
}

void myMemcpy(void *dest, void *src, size_t n){
    uint32_t cpyDir = CPY_DIR_LOWER_TO_HIGHER;
    char *pDest = (char *) dest;
    char *pSrc = (char *) src;

    if(pDest <= (pSrc + n) && (pSrc + n) <= (pDest + n) ) {
        cpyDir = CPY_DIR_HIGHER_TO_LOWER;
    }

    if(cpyDir == CPY_DIR_LOWER_TO_HIGHER) {
        for(int i = 0; i < n; i++) {
            *pDest++ = *pSrc++;
        }

        printf("copied from lower to higher\n");
    }
    else
    {
        for(int i = n - 1; i >= 0; i--) {
            *(pDest + i) = *(pSrc + i);
        }
        printf("copied from higher to lower\n");
    }
}

int main(int argc, char **argv) {
    char csrc[] = "iLoveEmbedded"; 
    char cdest[100]; 
    myMemcpy(cdest, csrc, strlen(csrc)+1); 
    printf("[1]Copied string is %s\n\n", cdest); 

    char csrc2[] = "iLoveEmbedded    "; 
    myMemcpy(csrc2 + 3, csrc2, strlen(csrc2)+1); 
    printf("[2]Copied string is %s\n\n", csrc2 + 3); 
    
    int isrc[] = {10, 20, 30, 40, 50}; 
    int n = sizeof(isrc)/sizeof(isrc[0]); 
    int idest[n], i; 
    myMemcpy(idest, isrc,  sizeof(isrc)); 
    printf("[3]Copied array is \n"); 
    for (i=0; i<n; i++) 
        printf("%d ", idest[i]); 
    return 0; 
}
```

## 改进的性能

下面的算法来自 GNU 的 newlib 源码。若源指针与目标指针都在 4 字节边界对齐，改进后的 GNU 算法会一次拷贝 32 位（4 字节）而不是 8 位。

Listing 2：改进的 GNU 算法
```C
void * memcpy(void * dst, void const * src, size_t len)
{
    long * plDst = (long *) dst;
    long const * plSrc = (long const *) src;

    if (!(src & 0xFFFFFFFC) && !(dst & 0xFFFFFFFC))
    {
        while (len >= 4)
    {
            *plDst++ = *plSrc++;
            len -= 4;
        }
    }

    char * pcDst = (char *) plDst;
    char const * pcDst = (char const *) plSrc;

    while (len–)
    {
        *pcDst++ = *pcSrc++;
    }

    return (dst);
}
```

## 分析
- 关键点：当**目标地址位于源地址之后且落在源区间内**（即重叠且目标偏移更大）时，必须**从后向前**拷贝，否则会覆盖尚未拷贝的源数据。
- `my_memmove` / `myMemcpy` 通过判断方向来决定正向或反向拷贝，从而实现安全的 memmove 语义。
- 改进版在 4 字节对齐时用 `long` 每次拷 4 字节，显著提速。

## 参考
https://www.embedded.com/optimizing-memcpy-improves-speed/

## 相关文档
- [[alignedMalloc]] —— 对齐内存分配
- [[memoryMap]] —— 内存映射寄存器
