---
tags: [并发, 线程, 同步, 嵌入式]
source: Data_Struct_Implementation/concurrency/ReaderWritter
created: 2026-08-27
---

# 读者写者问题（Reader-Writer）

### 代码

```C
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>

/*
本程序提供用 mutex 和信号量求解第一个读者-写者问题的可行方案。
我用了 10 个读者和 5 个写者来演示。你可以自由调整这些值。
*/

sem_t wrt;
pthread_mutex_t mutex;
int cnt = 1;
int numreader = 0;

void *writer(void *wno)
{   
    sem_wait(&wrt);
    cnt = cnt*2;
    printf("Writer %d modified cnt to %d\n",(*((int *)wno)),cnt);
    sem_post(&wrt);

}
void *reader(void *rno)
{   
    // 读者在修改 numreader 前先加锁
    pthread_mutex_lock(&mutex);
    numreader++;
    if(numreader == 1) {
        sem_wait(&wrt); // 若是第一个读者，则阻塞写者
    }
    pthread_mutex_unlock(&mutex);
    // 读区
    printf("Reader %d: read cnt as %d\n",*((int *)rno),cnt);

    // 读者在修改 numreader 前先加锁
    pthread_mutex_lock(&mutex);
    numreader--;
    if(numreader == 0) {
        sem_post(&wrt); // 若是最后一个读者，则唤醒写者
    }
    pthread_mutex_unlock(&mutex);
}

int main()
{   

    pthread_t read[10],write[5];
    pthread_mutex_init(&mutex, NULL);
    sem_init(&wrt,0,1);

    int a[10] = {1,2,3,4,5,6,7,8,9,10}; // 仅用于给生产者和消费者编号

    for(int i = 0; i < 10; i++) {
        pthread_create(&read[i], NULL, (void *)reader, (void *)&a[i]);
    }
    for(int i = 0; i < 5; i++) {
        pthread_create(&write[i], NULL, (void *)writer, (void *)&a[i]);
    }

    for(int i = 0; i < 10; i++) {
        pthread_join(read[i], NULL);
    }
    for(int i = 0; i < 5; i++) {
        pthread_join(write[i], NULL);
    }

    pthread_mutex_destroy(&mutex);
    sem_destroy(&wrt);

    return 0;
    
}
```

## 分析

- 用信号量 `wrt` 控制写者互斥地访问共享数据 `cnt`；只有第一个读者进入时对 `wrt` 加锁、最后一个读者离开时解锁。
- 用 `mutex` 保护 `numreader` 计数器的修改。
- 结果：多个读者可以同时读；写者与读者、写者与写者之间互斥。

## 参考

[Reader and writer by Shivam](https://shivammitra.com/reader-writer-problem-in-c/#)

## 相关文档
- [[BuildingH2O]] —— 生成水分子
- [[BoundedQueue]] —— 有界阻塞队列
