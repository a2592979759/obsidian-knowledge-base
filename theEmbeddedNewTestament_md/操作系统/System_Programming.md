---
tags:
  - 操作系统
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/System_Programming.md
created: 2026-08-27
---

# 系统编程(System Programming)

> **掌握应用与操作系统之间的接口**  
> 理解用于健壮嵌入式应用的 POSIX API、系统调用和信号处理

---

## 📋 **目录(Table of Contents)**

- [系统编程基础](#system-programming-fundamentals)
- [POSIX API 与标准](#posix-apis-and-standards)
- [系统调用接口](#system-call-interface)
- [信号处理](#signal-handling)
- [文件与 I/O 操作](#file-and-io-operations)
- [进程与线程控制](#process-and-thread-control)
- [内存管理 API](#memory-management-apis)
- [高级系统编程](#advanced-system-programming)

---

## 🏗️ **系统编程基础**

### **什么是系统编程?**

系统编程涉及编写直接与操作系统内核接口的软件，以访问硬件资源和系统服务。与应用编程不同，系统编程在更低层次运行，需要深入理解操作系统的架构和行为。

**系统编程在嵌入式系统中的角色:**

- **硬件访问(Hardware Access)**: 直接控制硬件资源
- **性能优化(Performance Optimization)**: 绕过高级抽象以提高效率
- **系统集成(System Integration)**: 与内核服务和驱动接口
- **资源管理(Resource Management)**: 控制内存、CPU 和 I/O 分配
- **实时需求(Real-time Requirements)**: 满足严格的时序和响应要求

#### **系统编程与应用编程**

理解系统编程与应用编程之间的区别对于嵌入式开发至关重要:

**应用编程:**
- **级别(Level)**: 用户空间、高级抽象
- **关注点(Focus)**: 业务逻辑、用户界面、数据处理
- **安全(Safety)**: 受内核保护、不会崩溃系统
- **性能(Performance)**: 中等、带有内核开销
- **复杂度(Complexity)**: 较低、有丰富库支持

**系统编程:**
- **级别(Level)**: 内核空间或特权用户空间
- **关注点(Focus)**: 系统服务、硬件控制、性能
- **安全(Safety)**: 可能影响系统稳定性、需要仔细设计
- **性能(Performance)**: 高、可直接访问硬件
- **复杂度(Complexity)**: 更高、需要手动资源管理

```
┌─────────────────────────────────────┐
│         Application Layer           │ ← 高级抽象
│         (User programs)             │   - 库和框架
│                                    │   - 业务逻辑
├─────────────────────────────────────┤
│         System Programming          │ ← 底层系统接口
│         (System calls, APIs)        │   - 内核服务
│                                    │   - 硬件抽象
├─────────────────────────────────────┤
│         Operating System            │ ← 内核与驱动
│         (Kernel space)              │   - 资源管理
│                                    │   - 硬件控制
├─────────────────────────────────────┤
│         Hardware Layer              │ ← 物理硬件
│         (CPU, memory, I/O)          │   - 原始硬件资源
└─────────────────────────────────────┘
```

#### **系统编程哲学**

系统编程遵循**效率与可靠性原则(efficiency and reliability principle)**——编写既高效又健壮的代码，并仔细关注资源管理和错误处理。

**设计原则:**

- **效率(Efficiency)**: 最小化开销并最大化性能
- **可靠性(Reliability)**: 优雅且可预测地处理错误
- **可移植性(Portability)**: 编写跨系统工作的代码
- **可维护性(Maintainability)**: 创建清晰、文档完善的接口
- **安全(Security)**: 遵循安全最佳实践和原则

---

## 📚 **POSIX API 与标准**

### **可移植操作系统接口**

POSIX(可移植操作系统接口 Portable Operating System Interface)定义一组标准 API，确保应用在不同类 Unix 操作系统之间的可移植性。理解 POSIX 对于编写可移植的嵌入式应用至关重要。

#### **POSIX 标准哲学**

POSIX 遵循**可移植性原则(portability principle)**——定义跨不同操作系统一致工作的接口，同时允许特定于系统的优化和扩展。

**POSIX 设计目标:**

- **可移植性(Portability)**: 代码在不同 POSIX 兼容系统上工作
- **一致性(Consistency)**: 相似函数处处以相同方式工作
- **效率(Efficiency)**: API 为性能设计
- **可扩展性(Extensibility)**: 系统可以在保持兼容性的同时增加特性
- **向后兼容(Backward Compatibility)**: 旧代码继续工作

#### **核心 POSIX 标准**

**POSIX.1(核心服务):**
- **文件操作(File Operations)**: 打开、读取、写入、关闭文件
- **进程管理(Process Management)**: 创建、终止、等待进程
- **信号(Signals)**: 处理异步事件
- **内存管理(Memory Management)**: 分配和管理内存
- **进程间通信(Inter-process Communication)**: 管道、消息队列、共享内存

**POSIX.1b(实时扩展):**
- **实时调度(Real-time Scheduling)**: 基于优先级的调度策略
- **定时器(Timers)**: 高精度定时器和时钟
- **同步 I/O(Synchronous I/O)**: 阻塞 I/O 操作
- **内存锁定(Memory Locking)**: 将内存锁定在物理 RAM
- **消息队列(Message Queues)**: 实时消息传递

**POSIX.1c(线程):**
- **线程创建(Thread Creation)**: 创建和管理线程
- **线程同步(Thread Synchronization)**: 互斥锁、条件变量、信号量
- **线程特定数据(Thread-specific Data)**: 每线程存储
- **线程取消(Thread Cancellation)**: 优雅地终止线程
- **线程调度(Thread Scheduling)**: 线程级调度控制

#### **POSIX 合规级别**

不同系统提供不同程度的 POSIX 合规:

```
┌─────────────────────────────────────┐
│         Full POSIX Compliance      │ ← 完整标准支持
│         (Linux, FreeBSD)           │   - 所有必需 API
│                                    │   - 可选扩展
├─────────────────────────────────────┤
│         Partial POSIX Compliance   │ ← 仅核心功能
│         (macOS, Windows Subsystem) │   - 基本 API
│                                    │   - 缺少一些扩展
├─────────────────────────────────────┤
│         Minimal POSIX Support      │ ← 基本兼容性
│         (Embedded RTOS)            │   - 仅核心 API
│                                    │   - 功能有限
└─────────────────────────────────────┘
```

---

## 🔧 **系统调用接口**

### **用户空间与内核空间之间的桥梁**

系统调用提供用户空间应用程序向内核请求服务的基本机制。理解系统调用如何工作对于高效的系统编程至关重要。

#### **系统调用架构**

系统调用遵循一个明确定义的过程，涉及几个架构层:

```
用户应用程序
       │
       ▼
   库函数 (例如 printf)
       │
       ▼
   系统调用包装器
       │
       ▼
   系统调用编号(存于寄存器)
       │
       ▼
   内核入口点
       │
       ▼
   系统调用处理程序
       │
       ▼
   内核服务函数
       │
       ▼
   返回用户空间
```

**系统调用流程:**

1. **用户准备**: 应用程序将系统调用编号和参数放入寄存器
2. **陷阱指令(trap)**: 特殊指令触发转换到内核模式
3. **内核入口**: 内核保存用户上下文并切换到内核模式
4. **参数验证**: 内核为安全验证所有参数
5. **服务执行**: 内核执行请求的操作
6. **上下文恢复**: 内核恢复用户上下文并返回结果

#### **常见系统调用类别**

**进程管理:**
- **`fork()`**: 创建新进程
- **`execve()`**: 执行程序
- **`wait()`**: 等待子进程
- **`exit()`**: 终止进程
- **`getpid()`**: 获取进程 ID

**文件操作:**
- **`open()`**: 打开文件或设备
- **`read()`**: 从文件读取数据
- **`write()`**: 向文件写入数据
- **`close()`**: 关闭文件描述符
- **`lseek()`**: 改变文件位置

**内存管理:**
- **`brk()`**: 改变数据段大小
- **`mmap()`**: 将文件或设备映射到内存
- **`munmap()`**: 取消映射内存区域
- **`mprotect()`**: 改变内存保护

**进程间通信:**
- **`pipe()`**: 创建用于 IPC 的管道
- **`shmget()`**: 获取共享内存段
- **`msgget()`**: 获取消息队列
- **`semget()`**: 获取信号量集

#### **系统调用性能注意事项**

系统调用有开销，可能影响嵌入式系统的性能:

**开销来源:**
- **模式切换(Mode Switch)**: 用户到内核模式的转换
- **上下文保存/恢复(Context Save/Restore)**: 保存和恢复 CPU 寄存器
- **参数验证(Parameter Validation)**: 内核验证所有参数
- **内存访问(Memory Access)**: 跨用户/内核边界的数据访问
- **调度(Scheduling)**: 系统调用期间可能发生上下文切换

**优化策略:**
- **批处理操作(Batch Operations)**: 将多个操作合并为单个系统调用
- **内存映射(Memory Mapping)**: 对大型 I/O 操作使用 `mmap()`
- **直接 I/O(Direct I/O)**: 适当时绕过内核缓冲
- **系统调用批处理(System Call Batching)**: 使用 `io_uring` 或类似机制
- **避免不必要的调用**: 缓存结果并最小化调用

---

## ⚡ **信号处理**

### **管理异步事件**

信号提供内核通知进程异步事件的机制。理解信号处理对于构建能够响应外部事件和系统条件的健壮嵌入式应用至关重要。

#### **信号哲学**

信号处理遵循**异步事件原则(asynchronous event principle)**——提供一种机制，使进程能够响应其正常执行流之外发生的事件。

**信号设计目标:**

- **异步(Asynchronous)**: 处理在不可预测时间发生的事件
- **高效(Efficient)**: 信号投递的最小开销
- **灵活(Flexible)**: 允许进程定制信号处理
- **可靠(Reliable)**: 确保信号投递到预期进程
- **安全(Safe)**: 防止信号处理破坏进程状态

#### **信号类型与类别**

**标准信号(1-31):**
- **终止信号(Termination Signals)**: SIGTERM、SIGKILL、SIGQUIT
- **错误信号(Error Signals)**: SIGSEGV、SIGBUS、SIGFPE、SIGILL
- **控制信号(Control Signals)**: SIGINT、SIGTSTP、SIGCONT
- **告警信号(Alarm Signals)**: SIGALRM、SIGVTALRM、SIGPROF
- **用户信号(User Signals)**: SIGUSR1、SIGUSR2

**实时信号(32-63):**
- **扩展范围(Extended Range)**: 供应用使用的额外信号编号
- **队列支持(Queue Support)**: 多个信号可以排队
- **优先级(Priority)**: 比标准信号更高优先级
- **自定义用途(Custom Use)**: 应用定义的目的

#### **信号处理实现**

信号处理涉及设置信号处理程序并管理信号投递:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

// Global variables for signal handling
volatile sig_atomic_t signal_received = 0;
volatile sig_atomic_t graceful_shutdown = 0;

// Signal handler for graceful shutdown
void signal_handler(int sig) {
    switch (sig) {
        case SIGINT:
            printf("Received SIGINT (Ctrl+C)\n");
            graceful_shutdown = 1;
            break;
            
        case SIGTERM:
            printf("Received SIGTERM\n");
            graceful_shutdown = 1;
            break;
            
        case SIGUSR1:
            printf("Received SIGUSR1\n");
            signal_received = 1;
            break;
            
        case SIGALRM:
            printf("Received SIGALRM (timeout)\n");
            signal_received = 1;
            break;
            
        default:
            printf("Received signal %d\n", sig);
            break;
    }
}

// Set up signal handling
int setup_signal_handling(void) {
    struct sigaction sa;
    
    // Initialize signal action structure
    memset(&sa, 0, sizeof(sa));
    sa.sa_handler = signal_handler;
    sa.sa_flags = SA_RESTART;  // Restart interrupted system calls
    
    // Set up signal handlers
    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("Failed to set SIGINT handler");
        return -1;
    }
    
    if (sigaction(SIGTERM, &sa, NULL) == -1) {
        perror("Failed to set SIGTERM handler");
        return -1;
    }
    
    if (sigaction(SIGUSR1, &sa, NULL) == -1) {
        perror("Failed to set SIGUSR1 handler");
        return -1;
    }
    
    if (sigaction(SIGALRM, &sa, NULL) == -1) {
        perror("Failed to set SIGALRM handler");
        return -1;
    }
    
    printf("Signal handlers set up successfully\n");
    return 0;
}

// Main application loop
int main() {
    // Set up signal handling
    if (setup_signal_handling() == -1) {
        exit(1);
    }
    
    printf("Application started (PID: %d)\n", getpid());
    printf("Send SIGUSR1 to trigger action, SIGINT/SIGTERM to exit\n");
    
    // Main application loop
    while (!graceful_shutdown) {
        // Check for signals
        if (signal_received) {
            printf("Processing signal...\n");
            signal_received = 0;
            
            // Perform signal-specific actions
            // This could involve updating state, logging, etc.
        }
        
        // Simulate application work
        sleep(1);
    }
    
    printf("Graceful shutdown initiated\n");
    
    // Cleanup code here
    printf("Application terminated\n");
    
    return 0;
}
```

**信号处理最佳实践:**

- **保持处理程序简单**: 信号处理程序应最小且快速
- **使用易变变量(volatile)**: 将信号标志标记为 volatile 以便正确访问
- **避免复杂操作**: 不要在处理程序中调用不可重入函数
- **处理所有信号**: 为应用使用的所有信号设置处理程序
- **优雅关机**: 在信号处理程序中实现适当清理

---

## 📁 **文件与 I/O 操作**

### **管理数据输入输出**

文件与 I/O 操作构成大多数系统编程任务的基础。理解如何高效地从文件、设备和网络连接读取和写入对于嵌入式应用至关重要。

#### **I/O 哲学**

I/O 操作遵循**效率与可靠性原则(efficiency and reliability principle)**——提供快速、可靠的数据访问，同时优雅地处理错误并高效管理系统资源。

**I/O 设计目标:**

- **性能(Performance)**: 最小化 I/O 开销并最大化吞吐量
- **可靠性(Reliability)**: 确保数据完整性并优雅地处理错误
- **效率(Efficiency)**: 为不同用例使用适当的 I/O 方法
- **可移植性(Portability)**: 跨不同系统一致工作
- **可扩展性(Scalability)**: 处理不同数据大小和 I/O 模式

#### **文件描述符管理**

文件描述符是类 Unix 系统中 I/O 操作的基本抽象:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <errno.h>

// File descriptor management example
int manage_file_descriptors(void) {
    int fd1, fd2, fd3;
    char buffer[1024];
    ssize_t bytes_read, bytes_written;
    
    // Open source file for reading
    fd1 = open("source.txt", O_RDONLY);
    if (fd1 == -1) {
        perror("Failed to open source.txt");
        return -1;
    }
    
    // Open destination file for writing
    fd2 = open("destination.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd2 == -1) {
        perror("Failed to open destination.txt");
        close(fd1);
        return -1;
    }
    
    // Open log file for appending
    fd3 = open("operation.log", O_WRONLY | O_CREAT | O_APPEND, 0644);
    if (fd3 == -1) {
        perror("Failed to open operation.log");
        close(fd1);
        close(fd2);
        return -1;
    }
    
    // Copy data from source to destination
    while ((bytes_read = read(fd1, buffer, sizeof(buffer))) > 0) {
        bytes_written = write(fd2, buffer, bytes_read);
        if (bytes_written == -1) {
            perror("Write failed");
            break;
        }
        
        // Log operation
        dprintf(fd3, "Copied %zd bytes\n", bytes_written);
    }
    
    if (bytes_read == -1) {
        perror("Read failed");
    }
    
    // Close all file descriptors
    close(fd1);
    close(fd2);
    close(fd3);
    
    return 0;
}
```

**文件描述符最佳实践:**

- **检查返回值**: 始终检查 I/O 函数的返回值
- **关闭描述符**: 确保所有文件描述符被正确关闭
- **错误处理**: 优雅地处理 I/O 错误
- **资源限制**: 注意系统中打开文件的限制
- **描述符泄漏**: 避免让文件描述符保持打开

#### **高级 I/O 技术**

**内存映射 I/O:**
```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>

int memory_mapped_io_example(void) {
    int fd;
    struct stat sb;
    char *mapped_data;
    
    // Open file
    fd = open("large_file.txt", O_RDONLY);
    if (fd == -1) {
        perror("Failed to open file");
        return -1;
    }
    
    // Get file size
    if (fstat(fd, &sb) == -1) {
        perror("Failed to get file stats");
        close(fd);
        return -1;
    }
    
    // Map file to memory
    mapped_data = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (mapped_data == MAP_FAILED) {
        perror("Failed to map file");
        close(fd);
        return -1;
    }
    
    // Process data in memory
    printf("File mapped to memory, size: %ld bytes\n", sb.st_size);
    
    // Unmap and close
    munmap(mapped_data, sb.st_size);
    close(fd);
    
    return 0;
}
```

**非阻塞 I/O:**
```c
#include <fcntl.h>

int set_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("Failed to get file flags");
        return -1;
    }
    
    flags |= O_NONBLOCK;
    if (fcntl(fd, F_SETFL, flags) == -1) {
        perror("Failed to set non-blocking mode");
        return -1;
    }
    
    return 0;
}
```

---

## 🔄 **进程与线程控制**

### **管理执行上下文**

进程与线程控制涉及创建、管理和协调多个执行上下文。这对于构建需要同时处理多个任务的嵌入式应用至关重要。

#### **进程控制哲学**

进程控制遵循**隔离与协调原则(isolation and coordination principle)**——为不同任务提供独立的执行上下文，同时实现受控的通信和资源共享。

**进程控制目标:**

- **隔离(Isolation)**: 独立进程之间不能相互干扰
- **资源管理(Resource Management)**: 系统资源的高效分配和共享
- **通信(Communication)**: 实现受控的进程间通信
- **调度(Scheduling)**: 协调多个进程的执行
- **安全(Security)**: 控制对系统资源的访问

#### **进程创建与管理**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>

// Process creation and management example
int process_management_example(void) {
    pid_t pid1, pid2;
    int status1, status2;
    
    printf("Parent process (PID: %d) starting\n", getpid());
    
    // Create first child process
    pid1 = fork();
    if (pid1 == -1) {
        perror("First fork failed");
        return -1;
    }
    
    if (pid1 == 0) {
        // First child process
        printf("First child (PID: %d) created\n", getpid());
        
        // Simulate work
        sleep(2);
        printf("First child completing\n");
        exit(10);  // Exit with status 10
    }
    
    // Create second child process
    pid2 = fork();
    if (pid2 == -1) {
        perror("Second fork failed");
        wait(&status1);  // Wait for first child
        return -1;
    }
    
    if (pid2 == 0) {
        // Second child process
        printf("Second child (PID: %d) created\n", getpid());
        
        // Simulate work
        sleep(1);
        printf("Second child completing\n");
        exit(20);  // Exit with status 20
    }
    
    // Parent process waits for both children
    printf("Parent waiting for children to complete\n");
    
    waitpid(pid1, &status1, 0);
    printf("First child (PID: %d) completed with status: %d\n", 
           pid1, WEXITSTATUS(status1));
    
    waitpid(pid2, &status2, 0);
    printf("Second child (PID: %d) completed with status: %d\n", 
           pid2, WEXITSTATUS(status2));
    
    printf("All children completed\n");
    return 0;
}
```

#### **线程创建与管理**

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

// Thread data structure
struct thread_data {
    int thread_id;
    int work_duration;
    pthread_mutex_t *mutex;
};

// Thread function
void *thread_function(void *arg) {
    struct thread_data *data = (struct thread_data *)arg;
    
    printf("Thread %d starting\n", data->thread_id);
    
    // Simulate work
    sleep(data->work_duration);
    
    // Critical section
    pthread_mutex_lock(data->mutex);
    printf("Thread %d in critical section\n", data->thread_id);
    sleep(1);  // Simulate critical work
    pthread_mutex_unlock(data->mutex);
    
    printf("Thread %d completing\n", data->thread_id);
    
    return (void *)(long)(data->thread_id * 100);
}

// Thread management example
int thread_management_example(void) {
    pthread_t threads[3];
    struct thread_data thread_data[3];
    pthread_mutex_t mutex;
    void *thread_result;
    int i;
    
    // Initialize mutex
    if (pthread_mutex_init(&mutex, NULL) != 0) {
        perror("Failed to initialize mutex");
        return -1;
    }
    
    // Create threads
    for (i = 0; i < 3; i++) {
        thread_data[i].thread_id = i;
        thread_data[i].work_duration = i + 1;
        thread_data[i].mutex = &mutex;
        
        if (pthread_create(&threads[i], NULL, thread_function, &thread_data[i]) != 0) {
            perror("Failed to create thread");
            return -1;
        }
    }
    
    // Wait for threads to complete
    for (i = 0; i < 3; i++) {
        if (pthread_join(threads[i], &thread_result) != 0) {
            perror("Failed to join thread");
            return -1;
        }
        
        printf("Thread %d returned: %ld\n", i, (long)thread_result);
    }
    
    // Clean up mutex
    pthread_mutex_destroy(&mutex);
    
    return 0;
}
```

---

## 💾 **内存管理 API**

### **控制内存分配与访问**

内存管理 API 提供对内存如何分配、管理和保护的控制的。理解这些 API 对于构建高效可靠的嵌入式应用至关重要。

#### **内存管理哲学**

内存管理遵循**效率与安全原则(efficiency and safety principle)**——提供快速、可靠的内存分配，同时防止内存相关错误并针对系统性能进行优化。

**内存管理目标:**

- **效率(Efficiency)**: 最小化内存分配开销
- **安全(Safety)**: 防止内存泄漏和缓冲区溢出
- **性能(Performance)**: 优化内存访问模式
- **灵活性(Flexibility)**: 支持各种分配策略
- **可靠性(Reliability)**: 优雅地处理分配失败

#### **动态内存分配**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

// Memory allocation with error handling
void *safe_malloc(size_t size) {
    void *ptr = malloc(size);
    if (ptr == NULL) {
        fprintf(stderr, "Memory allocation failed: %s\n", strerror(errno));
        exit(1);
    }
    return ptr;
}

// Memory allocation example
int memory_allocation_example(void) {
    int *int_array;
    char *string_buffer;
    struct {
        int id;
        char name[64];
        double value;
    } *data_array;
    
    // Allocate integer array
    int_array = safe_malloc(100 * sizeof(int));
    for (int i = 0; i < 100; i++) {
        int_array[i] = i * i;
    }
    
    // Allocate string buffer
    string_buffer = safe_malloc(1024);
    strcpy(string_buffer, "Hello, World!");
    
    // Allocate data structure array
    data_array = safe_malloc(50 * sizeof(*data_array));
    for (int i = 0; i < 50; i++) {
        data_array[i].id = i;
        snprintf(data_array[i].name, sizeof(data_array[i].name), "Item_%d", i);
        data_array[i].value = i * 1.5;
    }
    
    // Use allocated memory
    printf("Integer array[50] = %d\n", int_array[50]);
    printf("String buffer: %s\n", string_buffer);
    printf("Data array[25]: id=%d, name=%s, value=%.1f\n",
           data_array[25].id, data_array[25].name, data_array[25].value);
    
    // Free allocated memory
    free(int_array);
    free(string_buffer);
    free(data_array);
    
    return 0;
}
```

#### **内存映射与保护**

```c
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

// Memory mapping example
int memory_mapping_example(void) {
    int fd;
    char *mapped_memory;
    size_t page_size = getpagesize();
    size_t map_size = page_size * 2;  // Map 2 pages
    
    // Create temporary file for mapping
    fd = open("/tmp/memory_map", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("Failed to create temporary file");
        return -1;
    }
    
    // Extend file to desired size
    if (ftruncate(fd, map_size) == -1) {
        perror("Failed to extend file");
        close(fd);
        return -1;
    }
    
    // Map file to memory
    mapped_memory = mmap(NULL, map_size, PROT_READ | PROT_WRITE, 
                        MAP_SHARED, fd, 0);
    if (mapped_memory == MAP_FAILED) {
        perror("Failed to map memory");
        close(fd);
        return -1;
    }
    
    // Write data to mapped memory
    strcpy(mapped_memory, "Hello from mapped memory!");
    printf("Wrote to mapped memory: %s\n", mapped_memory);
    
    // Change memory protection to read-only
    if (mprotect(mapped_memory, map_size, PROT_READ) == -1) {
        perror("Failed to change memory protection");
    } else {
        printf("Memory protection changed to read-only\n");
        
        // Try to write (this should fail)
        strcpy(mapped_memory, "This should fail");
        printf("Read from memory: %s\n", mapped_memory);
    }
    
    // Unmap and cleanup
    munmap(mapped_memory, map_size);
    close(fd);
    unlink("/tmp/memory_map");
    
    return 0;
}
```

---

## 🚀 **高级系统编程**

### **超越基本系统接口**

高级系统编程涉及用于优化性能、管理复杂资源和构建健壮系统的复杂技术。这些技术对于高性能嵌入式应用至关重要。

#### **高级 I/O 技术**

**使用 `io_uring` 的异步 I/O:**
```c
#include <linux/io_uring.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <fcntl.h>

// Simplified io_uring example (Linux-specific)
int io_uring_example(void) {
    struct io_uring ring;
    struct io_uring_sqe *sqe;
    struct io_uring_cqe *cqe;
    int fd, ret;
    
    // Initialize io_uring
    ret = io_uring_queue_init(32, &ring, 0);
    if (ret < 0) {
        fprintf(stderr, "Failed to initialize io_uring: %s\n", strerror(-ret));
        return -1;
    }
    
    // Open file for I/O
    fd = open("test_file.txt", O_RDWR | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("Failed to open file");
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    // Submit write request
    sqe = io_uring_get_sqe(&ring);
    if (!sqe) {
        fprintf(stderr, "Failed to get SQE\n");
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    char write_data[] = "Hello, io_uring!";
    io_uring_prep_write(sqe, fd, write_data, strlen(write_data), 0);
    
    // Submit the request
    ret = io_uring_submit(&ring);
    if (ret < 0) {
        fprintf(stderr, "Failed to submit request: %s\n", strerror(-ret));
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    // Wait for completion
    ret = io_uring_wait_cqe(&ring, &cqe);
    if (ret < 0) {
        fprintf(stderr, "Failed to wait for completion: %s\n", strerror(-ret));
        close(fd);
        io_uring_queue_exit(&ring);
        return -1;
    }
    
    printf("Write completed: %d bytes\n", cqe->res);
    
    // Mark completion as seen
    io_uring_cqe_seen(&ring, cqe);
    
    // Cleanup
    close(fd);
    io_uring_queue_exit(&ring);
    
    return 0;
}
```

#### **进程与线程同步**

**使用 futex 的高级同步:**
```c
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdint.h>

// Futex-based mutex implementation
struct futex_mutex {
    int32_t value;
};

// Futex system call wrapper
int futex(int *uaddr, int op, int val, const struct timespec *timeout,
          int *uaddr2, int val3) {
    return syscall(SYS_futex, uaddr, op, val, timeout, uaddr2, val3);
}

// Initialize futex mutex
void futex_mutex_init(struct futex_mutex *mutex) {
    mutex->value = 0;
}

// Lock futex mutex
void futex_mutex_lock(struct futex_mutex *mutex) {
    int32_t expected = 0;
    
    // Try to acquire lock
    while (!__sync_bool_compare_and_swap(&mutex->value, expected, 1)) {
        // Wait for lock to become available
        futex(&mutex->value, FUTEX_WAIT, 1, NULL, NULL, 0);
        expected = 0;
    }
}

// Unlock futex mutex
void futex_mutex_unlock(struct futex_mutex *mutex) {
    int32_t expected = 1;
    
    // Release lock
    if (__sync_bool_compare_and_swap(&mutex->value, expected, 0)) {
        // Wake up waiting threads
        futex(&mutex->value, FUTEX_WAKE, 1, NULL, NULL, 0);
    }
}
```

---

## 🎯 **结论**

系统编程为构建直接与操作系统接口的健壮、高效嵌入式应用提供了基础。理解 POSIX API、系统调用和信号处理对于创建能高效管理系统资源并响应外部事件的应用至关重要。

**关键要点:**

- **POSIX 标准**确保跨不同系统的可移植性
- **系统调用**提供对内核服务的受控访问
- **信号处理**实现对异步事件的响应
- **I/O 操作**构成数据处理的基础
- **进程与线程控制**实现并发执行
- **内存管理 API**提供对资源分配的控制
- **高级技术**优化性能和可靠性

**前进之路:**

随着嵌入式系统变得更复杂并需要更复杂的操作系统交互，系统编程技能的重要性只会增加。现代系统继续演进，提供新的 API 和技术，使更强大、更高效的嵌入式应用成为可能。

**记住**: 系统编程不只是进行系统调用——它是关于理解操作系统如何工作、如何高效管理系统资源、以及如何构建能可靠与底层系统交互的应用。你在这里发展的技能将贯穿你的嵌入式系统职业生涯，让你能够创建健壮、高效、可维护的系统。
