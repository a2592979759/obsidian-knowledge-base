---
tags: [数据结构, 算法, 嵌入式, 哈希表]
source: Data_Struct_Implementation/hashTable
created: 2026-08-27
---

# 哈希表（Hash Table）

## 线性探测法哈希表（Linear Probing）

### 分析

**线性探测法**也许是最简单的一种解决冲突的方式。它通过将值插入到哈希函数给出的散列下标之后的下一个空闲位置来解决冲突。当值恰好落在不同的下标时，效果很好；但当**簇（cluster）**形成后，性能会急剧下降。

### 用法
```
make
./hashTable
```

### 代码

源码文件：`hashTable/hashTable.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>

#define SIZE 20

struct DataItem {
   int data;   
   int key;
};

struct DataItem* hashArray[SIZE]; 
struct DataItem* dummyItem;
struct DataItem* item;

int hashCode(int key) {
   return key % SIZE;
}

struct DataItem *search(int key) {
   // 获取哈希下标
   int hashIndex = hashCode(key);  
	
   // 在数组中移动直到遇到空槽
   while(hashArray[hashIndex] != NULL) {
	
      if(hashArray[hashIndex]->key == key)
         return hashArray[hashIndex]; 
				
      // 移到下一个槽
      ++hashIndex;
		
      // 回绕到表头
      hashIndex %= SIZE;
   }        
	
   return NULL;        
}

void insert(int key,int data) {

   struct DataItem *item = (struct DataItem*) malloc(sizeof(struct DataItem));
   item->data = data;  
   item->key = key;

   // 获取哈希下标
   int hashIndex = hashCode(key);

   // 在数组中移动直到遇到空槽或已删除槽
   while(hashArray[hashIndex] != NULL && hashArray[hashIndex]->key != -1) {
      // 移到下一个槽
      ++hashIndex;
		
      // 回绕到表头
      hashIndex %= SIZE;
   }
	
   hashArray[hashIndex] = item;
}

struct DataItem* delete(struct DataItem* item) {
   int key = item->key;

   // 获取哈希下标
   int hashIndex = hashCode(key);

   // 在数组中移动直到遇到空槽
   while(hashArray[hashIndex] != NULL) {
	
      if(hashArray[hashIndex]->key == key) {
         struct DataItem* temp = hashArray[hashIndex]; 
			
         // 在删除的位置放一个哑元（dummy item）
         hashArray[hashIndex] = dummyItem; 
         return temp;
      }
		
      // 移到下一个槽
      ++hashIndex;
		
      // 回绕到表头
      hashIndex %= SIZE;
   }      
	
   return NULL;        
}

void display() {
   int i = 0;
	
   for(i = 0; i<SIZE; i++) {
	
      if(hashArray[i] != NULL)
         printf(" (%d,%d)",hashArray[i]->key,hashArray[i]->data);
      else
         printf(" ~~ ");
   }
	
   printf("\n");
}

int main() {
   dummyItem = (struct DataItem*) malloc(sizeof(struct DataItem));
   dummyItem->data = -1;  
   dummyItem->key = -1; 

   insert(1, 20);
   insert(2, 70);
   insert(42, 80);
   insert(4, 25);
   insert(12, 44);
   insert(14, 32);
   insert(17, 11);
   insert(13, 78);
   insert(37, 97);

   display();
   item = search(37);

   if(item != NULL) {
      printf("Element found: %d\n", item->data);
   } else {
      printf("Element not found\n");
   }

   delete(item);
   item = search(37);

   if(item != NULL) {
      printf("Element found: %d\n", item->data);
   } else {
      printf("Element not found\n");
   }
}
```

## 链地址法哈希表（Chaining）

### 分析

**链地址法**。这是最简单的一种方式。每个下标都是它自己的一条链表，因此哈希下标相同的值会被放在同一条链表上。同样，当许多值落入同一个键时性能会提升，但不会像线性探测那样形成糟糕的簇。这意味着不会出现其它键的值散布其中，而是只在特定键的链表内查找。因此，性能只会受那些值很多的键影响，其余的键继续正常工作！

### 用法
```
make hashTable_chain
./hashTable_chain
```

