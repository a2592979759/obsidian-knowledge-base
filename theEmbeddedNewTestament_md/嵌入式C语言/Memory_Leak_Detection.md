---
tags:
  - 嵌入式C
source: Embedded_C/Memory_Leak_Detection.md
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 练习与深度学习
>
> 把这些 C / C++ 概念作为社区排名的面试题（附模型答案）来练习，并查看交互式深度学习指南。
>
> 👉 **[浏览 C / C++ 面试题 →](https://embeddedinterviewlab.com/questions/domain/c?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=embedded_c)** &nbsp;·&nbsp; **[浏览 C / C++ 指南 →](https://embeddedinterviewlab.com/categories/c?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=embedded_c)**

---

# 内存泄漏检测（Memory Leak Detection）

## 📋 目录（Table of Contents）
- [概述](#-overview)
- [内存泄漏的类型](#-types-of-memory-leaks)
- [静态分析工具](#-static-analysis-tools)
- [动态分析工具](#-dynamic-analysis-tools)
- [自定义内存追踪](#-custom-memory-tracking)
- [嵌入式特有的检测](#-embedded-specific-detection)
- [实时泄漏检测](#-real-time-leak-detection)
- [常见泄漏模式](#-common-leak-patterns)
- [预防策略](#-prevention-strategies)
- [最佳实践](#-best-practices)
- [面试题](#-interview-questions)
- [附加资源](#-additional-resources)

## 🎯 概述（Overview）

内存泄漏（Memory leak）发生在已分配的内存未被正确释放时，导致内存逐渐耗尽。在嵌入式系统中，由于资源有限且设备需要长时间运行（long uptimes），内存泄漏可能是灾难性的。

## 🔍 内存泄漏的类型（Types of Memory Leaks）

### 经典内存泄漏（Classic Memory Leaks）
```c
// Classic memory leak - allocated but never freed
// 经典内存泄漏 - 已分配但从未释放
void classic_memory_leak() {
    void* ptr = malloc(1024);
    // Use ptr...
    // 使用 ptr...
    // Forgot to call free(ptr) - LEAK!
    // 忘记调用 free(ptr) - 泄漏！
}

// Memory leak in error path
// 错误路径中的内存泄漏
void leak_in_error_path() {
    void* ptr1 = malloc(512);
    if (ptr1 == NULL) return;  // Early return without cleanup
                                // 提前返回而未清理
    
    void* ptr2 = malloc(256);
    if (ptr2 == NULL) {
        // Forgot to free ptr1 - LEAK!
        // 忘记释放 ptr1 - 泄漏！
        return;
    }
    
    // Use both pointers...
    // 使用两个指针...
    free(ptr1);
    free(ptr2);
}
```

### 资源泄漏（Resource Leaks）
```c
// File handle leak
// 文件句柄泄漏
void file_handle_leak() {
    FILE* file = fopen("data.txt", "r");
    if (file == NULL) return;
    
    // Process file...
    // 处理文件...
    // Forgot to call fclose(file) - RESOURCE LEAK!
    // 忘记调用 fclose(file) - 资源泄漏！
}

// Mutex leak
// 互斥锁泄漏
#include <pthread.h>

void mutex_leak() {
    pthread_mutex_t* mutex = malloc(sizeof(pthread_mutex_t));
    pthread_mutex_init(mutex, NULL);
    
    // Use mutex...
    // 使用互斥锁...
    // Forgot to call pthread_mutex_destroy() and free() - LEAK!
    // 忘记调用 pthread_mutex_destroy() 和 free() - 泄漏！
}
```

### 循环引用泄漏（Circular Reference Leaks）
```c
typedef struct node {
    int data;
    struct node* next;
} node_t;

void circular_reference_leak() {
    node_t* head = malloc(sizeof(node_t));
    node_t* tail = malloc(sizeof(node_t));
    
    head->next = tail;
    tail->next = head;  // Circular reference
                        // 循环引用
    
    // If we only free head, tail becomes unreachable
    // 如果我们只释放 head，tail 将变得不可达
    free(head);
    // tail is now leaked!
    // tail 现在泄漏了！
}
```

## 🔧 静态分析工具（Static Analysis Tools）

### 编译器警告（Compiler Warnings）
```c
// Enable all warnings for memory leak detection
// 启用所有警告以进行内存泄漏检测
// gcc -Wall -Wextra -Werror -fsanitize=address

void potential_leak_function() {
    void* ptr = malloc(100);
    // Compiler warning: variable 'ptr' set but not used
    // 编译器警告：变量 'ptr' 已设置但未被使用
    // This could indicate a leak
    // 这可能表示存在泄漏
}
```

### 使用 Clang 的静态分析（Static Analysis with Clang）
```c
// clang --analyze -Xanalyzer -analyzer-output=text

void analyzed_function() {
    void* ptr = malloc(100);
    if (some_condition()) {
        free(ptr);
        return;
    }
    // Static analyzer detects: ptr is not freed in this path
    // 静态分析器检测到：ptr 在此路径中未被释放
}
```

### 自定义静态分析（Custom Static Analysis）
```c
// Simple static analysis for common patterns
// 针对常见模式的简单静态分析
#include <regex.h>

void check_for_malloc_without_free(const char* source_code) {
    regex_t malloc_regex, free_regex;
    regcomp(&malloc_regex, "malloc\\s*\\(", REG_EXTENDED);
    regcomp(&free_regex, "free\\s*\\(", REG_EXTENDED);
    
    // Count malloc and free calls
    // 统计 malloc 和 free 调用次数
    // If malloc_count > free_count, potential leak
    // 如果 malloc_count > free_count，则可能存在泄漏
}
```

## 🔍 动态分析工具（Dynamic Analysis Tools）

### AddressSanitizer（ASan）
```c
// Compile with: gcc -fsanitize=address -g
// 使用以下命令编译：gcc -fsanitize=address -g

void asan_demo() {
    void* ptr = malloc(100);
    // Use ptr...
    // 使用 ptr...
    // If we forget to free(ptr), ASan will report leak
    // 如果我们忘记释放 free(ptr)，ASan 将报告泄漏
    
    // ASan also detects:
    // ASan 还能检测：
    // - Buffer overflows
    // - 缓冲区溢出（Buffer overflows）
    // - Use-after-free
    // - 释放后使用（Use-after-free）
    // - Double-free
    // - 双重释放（Double-free）
}

// ASan output example:
// ASan 输出示例：
// ==12345==ERROR: LeakSanitizer: detected memory leaks
// ==12345==SUMMARY: AddressSanitizer: 1 byte(s) leaked in 1 allocation(s)
```

### Valgrind
```c
// Run with: valgrind --leak-check=full --show-leak-kinds=all
// 使用以下命令运行：valgrind --leak-check=full --show-leak-kinds=all

void valgrind_demo() {
    void* ptr = malloc(100);
    // Use ptr...
    // 使用 ptr...
    // Valgrind will report:
    // Valgrind 将报告：
    // ==12345== 100 bytes in 1 blocks are definitely lost
    // ==12345==    at 0x4C2AB80: malloc (in /usr/lib/valgrind/vgpreload_memcheck.so)
    // ==12345==    by 0x400544: valgrind_demo (demo.c:5)
}
```

### 自定义内存追踪器（Custom Memory Tracker）
```c
typedef struct {
    void* ptr;
    size_t size;
    const char* file;
    int line;
    bool freed;
} allocation_record_t;

typedef struct {
    allocation_record_t* records;
    size_t count;
    size_t capacity;
} memory_tracker_t;

memory_tracker_t* create_memory_tracker(size_t initial_capacity) {
    memory_tracker_t* tracker = malloc(sizeof(memory_tracker_t));
    if (!tracker) return NULL;
    
    tracker->records = calloc(initial_capacity, sizeof(allocation_record_t));
    tracker->count = 0;
    tracker->capacity = initial_capacity;
    
    return tracker;
}

void* tracked_malloc(size_t size, const char* file, int line) {
    void* ptr = malloc(size);
    if (ptr) {
        // Add to tracker
        // 添加到追踪器
        allocation_record_t record = {
            .ptr = ptr,
            .size = size,
            .file = file,
            .line = line,
            .freed = false
        };
        add_allocation_record(&record);
    }
    return ptr;
}

void tracked_free(void* ptr) {
    if (ptr) {
        // Mark as freed in tracker
        // 在追踪器中标记为已释放
        mark_as_freed(ptr);
        free(ptr);
    }
}
```

## 🔧 嵌入式特有的检测（Embedded-Specific Detection）

### 内存池泄漏检测（Memory Pool Leak Detection）
```c
typedef struct {
    void* pool;
    size_t block_size;
    size_t total_blocks;
    bool* allocated_blocks;
    size_t allocated_count;
} leak_detecting_pool_t;

leak_detecting_pool_t* create_leak_detecting_pool(size_t block_size, 
                                                  size_t num_blocks) {
    leak_detecting_pool_t* pool = malloc(sizeof(leak_detecting_pool_t));
    if (!pool) return NULL;
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->allocated_count = 0;
    
    pool->pool = malloc(block_size * num_blocks);
    pool->allocated_blocks = calloc(num_blocks, sizeof(bool));
    
    return pool;
}

void* pool_allocate_with_tracking(leak_detecting_pool_t* pool) {
    for (size_t i = 0; i < pool->total_blocks; i++) {
        if (!pool->allocated_blocks[i]) {
            pool->allocated_blocks[i] = true;
            pool->allocated_count++;
            return (char*)pool->pool + (i * pool->block_size);
        }
    }
    return NULL;
}

void pool_free_with_tracking(leak_detecting_pool_t* pool, void* ptr) {
    size_t block_index = ((char*)ptr - (char*)pool->pool) / pool->block_size;
    if (block_index < pool->total_blocks && pool->allocated_blocks[block_index]) {
        pool->allocated_blocks[block_index] = false;
        pool->allocated_count--;
    }
}

void report_pool_leaks(leak_detecting_pool_t* pool) {
    if (pool->allocated_count > 0) {
        printf("MEMORY LEAK: %zu blocks not freed in pool\n", 
               pool->allocated_count);
        
        for (size_t i = 0; i < pool->total_blocks; i++) {
            if (pool->allocated_blocks[i]) {
                printf("  Block %zu still allocated\n", i);
            }
        }
    }
}
```

### 基于栈的内存追踪（Stack-Based Memory Tracking）
```c
// Track allocations on stack for embedded systems
// 在栈上追踪为嵌入式系统进行的分配
#define MAX_STACK_ALLOCATIONS 100

typedef struct {
    void* ptr;
    const char* function;
    int line;
} stack_allocation_t;

static stack_allocation_t allocation_stack[MAX_STACK_ALLOCATIONS];
static int stack_top = 0;

void* stack_tracked_malloc(size_t size, const char* function, int line) {
    void* ptr = malloc(size);
    if (ptr && stack_top < MAX_STACK_ALLOCATIONS) {
        allocation_stack[stack_top].ptr = ptr;
        allocation_stack[stack_top].function = function;
        allocation_stack[stack_top].line = line;
        stack_top++;
    }
    return ptr;
}

void stack_tracked_free(void* ptr) {
    if (ptr) {
        // Find and remove from stack
        // 从栈中找到并移除
        for (int i = 0; i < stack_top; i++) {
            if (allocation_stack[i].ptr == ptr) {
                // Remove by shifting
                // 通过移位移除
                for (int j = i; j < stack_top - 1; j++) {
                    allocation_stack[j] = allocation_stack[j + 1];
                }
                stack_top--;
                break;
            }
        }
        free(ptr);
    }
}

void check_stack_leaks() {
    if (stack_top > 0) {
        printf("STACK LEAKS DETECTED: %d allocations not freed\n", stack_top);
        for (int i = 0; i < stack_top; i++) {
            printf("  %s:%d - %p\n", 
                   allocation_stack[i].function,
                   allocation_stack[i].line,
                   allocation_stack[i].ptr);
        }
    }
}
```

## ⏱️ 实时泄漏检测（Real-time Leak Detection）

### 轻量级泄漏监视器（Lightweight Leak Monitor）
```c
typedef struct {
    uint32_t total_allocations;
    uint32_t total_frees;
    uint32_t current_allocated;
    uint32_t peak_allocated;
} rt_leak_monitor_t;

static rt_leak_monitor_t leak_monitor = {0};

void* rt_tracked_malloc(size_t size) {
    void* ptr = malloc(size);
    if (ptr) {
        leak_monitor.total_allocations++;
        leak_monitor.current_allocated++;
        
        if (leak_monitor.current_allocated > leak_monitor.peak_allocated) {
            leak_monitor.peak_allocated = leak_monitor.current_allocated;
        }
    }
    return ptr;
}

void rt_tracked_free(void* ptr) {
    if (ptr) {
        leak_monitor.total_frees++;
        leak_monitor.current_allocated--;
        free(ptr);
    }
}

void report_rt_leaks() {
    printf("Allocations: %u, Frees: %u, Current: %u, Peak: %u\n",
           leak_monitor.total_allocations,
           leak_monitor.total_frees,
           leak_monitor.current_allocated,
           leak_monitor.peak_allocated);
    
    if (leak_monitor.current_allocated > 0) {
        printf("POTENTIAL LEAK: %u blocks not freed\n", 
               leak_monitor.current_allocated);
    }
}
```

### 周期性泄漏检查（Periodic Leak Check）
```c
// Check for leaks periodically in real-time systems
// 在实时系统中周期性检查泄漏
void periodic_leak_check() {
    static uint32_t check_counter = 0;
    check_counter++;
    
    if (check_counter % 10000 == 0) {  // Check every 10k allocations
                                       // 每 10k 次分配检查一次
        uint32_t leak_count = leak_monitor.total_allocations - leak_monitor.total_frees;
        
        if (leak_count > 100) {  // Threshold for leak detection
                                 // 泄漏检测阈值
            printf("WARNING: Potential memory leak detected\n");
            report_rt_leaks();
        }
    }
}
```

## 🚨 常见泄漏模式（Common Leak Patterns）

### 1. 异常/错误路径泄漏（Exception/Error Path Leaks）
```c
// WRONG: Leak in error path
// 错误：错误路径中的泄漏
void* allocate_with_error_leak() {
    void* ptr1 = malloc(100);
    if (!ptr1) return NULL;
    
    void* ptr2 = malloc(200);
    if (!ptr2) {
        // LEAK: ptr1 not freed
        // 泄漏：ptr1 未释放
        return NULL;
    }
    
    // Use both pointers...
    // 使用两个指针...
    free(ptr1);
    free(ptr2);
    return ptr2;
}

// CORRECT: Proper cleanup
// 正确：正确清理
void* allocate_with_cleanup() {
    void* ptr1 = malloc(100);
    if (!ptr1) return NULL;
    
    void* ptr2 = malloc(200);
    if (!ptr2) {
        free(ptr1);  // Clean up before returning
                     // 返回前清理
        return NULL;
    }
    
    // Use both pointers...
    // 使用两个指针...
    free(ptr1);
    free(ptr2);
    return ptr2;
}
```

### 2. 循环泄漏（Loop Leaks）
```c
// WRONG: Leak in loop
// 错误：循环中的泄漏
void loop_leak() {
    for (int i = 0; i < 1000; i++) {
        void* ptr = malloc(100);
        // Use ptr...
        // 使用 ptr...
        // Forgot to free(ptr) - LEAK!
        // 忘记释放 free(ptr) - 泄漏！
    }
}

// CORRECT: Free in loop
// 正确：在循环中释放
void loop_no_leak() {
    for (int i = 0; i < 1000; i++) {
        void* ptr = malloc(100);
        // Use ptr...
        // 使用 ptr...
        free(ptr);  // Free before next iteration
                    // 在进入下一次迭代前释放
    }
}
```

### 3. 函数退出泄漏（Function Exit Leaks）
```c
// WRONG: Multiple exit points without cleanup
// 错误：多个退出点却未清理
void* function_with_exit_leaks() {
    void* ptr = malloc(100);
    
    if (condition1()) {
        return ptr;  // LEAK: ptr not freed
                     // 泄漏：ptr 未释放
    }
    
    if (condition2()) {
        return NULL;  // LEAK: ptr not freed
                      // 泄漏：ptr 未释放
    }
    
    // Use ptr...
    // 使用 ptr...
    free(ptr);
    return NULL;
}

// CORRECT: Single exit point or proper cleanup
// 正确：单一退出点或正确清理
void* function_with_cleanup() {
    void* ptr = malloc(100);
    if (!ptr) return NULL;
    
    void* result = NULL;
    
    if (condition1()) {
        result = ptr;
    } else if (condition2()) {
        free(ptr);
        result = NULL;
    } else {
        // Use ptr...
        // 使用 ptr...
        free(ptr);
        result = NULL;
    }
    
    return result;
}
```

## 🛡️ 预防策略（Prevention Strategies）

### 1. C 中的 RAII 模式（RAII Pattern in C）
```c
typedef struct {
    void* ptr;
    void (*cleanup)(void*);
} raii_wrapper_t;

raii_wrapper_t* create_raii_wrapper(void* ptr, void (*cleanup)(void*)) {
    raii_wrapper_t* wrapper = malloc(sizeof(raii_wrapper_t));
    if (wrapper) {
        wrapper->ptr = ptr;
        wrapper->cleanup = cleanup;
    }
    return wrapper;
}

void destroy_raii_wrapper(raii_wrapper_t* wrapper) {
    if (wrapper && wrapper->cleanup) {
        wrapper->cleanup(wrapper->ptr);
    }
    free(wrapper);
}

// Usage
// 用法
void example_raii() {
    raii_wrapper_t* wrapper = create_raii_wrapper(
        malloc(100), 
        free
    );
    
    // Use wrapper->ptr...
    // 使用 wrapper->ptr...
    
    destroy_raii_wrapper(wrapper);  // Automatically frees ptr
                                    // 自动释放 ptr
}
```

### 2. 带自动清理的内存池（Memory Pool with Automatic Cleanup）
```c
typedef struct {
    void* pool;
    size_t block_size;
    size_t total_blocks;
    bool* allocated_blocks;
} auto_cleanup_pool_t;

auto_cleanup_pool_t* create_auto_cleanup_pool(size_t block_size, 
                                              size_t num_blocks) {
    auto_cleanup_pool_t* pool = malloc(sizeof(auto_cleanup_pool_t));
    if (!pool) return NULL;
    
    pool->block_size = block_size;
    pool->total_blocks = num_blocks;
    pool->pool = malloc(block_size * num_blocks);
    pool->allocated_blocks = calloc(num_blocks, sizeof(bool));
    
    return pool;
}

void destroy_auto_cleanup_pool(auto_cleanup_pool_t* pool) {
    if (pool) {
        // Check for leaks before destroying
        // 在销毁前检查泄漏
        size_t leak_count = 0;
        for (size_t i = 0; i < pool->total_blocks; i++) {
            if (pool->allocated_blocks[i]) {
                leak_count++;
            }
        }
        
        if (leak_count > 0) {
            printf("WARNING: %zu blocks leaked in pool\n", leak_count);
        }
        
        free(pool->pool);
        free(pool->allocated_blocks);
        free(pool);
    }
}
```

### 3. 智能指针模式（Smart Pointer Pattern）
```c
typedef struct {
    void* ptr;
    int ref_count;
} smart_ptr_t;

smart_ptr_t* create_smart_ptr(void* ptr) {
    smart_ptr_t* smart = malloc(sizeof(smart_ptr_t));
    if (smart) {
        smart->ptr = ptr;
        smart->ref_count = 1;
    }
    return smart;
}

smart_ptr_t* smart_ptr_ref(smart_ptr_t* smart) {
    if (smart) {
        smart->ref_count++;
    }
    return smart;
}

void smart_ptr_unref(smart_ptr_t* smart) {
    if (smart) {
        smart->ref_count--;
        if (smart->ref_count <= 0) {
            free(smart->ptr);
            free(smart);
        }
    }
}
```

## ✅ 最佳实践（Best Practices）

### 1. 使用一致的分配/释放模式（Use Consistent Allocation/Deallocation Patterns）
```c
// Define allocation patterns
// 定义分配模式
#define ALLOCATE_AND_CHECK(ptr, size) \
    do { \
        ptr = malloc(size); \
        if (!ptr) { \
            fprintf(stderr, "Allocation failed at %s:%d\n", __FILE__, __LINE__); \
            return NULL; \
        } \
    } while(0)

#define FREE_AND_NULL(ptr) \
    do { \
        if (ptr) { \
            free(ptr); \
            ptr = NULL; \
        } \
    } while(0)

// Usage
// 用法
void* safe_allocation_example() {
    void* ptr1 = NULL, *ptr2 = NULL;
    
    ALLOCATE_AND_CHECK(ptr1, 100);
    ALLOCATE_AND_CHECK(ptr2, 200);
    
    // Use pointers...
    // 使用指针...
    
    FREE_AND_NULL(ptr2);
    FREE_AND_NULL(ptr1);
    return NULL;
}
```

### 2. 在调试构建中实现内存泄漏检测（Implement Memory Leak Detection in Debug Builds）
```c
#ifdef DEBUG
    #define MALLOC(size) debug_malloc(size, __FILE__, __LINE__)
    #define FREE(ptr) debug_free(ptr, __FILE__, __LINE__)
#else
    #define MALLOC(size) malloc(size)
    #define FREE(ptr) free(ptr)
#endif

void* debug_malloc(size_t size, const char* file, int line) {
    void* ptr = malloc(size);
    if (ptr) {
        printf("DEBUG: malloc(%zu) at %s:%d -> %p\n", size, file, line, ptr);
        add_to_debug_tracker(ptr, size, file, line);
    }
    return ptr;
}

void debug_free(void* ptr, const char* file, int line) {
    if (ptr) {
        printf("DEBUG: free(%p) at %s:%d\n", ptr, file, line);
        remove_from_debug_tracker(ptr);
        free(ptr);
    }
}
```

### 3. 定期内存审计（Regular Memory Audits）
```c
void perform_memory_audit() {
    struct mallinfo info = mallinfo();
    
    printf("Memory Audit:\n");
    printf("  Total allocated: %d bytes\n", info.uordblks);
    printf("  Total free: %d bytes\n", info.fordblks);
    printf("  Largest free block: %d bytes\n", info.mxordblk);
    printf("  Number of free blocks: %d\n", info.ordblks);
    
    // Check for potential leaks
    // 检查潜在泄漏
    if (info.uordblks > 0 && info.fordblks < 1000) {
        printf("WARNING: Low free memory - potential leak?\n");
    }
}
```

## 🎯 面试题（Interview Questions）

### 基础问题（Basic Questions）
1. **什么是内存泄漏（Memory leak），为什么它在嵌入式系统中是个问题？**
   - 内存泄漏：已分配的内存未被正确释放
   - 在嵌入式中的问题：资源有限、长时间运行（long uptimes）、没有虚拟内存（virtual memory）

2. **你会如何检测内存泄漏（memory leaks）？**
   - 静态分析工具（静态分析工具：编译器警告、静态分析器）
   - 动态分析工具（Valgrind、AddressSanitizer）
   - 自定义内存追踪

3. **内存泄漏的常见原因有哪些？**
   - 忘记释放已分配的内存
   - 没有清理的错误路径
   - 循环引用（Circular references）
   - 没有清理的异常处理

### 进阶问题（Advanced Questions）
1. **你会如何为嵌入式系统实现一个内存泄漏检测器（memory leak detector）？**
   - 低开销的轻量级追踪
   - 基于栈的分配追踪
   - 周期性泄漏检查（Periodic leak checks）

2. **有哪些策略可以预防内存泄漏（memory leaks）？**
   - RAII 模式
   - 智能指针（Smart pointers）
   - 内存池（Memory pools）
   - 一致的分配/释放模式

3. **你会如何应对实时系统（real-time systems）中的内存泄漏？**
   - 预先分配内存池（memory pools）
   - 尽可能使用静态分配（static allocation）
   - 实现轻量级泄漏检测

## 📚 附加资源（Additional Resources）

### 标准与文档（Standards and Documentation）
- **C 标准库（C Standard Library）**：内存管理函数（Memory management functions）
- **Valgrind 文档（Valgrind Documentation）**：内存错误检测（Memory error detection）
- **AddressSanitizer**：运行时内存错误检测器（Runtime memory error detector）

### 相关主题（Related Topics）
- [[Memory_Pool_Allocation]] —— 内存池分配，实现高效的内存管理
- [[Memory_Fragmentation]] —— 内存碎片化，了解内存碎片化问题
- [[Stack_Overflow_Prevention]] —— 栈溢出防护，栈管理
- [[Memory_Protection]] —— 内存保护，内存安全

### 工具与库（Tools and Libraries）
- **Valgrind**：全面的内存分析（Comprehensive memory analysis）
- **AddressSanitizer**：快速的内存错误检测（Fast memory error detection）
- **自定义内存追踪器（Custom memory trackers）**：嵌入式特有的解决方案

---

**下一主题：** [[Stack_Overflow_Prevention]] —— 栈溢出防护 → [[Memory_Protection]] —— 内存保护
