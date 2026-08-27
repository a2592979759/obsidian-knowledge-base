---
tags: [嵌入式, 硬件, 内存映射, 字节序]
source: Data_Struct_Implementation/endianess
created: 2026-08-27
---

# 字节序判断（Check Endianness）

#### 用法
```
make
./endianess
```

## 代码

源码文件：`endianess/endianess.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

typedef enum {
    FALSE = 0,
    TRUE,
} BOOL;


BOOL is_small_endian() {
	uint32_t num = 1;
	uint8_t byte;
	
	// 检查 4 字节 uint32_t 的首个字节内存
	byte = *((uint8_t*) &num);

	return (byte == 1) ? TRUE : FALSE;
}

void mem_layout() {
	int num = 0x01234567;
	char *byte;
	int i;

	byte = (char*) &num;
	for (i = 0; i < sizeof(num); i++) {
		printf("Byte pos: %d, content: %02x\n", i, byte[i]);
	}
}

int main (int argc, char *argv[]) {

	mem_layout();

	if (is_small_endian())
		printf("Systen is of small endian!\n");
    else
		printf("Systen is of big endian!\n");

	return 0;
}
```

## 分析

- 在内存中存放多字节值（如 `uint32_t`）时，不同机器有不同的字节顺序：
  - **小端（little endian）**：最低有效字节（LSB）存放在最低地址。
  - **大端（big endian）**：最高有效字节（MSB）存放在最低地址。
- 判断方法：设 `num = 1`，取其**第一个字节**（低地址）的内存。若为 1，则是小端；若为 0，则是大端。
- `mem_layout`：打印 `0x01234567` 在内存中逐字节的布局，直观展示字节序。

```text
0x01234567
小端：67 45 23 01
大端：01 23 45 67
```

## 参考
https://www.geeksforgeeks.org/little-and-big-endian-mystery/

## 相关文档
- [[endianessSwap]] —— 字节序转换
- [[memoryMap]] —— 内存映射寄存器（嵌入式小端机器的寄存器布局）