### 代码

源码文件：`hashTable/hashTable_chain.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define SIZE 20

typedef struct DataItem {
   int data;   
   int key;
   struct DataItem *next;
} DataItem, *pDataItem;

pDataItem hashArray[SIZE]; 
pDataItem dummyItem;
pDataItem item;

int hashCode(int key) {
   return key % SIZE;
}

pDataItem search(int key) {
   // 获取哈希下标
   int hashIndex = hashCode(key);  
   pDataItem dummy;

   dummy = hashArray[hashIndex];
   while(dummy) {
      dummy = hashArray[hashIndex];

      while (dummy) {
        if (dummy->key == key)
            return dummy; 
	      dummy = dummy->next;
      }	
   }        
	
   return NULL;        
}

void insert(int key,int data) {
   pDataItem dummy;
   pDataItem item;

   if (search(key)) {
      printf("Item with the same key %d already exist!\n", key);
      return;
   }
   
   item = (pDataItem) malloc(sizeof(DataItem));
   item->data = data;  
   item->key = key;
   item->next = NULL;

   // 获取哈希下标
   int hashIndex = hashCode(key);
   dummy = hashArray[hashIndex];

   // 若该链为空
   if (!dummy) {
       hashArray[hashIndex] = item;
       return;
   }

   // 移动到链尾
   while(dummy && dummy->next) {
      // 移到下一个
      dummy = dummy->next;
   }
   dummy->next = item;
}

void delete(int key) {
   pDataItem dummy;
   pDataItem dummy2;

   int hashIndex = hashCode(key);

   dummy = hashArray[hashIndex];

   // 若键在链头
   if (dummy && hashArray[hashIndex]->key == key) {
      dummy = hashArray[hashIndex];
      hashArray[hashIndex] = dummy->next;
      free(dummy);
      return;
   }
   
   // 若键在链中或链尾
   while(dummy && dummy->next) {

      // 若键在链中间
      if(dummy->key == key) {
         dummy2 = dummy->next;
         *dummy = *(dummy->next);
         free(dummy2);
         return;
      }

      // 若键在链尾
      if (!dummy->next->next && dummy->next->key == key) {
         dummy2 = dummy->next;
         dummy->next = NULL;
         free(dummy2);
         return;
      }

      dummy = dummy->next;
   }

   // 若键未找到
}

void display() {
   int i = 0;
   pDataItem dummy;
	
   printf("===================\n");
   for(i = 0; i<SIZE; i++) {
      dummy = hashArray[i];
      while (dummy) {
         printf(" (%d,%d)",dummy->key,dummy->data);
         dummy = dummy->next;
      }
      
      if (!dummy)
         printf(" ~~ ");
      
      printf("\n");
   }
	
    printf("===================\n");
}

static void check_item(int key) {
   pDataItem item;
   
   item = search(key);

   if(item != NULL) {
      printf("Element found: %d\n", item->data);
   } else {
      printf("Element with key %d not found\n", key);
   }
}

int main() {
   pDataItem item;

   insert(1, 20);
   insert(2, 70);
   insert(42, 80);
   insert(4, 25);
   insert(12, 44);
   insert(14, 32);
   insert(17, 11);
   insert(13, 78);
   insert(37, 97);
   insert(107, 27);
   insert(57, 47);

   // 检查哈希表并测试查找
   display();
   check_item(17);
   check_item(37);

   // 测试删除并查找一个不存在的项
   delete(37);
   check_item(37);
   check_item(17);
   display();

   // 删除第一个项
   insert(77, 438);
   insert(97, 438);
   delete(17);
   display();

   // 删除最后一个项
   delete(97);
   display();
   insert(97, 338);
   display();
}
```

## 复杂度
- 平均查找 / 插入 / 删除：O(1)
- 最坏查找：O(n)（线性探测出现大量簇；链地址法某条链过长）

## 参考
https://www.tutorialspoint.com/data_structures_algorithms/hash_table_program_in_c.htm

https://steemit.com/programming/@drifter1/programming-c-hashtables-with-chaining

## 相关文档
- [[BST]] —— 二叉搜索树
- [[memoryPoolAllocator]] —— 内存池
