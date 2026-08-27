---
tags: [数据结构, 算法, 嵌入式, 内存池, 内存管理]
source: Data_Struct_Implementation/memoryPoolAllocator
created: 2026-08-27
---

# 通用内存池实现

> 参考：[A generic C memory pool implementation](https://github.com/jobtalle/pool)

源码文件：`memoryPoolAllocator/pool.h`、`pool.c`、`test.c`

## 重要的数据结构

```c
/*
这是链表数据结构，用于记录内存池中的空闲内存块。
*/
typedef struct poolFreed{
	struct poolFreed *nextFree;
} poolFreed;

/*
内存池数据结构，包含关于内存池的元数据，
并维护一个待分配的空闲块链表。
*/
typedef struct {
	uint32_t elementSize; // 每个 chunk 的大小
	uint32_t blockSize;   // 每个 block 的大小
	uint32_t used;        // 当前 block 中已使用的 chunk 数
	int32_t block;        // 当前 block 的下标
	poolFreed *freed;     // 全局空闲 chunk 链表
	uint32_t blocksUsed;  // 已分配的 block 数
	uint8_t **blocks;     // block 内存
} pool;
```

> 注：`elementSize` 即 README 中 `chunkSize` 的别称。

## 内存池实现

`pool.h`：

```c
#pragma once

#include <stdint.h>

#define POOL_BLOCKS_INITIAL 1

typedef struct poolFreed{
	struct poolFreed *nextFree;
} poolFreed;

typedef struct {
	uint32_t elementSize;
	uint32_t blockSize;
	uint32_t used;
	int32_t block;
	poolFreed *freed;
	uint32_t blocksUsed;
	uint8_t **blocks;
} pool;

void poolInitialize(pool *p, const uint32_t elementSize, const uint32_t blockSize);
void poolFreePool(pool *p);

#ifndef DISABLE_MEMORY_POOLING
void *poolMalloc(pool *p);
void poolFree(pool *p, void *ptr);
#else
#include <stdlib.h>
#define poolMalloc(p) malloc((p)->blockSize)
#define poolFree(p, d) free(d)
#endif
void poolFreeAll(pool *p);
```

`pool.c`：

```c
#include <string.h>
#include <stdlib.h>

#include "pool.h"

#ifndef max
#define max(a,b) ((a)<(b)?(b):(a))
#endif

void poolInitialize(pool *p, const uint32_t elementSize, const uint32_t blockSize)
{
	uint32_t i;

	p->elementSize = max(elementSize, sizeof(poolFreed));
	p->blockSize = blockSize;
	
	poolFreeAll(p);

	p->blocksUsed = POOL_BLOCKS_INITIAL;
	p->blocks = malloc(sizeof(uint8_t*)* p->blocksUsed);

	for(i = 0; i < p->blocksUsed; ++i)
		p->blocks[i] = NULL;
}

void poolFreePool(pool *p)
{
	uint32_t i;
	for(i = 0; i < p->blocksUsed; ++i) {
		if(p->blocks[i] == NULL)
			break;
		else
			free(p->blocks[i]);
	}

	free(p->blocks);
}

#ifndef DISABLE_MEMORY_POOLING
void *poolMalloc(pool *p)
{
	if(p->freed != NULL) {
		void *recycle = p->freed;
		p->freed = p->freed->nextFree;
		return recycle;
	}

	if(++p->used == p->blockSize) {
		p->used = 0;
		if(++p->block == (int32_t)p->blocksUsed) {
			uint32_t i;

			p->blocksUsed <<= 1;
			p->blocks = realloc(p->blocks, sizeof(uint8_t*)* p->blocksUsed);

			for(i = p->blocksUsed >> 1; i < p->blocksUsed; ++i)
				p->blocks[i] = NULL;
		}

		if(p->blocks[p->block] == NULL)
			p->blocks[p->block] = malloc(p->elementSize * p->blockSize);
	}
	
	return p->blocks[p->block] + p->used * p->elementSize;
}

void poolFree(pool *p, void *ptr)
{
	poolFreed *pFreed = p->freed;

	p->freed = ptr;
	p->freed->nextFree = pFreed;
}
#endif

void poolFreeAll(pool *p)
{
	p->used = p->blockSize - 1;
	p->block = -1;
	p->freed = NULL;
}
```

`test.c`：

```c
#include <stdio.h>
#include <stdlib.h>
#include "pool.h"

#define SUCCESS 0
#define FAILURE -1

int  test_pool(int element_size, int block_size)
{
	pool pool_ptr;
	int *test_ptr1 = NULL;
	int *test_ptr2 = NULL;
	
	/* 用给定参数初始化内存池 */
	poolInitialize(&pool_ptr, element_size, block_size);
	
	/* 从内存池分配内存 */
	test_ptr1 = poolMalloc(&pool_ptr);
	test_ptr2 = poolMalloc(&pool_ptr);
	
	/* 测试分配的内存有效性 */
	if(!(test_ptr1 && test_ptr2))
	{
		printf("memory allocation failure \n");
		
		return FAILURE;
	}
	
	/* 释放从内存池分配的内存 */
	poolFree(&pool_ptr, test_ptr1);
	
	/* 释放该内存池的所有内存 */
	poolFreePool(&pool_ptr);
	
	return SUCCESS;
}

int main()
{
	if(test_pool(4, 8) == FAILURE)
		printf("test_pool failure %s %d \n", __FILE__, __LINE__);
	
	if(test_pool(8, 8) == FAILURE)
		printf("test_pool failure %s %d \n", __FILE__, __LINE__);
	
	if(test_pool(16, 8) == FAILURE)
		printf("test_pool failure %s %d \n", __FILE__, __LINE__);

	if(test_pool(32, 8) == FAILURE)
		printf("test_pool failure %s %d \n", __FILE__, __LINE__);
  

	return SUCCESS;
}
```

## 进阶阅读

[Writing a Pool Allocator I](http://dmitrysoshnikov.com/compilers/writing-a-memory-allocator/)

[Writing a Pool Allocator II](http://dmitrysoshnikov.com/compilers/writing-a-pool-allocator/)

## 相关文档
- [[alignedMalloc]] —— 对齐内存分配
- [[hashTable]] —— 哈希表
