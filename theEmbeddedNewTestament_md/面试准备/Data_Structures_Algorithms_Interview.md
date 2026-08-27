---
tags:
  - 面试准备
  - 嵌入式面试
source: "Interview_Preparation/Foundation_Level/Data_Structures_Algorithms_Interview.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入
>
> 在网站上刷社区排名的题库、带 AI 反馈的编程练习，以及结构化的面试准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

# 🧮 数据结构与算法面试准备

## 🚀 **快速导航**
- [数据结构基础](#data-structures-fundamentals)
- [算法分析](#algorithm-analysis)
- [常见算法](#common-algorithms)
- [嵌入式特定考虑](#embedded-specific-considerations)
- [优化技巧](#optimization-techniques)

## 📚 **速查表：核心概念**
- **数据结构**：数组、链表、栈、队列、树、哈希表
- **算法复杂度**：时间复杂度（O 记法）、空间复杂度
- **搜索算法**：线性、二分、基于哈希的搜索
- **排序算法**：冒泡、插入、选择、归并、快速、堆排序
- **内存效率**：缓存友好结构、最小内存使用

## 🧮 **数据结构基础**

### **数组与内存布局**

**数组实现**：
```c
// 静态数组
int static_array[100];

// 动态数组
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} dynamic_array_t;

// 初始化动态数组
bool init_dynamic_array(dynamic_array_t *arr, size_t initial_capacity) {
    if (!arr || initial_capacity == 0) return false;
    
    arr->data = malloc(initial_capacity * sizeof(int));
    if (!arr->data) return false;
    
    arr->size = 0;
    arr->capacity = initial_capacity;
    return true;
}

// 向动态数组添加元素
bool add_element(dynamic_array_t *arr, int value) {
    if (!arr) return false;
    
    // 检查是否需要扩容
    if (arr->size >= arr->capacity) {
        size_t new_capacity = arr->capacity * 2;
        int *new_data = realloc(arr->data, new_capacity * sizeof(int));
        
        if (!new_data) return false;
        
        arr->data = new_data;
        arr->capacity = new_capacity;
    }
    
    // 添加元素
    arr->data[arr->size] = value;
    arr->size++;
    
    return true;
}

// 缓存友好的数组访问（行主序）
void matrix_multiply_cache_friendly(float *A, float *B, float *C, int N) {
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            float sum = 0.0f;
            for (int k = 0; k < N; k++) {
                // 以行主序访问元素以获得更好缓存性能
                sum += A[i * N + k] * B[k * N + j];
            }
            C[i * N + j] = sum;
        }
    }
}
```

**内存布局考虑**：
```
1. 缓存行大小
   - 典型缓存行：64 字节
   - 将数据结构对齐到缓存行
   - 最小化缓存未命中

2. 内存访问模式
   - 顺序访问比随机访问快
   - 行主序 vs 列主序
   - 结构数组（SoA）vs 数组结构（AoS）

3. 对齐
   - 为性能进行自然对齐
   - 填充考虑
   - 内存浪费 vs 性能的权衡
```

### **链表**

**单链表实现**：
```c
typedef struct node {
    int data;
    struct node *next;
} node_t;

typedef struct {
    node_t *head;
    node_t *tail;
    size_t size;
} linked_list_t;

// 创建新节点
node_t* create_node(int data) {
    node_t *new_node = malloc(sizeof(node_t));
    if (new_node) {
        new_node->data = data;
        new_node->next = NULL;
    }
    return new_node;
}

// 在开头插入
bool insert_at_beginning(linked_list_t *list, int data) {
    if (!list) return false;
    
    node_t *new_node = create_node(data);
    if (!new_node) return false;
    
    new_node->next = list->head;
    list->head = new_node;
    
    if (!list->tail) {
        list->tail = new_node;
    }
    
    list->size++;
    return true;
}

// 在末尾插入
bool insert_at_end(linked_list_t *list, int data) {
    if (!list) return false;
    
    node_t *new_node = create_node(data);
    if (!new_node) return false;
    
    if (!list->head) {
        list->head = new_node;
        list->tail = new_node;
    } else {
        list->tail->next = new_node;
        list->tail = new_node;
    }
    
    list->size++;
    return true;
}

// 查找元素
node_t* find_element(linked_list_t *list, int data) {
    if (!list) return NULL;
    
    node_t *current = list->head;
    while (current) {
        if (current->data == data) {
            return current;
        }
        current = current->next;
    }
    
    return NULL;
}

// 删除元素
bool delete_element(linked_list_t *list, int data) {
    if (!list || !list->head) return false;
    
    node_t *current = list->head;
    node_t *prev = NULL;
    
    // 查找要删除的元素
    while (current && current->data != data) {
        prev = current;
        current = current->next;
    }
    
    if (!current) return false;  // 未找到元素
    
    // 更新链表结构
    if (prev) {
        prev->next = current->next;
        if (!prev->next) {
            list->tail = prev;
        }
    } else {
        list->head = current->next;
        if (!list->head) {
            list->tail = NULL;
        }
    }
    
    // 释放内存
    free(current);
    list->size--;
    
    return true;
}
```

**双向链表**：
```c
typedef struct dnode {
    int data;
    struct dnode *prev;
    struct dnode *next;
} dnode_t;

typedef struct {
    dnode_t *head;
    dnode_t *tail;
    size_t size;
} doubly_linked_list_t;

// 按有序顺序插入（用于有序列表）
bool insert_sorted(doubly_linked_list_t *list, int data) {
    if (!list) return false;
    
    dnode_t *new_node = malloc(sizeof(dnode_t));
    if (!new_node) return false;
    
    new_node->data = data;
    
    // 空列表
    if (!list->head) {
        new_node->prev = NULL;
        new_node->next = NULL;
        list->head = new_node;
        list->tail = new_node;
        list->size++;
        return true;
    }
    
    // 查找插入位置
    dnode_t *current = list->head;
    while (current && current->data < data) {
        current = current->next;
    }
    
    // 在 current 之前插入
    if (current) {
        new_node->next = current;
        new_node->prev = current->prev;
        
        if (current->prev) {
            current->prev->next = new_node;
        } else {
            list->head = new_node;
        }
        
        current->prev = new_node;
    } else {
        // 在末尾插入
        new_node->next = NULL;
        new_node->prev = list->tail;
        list->tail->next = new_node;
        list->tail = new_node;
    }
    
    list->size++;
    return true;
}
```

### **栈与队列**

**栈实现**：
```c
typedef struct {
    int *data;
    size_t top;
    size_t capacity;
} stack_t;

// 初始化栈
bool init_stack(stack_t *stack, size_t capacity) {
    if (!stack || capacity == 0) return false;
    
    stack->data = malloc(capacity * sizeof(int));
    if (!stack->data) return false;
    
    stack->top = 0;
    stack->capacity = capacity;
    return true;
}

// 压入元素
bool push(stack_t *stack, int value) {
    if (!stack || stack->top >= stack->capacity) return false;
    
    stack->data[stack->top++] = value;
    return true;
}

// 弹出元素
bool pop(stack_t *stack, int *value) {
    if (!stack || !value || stack->top == 0) return false;
    
    *value = stack->data[--stack->top];
    return true;
}

// 查看栈顶元素
bool peek(stack_t *stack, int *value) {
    if (!stack || !value || stack->top == 0) return false;
    
    *value = stack->data[stack->top - 1];
    return true;
}

// 检查栈是否为空
bool is_stack_empty(stack_t *stack) {
    return (!stack || stack->top == 0);
}

// 检查栈是否已满
bool is_stack_full(stack_t *stack) {
    return (stack && stack->top >= stack->capacity);
}
```

**队列实现**：
```c
typedef struct {
    int *data;
    size_t front;
    size_t rear;
    size_t size;
    size_t capacity;
} queue_t;

// 初始化队列
bool init_queue(queue_t *queue, size_t capacity) {
    if (!queue || capacity == 0) return false;
    
    queue->data = malloc(capacity * sizeof(int));
    if (!queue->data) return false;
    
    queue->front = 0;
    queue->rear = 0;
    queue->size = 0;
    queue->capacity = capacity;
    return true;
}

// 入队元素
bool enqueue(queue_t *queue, int value) {
    if (!queue || queue->size >= queue->capacity) return false;
    
    queue->data[queue->rear] = value;
    queue->rear = (queue->rear + 1) % queue->capacity;
    queue->size++;
    
    return true;
}

// 出队元素
bool dequeue(queue_t *queue, int *value) {
    if (!queue || !value || queue->size == 0) return false;
    
    *value = queue->data[queue->front];
    queue->front = (queue->front + 1) % queue->capacity;
    queue->size--;
    
    return true;
}

// 查看队首元素
bool peek_front(queue_t *queue, int *value) {
    if (!queue || !value || queue->size == 0) return false;
    
    *value = queue->data[queue->front];
    return true;
}

// 检查队列是否为空
bool is_queue_empty(queue_t *queue) {
    return (!queue || queue->size == 0);
}

// 检查队列是否已满
bool is_queue_full(queue_t *queue) {
    return (queue && queue->size >= queue->capacity);
}
```

## 🧮 **算法分析**

### **时间复杂度分析**

**Big O 记法示例**：
```c
// O(1) - 常数时间
int get_first_element(int arr[], int size) {
    if (size > 0) return arr[0];
    return -1;
}

// O(n) - 线性时间
int find_element(int arr[], int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}

// O(n²) - 二次时间
void bubble_sort(int arr[], int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // 交换元素
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

// O(log n) - 对数时间
int binary_search(int arr[], int size, int target) {
    int left = 0;
    int right = size - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;
}

// O(n log n) - 线性对数时间
void merge_sort(int arr[], int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        
        merge_sort(arr, left, mid);
        merge_sort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}
```

**复杂度比较**：
```
算法          | 最好情况 | 平均情况 | 最坏情况 | 空间
--------------|---------|---------|---------|------
线性搜索     | O(1)    | O(n)    | O(n)    | O(1)
二分搜索     | O(1)    | O(log n)| O(log n)| O(1)
冒泡排序     | O(n)    | O(n²)   | O(n²)   | O(1)
插入排序     | O(n)    | O(n²)   | O(n²)   | O(1)
归并排序     | O(n log n)| O(n log n)| O(n log n)| O(n)
快速排序     | O(n log n)| O(n log n)| O(n²)  | O(log n)
堆排序       | O(n log n)| O(n log n)| O(n log n)| O(1)
```

### **空间复杂度分析**

**内存使用示例**：
```c
// O(1) - 常数空间
int sum_array(int arr[], int size) {
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += arr[i];
    }
    return sum;
}

// O(n) - 线性空间
int* copy_array(int arr[], int size) {
    int *copy = malloc(size * sizeof(int));
    if (!copy) return NULL;
    
    for (int i = 0; i < size; i++) {
        copy[i] = arr[i];
    }
    return copy;
}

// O(n²) - 二次空间
int** create_matrix(int rows, int cols) {
    int **matrix = malloc(rows * sizeof(int*));
    if (!matrix) return NULL;
    
    for (int i = 0; i < rows; i++) {
        matrix[i] = malloc(cols * sizeof(int));
        if (!matrix[i]) {
            // 失败时清理
            for (int j = 0; j < i; j++) {
                free(matrix[j]);
            }
            free(matrix);
            return NULL;
        }
    }
    
    return matrix;
}

// 递归空间复杂度
int factorial_recursive(int n) {
    if (n <= 1) return 1;
    return n * factorial_recursive(n - 1);
    // 空间复杂度：O(n) 由于调用栈
}

int factorial_iterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
    // 空间复杂度：O(1)
}
```

## 🧮 **常见算法**

### **搜索算法**

**带优化的线性搜索**：
```c
// 基本线性搜索
int linear_search(int arr[], int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}

// 带哨兵的线性搜索（消除边界检查）
int linear_search_sentinel(int arr[], int size, int target) {
    // 保存原始末元素
    int last = arr[size - 1];
    
    // 设置哨兵
    arr[size - 1] = target;
    
    int i = 0;
    while (arr[i] != target) {
        i++;
    }
    
    // 恢复原始元素
    arr[size - 1] = last;
    
    // 检查是否找到目标
    if (i < size - 1 || last == target) {
        return i;
    }
    
    return -1;
}

// 带提前终止的线性搜索
int linear_search_early_termination(int arr[], int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) return i;
        
        // 如果已越过目标应在位置则提前终止
        if (arr[i] > target) break;
    }
    return -1;
}
```

**二分搜索变体**：
```c
// 标准二分搜索
int binary_search(int arr[], int size, int target) {
    int left = 0;
    int right = size - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;
}

// 二分搜索找首次出现
int binary_search_first(int arr[], int size, int target) {
    int left = 0;
    int right = size - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            right = mid - 1;  // 继续向左搜索
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

// 二分搜索找最后一次出现
int binary_search_last(int arr[], int size, int target) {
    int left = 0;
    int right = size - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            left = mid + 1;  // 继续向右搜索
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

// 二分搜索找插入点
int binary_search_insertion_point(int arr[], int size, int target) {
    int left = 0;
    int right = size;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    
    return left;
}
```

### **排序算法**

**带优化的插入排序**：
```c
// 基本插入排序
void insertion_sort(int arr[], int size) {
    for (int i = 1; i < size; i++) {
        int key = arr[i];
        int j = i - 1;
        
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        
        arr[j + 1] = key;
    }
}

// 二分插入排序（用二分搜索找插入点）
void binary_insertion_sort(int arr[], int size) {
    for (int i = 1; i < size; i++) {
        int key = arr[i];
        
        // 用二分搜索找插入点
        int left = 0;
        int right = i;
        
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (arr[mid] <= key) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        
        // 移动元素腾出空间
        for (int j = i; j > left; j--) {
            arr[j] = arr[j - 1];
        }
        
        arr[left] = key;
    }
}

// 希尔排序（改进的插入排序）
void shell_sort(int arr[], int size) {
    // 生成间隔序列
    for (int gap = size / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < size; i++) {
            int key = arr[i];
            int j = i;
            
            while (j >= gap && arr[j - gap] > key) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            
            arr[j] = key;
        }
    }
}
```

**快速排序实现**：
```c
// 分区函数
int partition(int arr[], int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            // 交换 arr[i] 和 arr[j]
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    
    // 交换 arr[i+1] 和 arr[high]（枢轴）
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    
    return i + 1;
}

// 针对小子数组优化的快速排序
void quick_sort(int arr[], int low, int high) {
    if (low < high) {
        // 小子数组用插入排序
        if (high - low + 1 <= 10) {
            insertion_sort(&arr[low], high - low + 1);
            return;
        }
        
        int pi = partition(arr, low, high);
        
        quick_sort(arr, low, pi - 1);
        quick_sort(arr, pi + 1, high);
    }
}

// 三数取中选枢轴的快速排序
int median_of_three(int arr[], int low, int high) {
    int mid = low + (high - low) / 2;
    
    // 对 low、mid、high 排序
    if (arr[low] > arr[mid]) {
        int temp = arr[low];
        arr[low] = arr[mid];
        arr[mid] = temp;
    }
    if (arr[low] > arr[high]) {
        int temp = arr[low];
        arr[low] = arr[high];
        arr[high] = temp;
    }
    if (arr[mid] > arr[high]) {
        int temp = arr[mid];
        arr[mid] = arr[high];
        arr[high] = temp;
    }
    
    // 将枢轴移到 high-1
    int temp = arr[mid];
    arr[mid] = arr[high - 1];
    arr[high - 1] = temp;
    
    return arr[high - 1];
}
```

## 🧮 **嵌入式特定考虑**

### **内存约束**

**内存高效数据结构**：
```c
// 紧凑链表节点（最小开销）
typedef struct compact_node {
    uint16_t data;           // 2 字节
    uint16_t next_offset;    // 2 字节（从池起点的偏移）
} compact_node_t;

// 紧凑节点内存池
typedef struct {
    compact_node_t *pool;
    uint16_t free_list_head;
    uint16_t pool_size;
} compact_list_pool_t;

// 初始化池
bool init_compact_pool(compact_list_pool_t *pool, uint16_t size) {
    pool->pool = malloc(size * sizeof(compact_node_t));
    if (!pool->pool) return false;
    
    pool->pool_size = size;
    
    // 初始化空闲链表
    for (uint16_t i = 0; i < size - 1; i++) {
        pool->pool[i].next_offset = i + 1;
    }
    pool->pool[size - 1].next_offset = 0xFFFF;  // 结束标记
    pool->free_list_head = 0;
    
    return true;
}

// 从池中分配节点
uint16_t allocate_node(compact_list_pool_t *pool) {
    if (pool->free_list_head == 0xFFFF) return 0xFFFF;  // 池已满
    
    uint16_t node_index = pool->free_list_head;
    pool->free_list_head = pool->pool[node_index].next_offset;
    
    return node_index;
}

// 将节点归还到池
void free_node(compact_list_pool_t *pool, uint16_t node_index) {
    if (node_index >= pool->pool_size) return;
    
    pool->pool[node_index].next_offset = pool->free_list_head;
    pool->free_list_head = node_index;
}
```

**缓存优化数据结构**：
```c
// 结构数组（SoA）以获得更好缓存性能
typedef struct {
    float *temperatures;     // 所有温度放在一起
    float *humidities;       // 所有湿度放在一起
    uint32_t *timestamps;    // 所有时间戳放在一起
    size_t size;
    size_t capacity;
} sensor_data_soa_t;

// 数组结构（AoS）- 传统方法
typedef struct {
    float temperature;
    float humidity;
    uint32_t timestamp;
} sensor_data_aos_t;

// SoA 初始化
bool init_sensor_data_soa(sensor_data_soa_t *data, size_t capacity) {
    data->temperatures = malloc(capacity * sizeof(float));
    data->humidities = malloc(capacity * sizeof(float));
    data->timestamps = malloc(capacity * sizeof(uint32_t));
    
    if (!data->temperatures || !data->humidities || !data->timestamps) {
        // 失败时清理
        free(data->temperatures);
        free(data->humidities);
        free(data->timestamps);
        return false;
    }
    
    data->size = 0;
    data->capacity = capacity;
    return true;
}

// 缓存友好的处理
void process_sensor_data_soa(sensor_data_soa_t *data) {
    // 处理温度（顺序内存访问）
    for (size_t i = 0; i < data->size; i++) {
        data->temperatures[i] = apply_calibration(data->temperatures[i]);
    }
    
    // 处理湿度（顺序内存访问）
    for (size_t i = 0; i < data->size; i++) {
        data->humidities[i] = apply_filter(data->humidities[i]);
    }
}
```

### **实时考虑**

**确定性算法**：
```c
// 固定时间复杂度的确定性排序
void deterministic_sort(int arr[], int size) {
    // 对有界整数值使用计数排序
    // 时间复杂度：O(n + k)，其中 k 是值的范围
    // 空间复杂度：O(k)
    
    const int MAX_VALUE = 1000;  // 已知上界
    int count[MAX_VALUE + 1] = {0};
    
    // 统计出现次数
    for (int i = 0; i < size; i++) {
        count[arr[i]]++;
    }
    
    // 重建有序数组
    int index = 0;
    for (int i = 0; i <= MAX_VALUE; i++) {
        while (count[i] > 0) {
            arr[index++] = i;
            count[i]--;
        }
    }
}

// 带超时的有界搜索
int bounded_search(int arr[], int size, int target, uint32_t max_time_ms) {
    uint32_t start_time = get_system_time();
    
    for (int i = 0; i < size; i++) {
        // 检查超时
        if (get_system_time() - start_time > max_time_ms) {
            return -2;  // 超时指示
        }
        
        if (arr[i] == target) return i;
    }
    
    return -1;  // 未找到
}
```

## 🧮 **优化技巧**

### **算法优化**

**循环优化**：
```c
// 展开循环以获得更好性能
void matrix_multiply_unrolled(float *A, float *B, float *C, int N) {
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j += 4) {  // 一次处理 4 个元素
            float sum0 = 0.0f, sum1 = 0.0f, sum2 = 0.0f, sum3 = 0.0f;
            
            for (int k = 0; k < N; k++) {
                float a_val = A[i * N + k];
                sum0 += a_val * B[k * N + j];
                sum1 += a_val * B[k * N + j + 1];
                sum2 += a_val * B[k * N + j + 2];
                sum3 += a_val * B[k * N + j + 3];
            }
            
            C[i * N + j] = sum0;
            C[i * N + j + 1] = sum1;
            C[i * N + j + 2] = sum2;
            C[i * N + j + 3] = sum3;
        }
    }
}

// 缓存分块的矩阵乘法
void matrix_multiply_blocked(float *A, float *B, float *C, int N) {
    const int BLOCK_SIZE = 32;  // 针对缓存大小优化
    
    for (int i0 = 0; i0 < N; i0 += BLOCK_SIZE) {
        for (int j0 = 0; j0 < N; j0 += BLOCK_SIZE) {
            for (int k0 = 0; k0 < N; k0 += BLOCK_SIZE) {
                // 处理块
                for (int i = i0; i < MIN(i0 + BLOCK_SIZE, N); i++) {
                    for (int j = j0; j < MIN(j0 + BLOCK_SIZE, N); j++) {
                        float sum = 0.0f;
                        for (int k = k0; k < MIN(k0 + BLOCK_SIZE, N); k++) {
                            sum += A[i * N + k] * B[k * N + j];
                        }
                        C[i * N + j] += sum;
                    }
                }
            }
        }
    }
}
```

**内存访问优化**：
```c
// 为更好缓存性能预取数据
void prefetch_optimized_processing(int *data, int size) {
    for (int i = 0; i < size; i++) {
        // 预取下一个缓存行
        if (i + 16 < size) {
            __builtin_prefetch(&data[i + 16], 0, 3);  // 读，高局部性
        }
        
        // 处理当前元素
        process_element(data[i]);
    }
}

// 对齐内存分配
void* aligned_allocate(size_t size, size_t alignment) {
    void *ptr = NULL;
    
    #ifdef _MSC_VER
        ptr = _aligned_malloc(size, alignment);
    #else
        if (posix_memalign(&ptr, alignment, size) != 0) {
            ptr = NULL;
        }
    #endif
    
    return ptr;
}

// SIMD 优化操作
void vector_add_simd(float *a, float *b, float *result, int size) {
    // 确保 SIMD 操作的对齐
    assert(((uintptr_t)a & 0xF) == 0);
    assert(((uintptr_t)b & 0xF) == 0);
    assert(((uintptr_t)result & 0xF) == 0);
    
    for (int i = 0; i < size; i += 4) {
        // 加载向量
        __m128 va = _mm_load_ps(&a[i]);
        __m128 vb = _mm_load_ps(&b[i]);
        
        // 向量相加
        __m128 sum = _mm_add_ps(va, vb);
        
        // 存储结果
        _mm_store_ps(&result[i], sum);
    }
}
```

## 🧪 **常见面试问题**

### **问题 1：找出缺失的数字**

**问题**：给定一个范围 [1, n] 内 n-1 个整数的数组，找出缺失的数字。

**求解思路**：
```
1. 数学方法：前 n 个数之和减去数组之和
2. XOR 方法：将 1 到 n 的所有数与数组元素异或
3. 排序方法：排序后找空缺
4. 哈希表方法：标记已出现的数字
```

**最优解**：
```c
// 数学方法 - O(n) 时间，O(1) 空间
int find_missing_number_math(int arr[], int size) {
    int n = size + 1;
    int expected_sum = n * (n + 1) / 2;
    int actual_sum = 0;
    
    for (int i = 0; i < size; i++) {
        actual_sum += arr[i];
    }
    
    return expected_sum - actual_sum;
}

// XOR 方法 - O(n) 时间，O(1) 空间
int find_missing_number_xor(int arr[], int size) {
    int xor_result = 0;
    
    // 将 1 到 n 的所有数异或
    for (int i = 1; i <= size + 1; i++) {
        xor_result ^= i;
    }
    
    // 与数组元素异或
    for (int i = 0; i < size; i++) {
        xor_result ^= arr[i];
    }
    
    return xor_result;
}
```

### **问题 2：反转链表**

**问题**：反转单链表。

**求解思路**：
```
1. 迭代方法：使用三个指针
2. 递归方法：递归反转链表其余部分
3. 栈方法：将所有节点压栈，然后弹出反转
```

**迭代解**：
```c
node_t* reverse_linked_list_iterative(node_t *head) {
    node_t *prev = NULL;
    node_t *current = head;
    node_t *next = NULL;
    
    while (current != NULL) {
        // 保存下一节点
        next = current->next;
        
        // 反转当前节点指针
        current->next = prev;
        
        // 指针前移
        prev = current;
        current = next;
    }
    
    return prev;  // 新头节点
}

// 递归解
node_t* reverse_linked_list_recursive(node_t *head) {
    // 基本情况：空链表或单节点
    if (head == NULL || head->next == NULL) {
        return head;
    }
    
    // 递归反转其余部分
    node_t *rest = reverse_linked_list_recursive(head->next);
    
    // 反转当前节点
    head->next->next = head;
    head->next = NULL;
    
    return rest;
}
```

### **问题 3：检测链表中的环**

**问题**：检测链表是否有环。

**求解思路**：
```
1. 哈希表方法：存储已访问节点
2. Floyd 判环算法（龟兔赛跑）
3. 标记方法：标记已访问节点
```

**Floyd 算法**：
```c
bool has_cycle_floyd(node_t *head) {
    if (!head || !head->next) return false;
    
    node_t *slow = head;
    node_t *fast = head->next;
    
    while (slow != fast) {
        if (!fast || !fast->next) return false;
        
        slow = slow->next;        // 走一步
        fast = fast->next->next;  // 走两步
    }
    
    return true;  // 检测到环
}

// 找环的起点
node_t* find_cycle_start(node_t *head) {
    if (!head || !head->next) return NULL;
    
    // 找相遇点
    node_t *slow = head;
    node_t *fast = head;
    
    do {
        if (!fast || !fast->next) return NULL;
        slow = slow->next;
        fast = fast->next->next;
    } while (slow != fast);
    
    // 找环起点
    slow = head;
    while (slow != fast) {
        slow = slow->next;
        fast = fast->next;
    }
    
    return slow;
}
```

## 🧪 **练习题**

### **问题 1：高效字符串匹配**

**场景**：实现一个函数，在字符串中查找子串首次出现的位置。

**问题**：设计一个时间复杂度为 O(n+m) 的高效算法。

**预期分析**：
```
1. 算法选择：
   - 朴素方法：O(n*m) 时间
   - KMP 算法：O(n+m) 时间，O(m) 空间
   - Boyer-Moore：平均 O(n/m)

2. 实现：
   - 为 KMP 预处理模式
   - 用失效函数高效匹配
   - 处理边界情况（空字符串、无匹配）

3. 优化：
   - 提前终止
   - 内存高效预处理
   - 缓存友好的访问模式
```

### **问题 2：内存高效队列**

**场景**：设计一个能以最小内存开销处理数百万个元素的队列。

**问题**：实现一个自动扩容的环形缓冲区。

**预期分析**：
```
1. 数据结构设计：
   - 动态大小的环形缓冲区
   - 高效内存分配策略
   - 最小内存碎片

2. 性能考虑：
   - O(1) 入队/出队操作
   - 高效扩容（摊还 O(1)）
   - 内存使用优化

3. 实现细节：
   - 处理回绕条件
   - 高效容量倍增
   - 扩容时的内存清理
```

## ✅ **自我评估清单**

### **数据结构** ✅
- [ ] 能实现数组和动态数组
- [ ] 能实现链表（单链表和双向链表）
- [ ] 能实现栈和队列
- [ ] 能实现树和哈希表

### **算法分析** ✅
- [ ] 能用 Big O 记法分析时间复杂度
- [ ] 能分析空间复杂度
- [ ] 能比较算法效率
- [ ] 能针对更好性能优化算法

### **搜索与排序** ✅
- [ ] 能实现线性和二分搜索
- [ ] 能实现基本排序算法
- [ ] 能针对特定情况优化搜索和排序
- [ ] 能处理边界情况和错误条件

### **嵌入式优化** ✅
- [ ] 能设计内存高效的数据结构
- [ ] 能针对缓存性能优化
- [ ] 能实现确定性算法
- [ ] 能在内存使用与性能之间平衡

### **问题求解** ✅
- [ ] 能分解复杂问题
- [ ] 能选择合适的数据结构
- [ ] 能实现高效方案
- [ ] 能优化和重构代码

## 🔗 **相关主题**
- [[C_Programming_Interview]]
- [[RTOS_Interview]]
- [[Performance_Optimization_Interview]]
- [[System_Integration_Interview]]

## 📚 **附加资源**
- **算法可视化**：[VisuAlgo](https://visualgo.net/)
- **练习问题**：[LeetCode](https://leetcode.com/)、[HackerRank](https://www.hackerrank.com/)
- **算法分析**：[Big-O 速查表](https://www.bigocheatsheet.com/)
- **书籍**：《Introduction to Algorithms》作者 CLRS、《Algorithms》作者 Sedgewick

## 相关页面

- [[C_Programming_Interview]]
- [[RTOS_Interview]]
- [[Problem_Solving_Approach]]
- [[Basic_Hardware_Interview]]
- [[00-索引]]

返回索引 [[00-索引]]
