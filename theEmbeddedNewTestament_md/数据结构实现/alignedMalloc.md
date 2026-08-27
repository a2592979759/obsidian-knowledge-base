---
tags: [内存, 内存管理, 嵌入式]
source: Data_Struct_Implementation/alignedMalloc
created: 2026-08-27
---

# 对齐内存分配（Aligned Malloc）

#### 用法
```
make
./memalign
```

## 核心代码

源码文件：`alignedMalloc/memalign.c`

```c
#include <stdio.h>
#include <stdlib.h>

void *aligned_memory(size_t required, size_t alignment){
	void *p1;
	void **p2;

	p1 = malloc(alignment - 1 + sizeof(void *) + required);
	p2 = (void **)(((size_t)p1 + alignment - 1 + sizeof(void *)) & ~(alignment - 1));

	p2[-1] = p1;
	return p2;
}

void free_aligned(void *p) {
	free(((void **)p)[-1]);
}

int main (int argc, char *argv[]) {
	if(argc < 3) {
		printf("Wrong input\n");
		exit(1);
	}

	int required = atoi(argv[1]);
	int alignment = atoi(argv[2]);

	void *aligned_p = aligned_memory(required, alignment);

	printf("%p\n", aligned_p);

	return 0;
}
```

## 其它版本

`memalign_new.c`：

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

void *aligned_memory(size_t required, size_t alignment){

    void* ptr = (void*) malloc(required + (alignment - 1) + sizeof(void*));
    void** ret_ptr = (void**)((size_t)(ptr + sizeof(void*) + (alignment - 1)) & ~(alignment - 1));

    ret_ptr[-1] = ptr; 
    return ret_ptr;
}

void free_aligned(void *p) {
    void** _p = &p;
    
    free(_p[-1]);
}

int main (int argc, char *argv[]) {
	if(argc < 3) {
		printf("Wrong input\n");
		exit(1);
	}

	int required = atoi(argv[1]);
	int alignment = atoi(argv[2]);

	void *aligned_p = aligned_memory(required, alignment);
        void **_aligned_p = &aligned_p;

	printf("Aligned address: %p  Malloc address: %p\n", aligned_p, _aligned_p[-1]);

	return 0;
}
```

## 分析

由于 malloc 并不保证为我们对齐内存，我们需要额外两步：

1. 申请额外的字节，以便能返回一个对齐的地址。
2. 申请额外的字节，并保存原始指针与对齐指针之间的偏移量。

考虑以下情形：
- 调用 malloc 得到内存地址 X。
- 需要存储一个指针偏移量 Y，其大小固定。
- 对齐值 Z 是变量。
- 为了通用地处理，我们总是需要存储对齐偏移量。
  - 即使指针本就对齐也如此。
- 分配时，X+Y（地址+偏移大小）可能已对齐，也可能未对齐。
  - ***若 X+Y 已对齐，则无需额外字节***
  - ***若 X+Y 未对齐，最坏情况需要 Z-1 个额外字节***

示例：
- 请求对齐值 8
- malloc 返回 0xF07
- 加两个字节存偏移，到达 0xF09
- 需要 7 个额外字节，到 0xF10。

示例 #2（验证为何不需要 8）：
- 请求对齐值 8
- malloc 返回 0xF06
- 加两个字节存偏移，到达 0xF08
- 现在已按 8 字节对齐。

因此 malloc 的最坏填充为：
```C
sizeof(offset_t) + (alignment - 1)
```

对应到我们的分配：
```C
uint32_t hdr_size = PTR_OFFSET_SZ + (align - 1);
void * p = malloc(size + hdr_size);
```

## 参考
https://embeddedartistry.com/blog/2017/02/22/generating-aligned-memory/

https://codeyarns.com/tech/2017-02-28-aligned-memory-allocation.html

## 相关文档
- [[memoryPoolAllocator]] —— 内存池
- [[memcpy_memmove]] —— 内存拷贝
- [[sizeof]] —— 结构体对齐
