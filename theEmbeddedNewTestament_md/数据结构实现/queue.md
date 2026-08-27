---
tags: [数据结构, 算法, 嵌入式, 队列]
source: Data_Struct_Implementation/queue
created: 2026-08-27
---

# 队列（链表实现）

#### 用法
```
make
./queue
```

## 代码（链表实现）

```c
// 基于链表的队列实现演示程序
#include <stdio.h> 
#include <stdlib.h> 
  
// 用于存储队列元素的一个链表节点
struct QNode { 
    int key; 
    struct QNode* next; 
}; 
  
// 队列：front 存链表头节点，rear 存链表尾节点
struct Queue { 
    struct QNode *front, *rear; 
}; 
  
// 创建一个新的链表节点
struct QNode* newNode(int k) 
{ 
    struct QNode* temp = (struct QNode*)malloc(sizeof(struct QNode)); 
    temp->key = k; 
    temp->next = NULL; 
    return temp; 
} 
  
// 创建一个空队列
struct Queue* createQueue() 
{ 
    struct Queue* q = (struct Queue*)malloc(sizeof(struct Queue)); 
    q->front = q->rear = NULL; 
    return q; 
} 
  
// 入队
void enQueue(struct Queue* q, int k) 
{ 
    // 创建新链表节点
    struct QNode* temp = newNode(k); 
  
    // 若队列为空，新节点同时作为 front 和 rear
    if (q->rear == NULL) { 
        q->front = q->rear = temp; 
        return; 
    } 
  
    // 将新节点接到队尾，并更新 rear
    q->rear->next = temp; 
    q->rear = temp; 
} 
  
// 出队
void deQueue(struct Queue* q) 
{ 
    // 若队列为空，返回
    if (q->front == NULL) 
        return; 
  
    // 保存原 front，front 后移一个节点
    struct QNode* temp = q->front; 
  
    q->front = q->front->next; 
  
    // 若 front 变为 NULL，rear 也置为 NULL
    if (q->front == NULL) 
        q->rear = NULL; 
  
    free(temp); 
} 
  
// 主程序测试上述函数
int main() 
{ 
    struct Queue* q = createQueue(); 
    enQueue(q, 10); 
    enQueue(q, 20); 
    deQueue(q); 
    deQueue(q); 
    enQueue(q, 30); 
    enQueue(q, 40); 
    enQueue(q, 50); 
    deQueue(q); 
    printf("Queue Front : %d \n", q->front->key); 
    printf("Queue Rear : %d\n", q->rear->key); 
    return 0; 
} 
```

## 进阶队列（支持 front()、back()、empty() 方法）

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct Qnode {
    int val;
    struct Qnode* next;
} Qnode;

typedef struct Queue {
    int size;
    int cap;
    Qnode *head;
    Qnode *tail;
} Queue;

Queue* create_Q (int size) {
    if (size <= 0)
        return NULL;

    Queue* new_Q = (Queue*) malloc(sizeof(Queue));
    if (!new_Q)
        return NULL;

    new_Q->cap = size;
    new_Q->size = 0;
    new_Q->head = new_Q->tail = NULL;
}

int pushQ(Queue* queue, int val) {
    if (!queue)
        return -1;
    
    if (queue->size >= queue->cap) {
        printf("Queue full! cannot push more!\n");
        return -1;
    }

    Qnode* new_node = (Qnode*) malloc(sizeof(Qnode));
    new_node->val = val;
    new_node->next = NULL;

    queue->size ++;

    if (queue->size == 1) {
        queue->head = queue->tail = new_node;
    } else {
        queue->tail->next = new_node;
        queue->tail = new_node;
    }

    return 0;
}

Qnode* front(Queue* queue) {
    if (!queue)
        return NULL;

    return queue->head;
}

Qnode* back(Queue* queue) {
    if (!queue)
        return NULL;

    return queue->tail;
}

void pop(Queue* queue) {
    Qnode* tmp; 
    
    if (!queue)
        return;
    
    tmp = queue->head;
    queue->head = tmp->next;

    free(tmp);
    queue->size--;
}

int is_empty(Queue* queue) {
    if (!queue)
        return -1;

    return queue->size == 0 ? 1 : 0;
}

int size(Queue* queue) {
    if (!queue)
        return -1;

    return queue->size;
}

int main(int argc, int** argv) {
    Queue *new_queue = create_Q(10); 
    Qnode *new_node;

    pushQ(new_queue, 1);
    pushQ(new_queue, 2);
    pushQ(new_queue, 3);
    pushQ(new_queue, 4);
    pushQ(new_queue, 5);
    pushQ(new_queue, 6);

    new_node = front(new_queue);
    printf("Front val: %d\n", new_node->val);

    new_node = back(new_queue);
    printf("Back val: %d size: %d\n", new_node->val, new_queue->size);
    
    pop(new_queue);
    pop(new_queue);

    new_node = front(new_queue);
    printf("Front val: %d\n", new_node->val);

    new_node = back(new_queue);
    printf("Back val: %d size: %d\n", new_node->val, new_queue->size);
    
    return 0;
}
```

## 复杂度
- enQueue / deQueue：O(1)
- 空间：O(n)，链表动态分配

## 相关文档
- [[stack]] —— 栈用数组实现，与队列对比
- [[circularRingBuffer]] —— 环形缓冲区
- 参考：https://www.geeksforgeeks.org/queue-linked-list-implementation/
