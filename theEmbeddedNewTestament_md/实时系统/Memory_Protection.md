---
tags:
  - 实时系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Real_Time_Systems/Memory_Protection.md
created: 2026-08-27
---

# 实时系统中的内存保护

> **在嵌入式实时系统中实现内存保护单元 (MPU) 以实现任务隔离、内存安全和系统安全的综合指南，包含 FreeRTOS 示例**

## 🎯 **概念 → 为什么重要 → 最小示例 → 动手试试 → 要点**

### **概念**
内存保护就像在建筑物的不同门口安排保安。与其让任何人访问任何房间，不如让每个人（任务）只能访问分配给他们的区域。如果有人试图闯入受限区域，安全系统（MPU）会立即阻止他们并发出警报。

### **为什么重要**
在嵌入式系统中，一个任务中的单个 bug 就可能破坏其他任务使用的内存，导致整个系统崩溃。内存保护在任务之间创建"防火墙"，因此即使某个任务崩溃或有 bug，也无法拖垮整个系统。这对于系统故障可能很危险的安全关键型应用尤为重要。

### **最小示例**
```c
// Configure MPU region for task stack protection
void configure_task_protection(TaskHandle_t task, uint8_t region_num) {
    // Get task stack boundaries
    uint32_t *stack_start = pxTaskGetStackStart(task);
    uint32_t stack_size = uxTaskGetStackHighWaterMark(task) * sizeof(StackType_t);
    
    // Configure MPU region
    MPU_Region_InitTypeDef region;
    region.Number = region_num;
    region.BaseAddress = (uint32_t)stack_start;
    region.Size = MPU_REGION_SIZE_1KB;  // Adjust based on actual size
    region.AccessPermission = MPU_REGION_PRIV_RO;  // Read-only for other tasks
    region.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
    region.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
    
    // Enable the region
    HAL_MPU_ConfigRegion(&region);
}

// Task creation with protection
void create_protected_task(void) {
    TaskHandle_t task_handle;
    xTaskCreate(task_function, "Protected", 128, NULL, 1, &task_handle);
    
    // Configure memory protection for this task
    configure_task_protection(task_handle, 1);
}
```

### **动手试试**
- **实验**：为不同任务设置 MPU 区域，并尝试访问受保护的内存
- **挑战**：实现一个关键任务与非关键任务完全隔离的系统
- **调试**：使用 MPU 故障处理程序来检测和记录内存访问违规

### **要点**
内存保护将你的系统从"人人都有权"（任何 bug 都可能让一切崩溃）转变为一个健壮、容错的系统，其中问题被限制和隔离。

---

## 📋 **目录**
- [概述](#overview)
- [MPU 基础](#mpu-fundamentals)
- [任务隔离策略](#task-isolation-strategies)
- [内存区域配置](#memory-region-configuration)
- [实现示例](#implementation-examples)
- [安全考虑](#security-considerations)
- [最佳实践](#best-practices)
- [面试题](#interview-questions)

---

## 🎯 **概述**

内存保护单元 (MPU) 提供硬件强制的任务间内存隔离，防止未经授权的内存访问并确保系统可靠性。在实时系统中，MPU 对于创建安全、容错的应用至关重要，此时任务失败不会损害系统完整性。

### **关键概念**
- **内存保护单元 (MPU)** - 硬件内存访问控制
- **任务隔离(Task Isolation)** - 防止任务访问彼此的内存
- **内存区域(Memory Regions)** - 具有特定权限的可配置内存区域
- **访问控制(Access Control)** - 内存区域的读、写和执行权限
- **故障处理(Fault Handling)** - 响应内存访问违规

---

## 🛡️ **MPU 基础**

### **什么是 MPU？**

内存保护单元是一种在运行时强制执行内存访问权限的硬件组件。与提供虚拟内存的内存管理单元 (MMU) 不同，MPU 处理物理地址，并提供更简单但有效的内存保护。

**MPU 与 MMU 对比：**
- **MPU**：物理地址，有限区域（8-16 个），简单权限
- **MMU**：虚拟地址，无限页面，复杂内存管理

### **MPU 架构**

**核心组件：**
- **区域寄存器(Region Registers)**：定义内存区域及其权限
- **权限逻辑(Permission Logic)**：在内存操作期间强制执行访问权限
- **故障检测(Fault Detection)**：在访问违规时触发异常
- **区域选择(Region Selection)**：为每次内存访问选择合适的区域

**内存访问流程：**
```
CPU Memory Request → MPU Region Check → Permission Validation → Access Granted/Denied
```

### **RTOS 中 MPU 的好处**

**1. 任务隔离：**
- 防止任务 A 破坏任务 B 的内存
- 隔离关键系统组件
- 创建安全的执行环境

**2. 故障隔离：**
- 限制软件 bug 的影响
- 防止栈溢出损害
- 防止缓冲区溢出

**3. 安全增强：**
- 防止未经授权的内存访问
- 保护敏感数据结构
- 创建受信任的执行区域

---

## 🔒 **任务隔离策略**

### **1. 栈隔离**

**工作原理：**
- 每个任务获得专用的栈内存区域
- MPU 防止访问其他任务栈
- 栈溢出触发 MPU 故障而不是内存破坏

**实现示例：**
```c
typedef struct {
    uint32_t stack_start;
    uint32_t stack_size;
    uint8_t region_number;
    MPU_Region_InitTypeDef mpu_region;
} task_stack_protection_t;

void vConfigureTaskStackProtection(TaskHandle_t task_handle, uint8_t region_num) {
    task_stack_protection_t *protection = pvPortMalloc(sizeof(task_stack_protection_t));
    
    if (protection != NULL) {
        // Get task stack information
        UBaseType_t stack_size = uxTaskGetStackHighWaterMark(task_handle);
        uint32_t *stack_ptr = (uint32_t*)pxTaskGetStackStart(task_handle);
        
        protection->stack_start = (uint32_t)stack_ptr;
        protection->stack_size = stack_size * sizeof(StackType_t);
        protection->region_number = region_num;
        
        // Configure MPU region for stack protection
        protection->mpu_region.Number = region_num;
        protection->mpu_region.BaseAddress = protection->stack_start;
        protection->mpu_region.Size = vCalculateMPUSize(protection->stack_size);
        protection->mpu_region.AccessPermission = MPU_REGION_FULL_ACCESS;
        protection->mpu_region.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
        protection->mpu_region.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
        protection->mpu_region.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
        protection->mpu_region.Number = MPU_REGION_NUMBER0;
        protection->mpu_region.TypeExtField = MPU_TEX_LEVEL0;
        protection->mpu_region.SubRegionDisable = 0x00;
        protection->mpu_region.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;
        
        // Apply MPU configuration
        HAL_MPU_ConfigRegion(&protection->mpu_region);
    }
}
```

### **2. 数据结构隔离**

**工作原理：**
- 关键数据结构位于受保护的内存区域
- 任务只能访问分配给它们的区域
- 共享数据位于特殊配置的区域

**实现示例：**
```c
typedef struct {
    uint32_t data_start;
    uint32_t data_size;
    uint8_t access_permissions;
    bool is_shared;
} data_protection_t;

void vConfigureDataProtection(uint8_t region_num, uint32_t start_addr, 
                            uint32_t size, uint8_t permissions, bool shared) {
    MPU_Region_InitTypeDef mpu_region;
    
    mpu_region.Number = region_num;
    mpu_region.BaseAddress = start_addr;
    mpu_region.Size = vCalculateMPUSize(size);
    
    if (shared) {
        mpu_region.AccessPermission = MPU_REGION_FULL_ACCESS;
        mpu_region.IsShareable = MPU_ACCESS_SHAREABLE;
    } else {
        mpu_region.AccessPermission = permissions;
        mpu_region.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
    }
    
    mpu_region.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
    mpu_region.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
    mpu_region.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;
    mpu_region.TypeExtField = MPU_TEX_LEVEL0;
    mpu_region.SubRegionDisable = 0x00;
    
    HAL_MPU_ConfigRegion(&mpu_region);
}
```

### **3. 代码保护**

**工作原理：**
- 可执行代码位于只读内存区域
- 防止运行时修改代码
- 分离代码和数据区域

**实现示例：**
```c
void vConfigureCodeProtection(uint8_t region_num, uint32_t code_start, uint32_t code_size) {
    MPU_Region_InitTypeDef mpu_region;
    
    mpu_region.Number = region_num;
    mpu_region.BaseAddress = code_start;
    mpu_region.Size = vCalculateMPUSize(code_size);
    mpu_region.AccessPermission = MPU_REGION_PRIVILEGED_READ_ONLY;
    mpu_region.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
    mpu_region.IsCacheable = MPU_ACCESS_CACHEABLE;
    mpu_region.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
    mpu_region.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;
    mpu_region.TypeExtField = MPU_TEX_LEVEL0;
    mpu_region.SubRegionDisable = 0x00;
    
    HAL_MPU_ConfigRegion(&mpu_region);
}
```

---

## 🗺️ **内存区域配置**

### **MPU 区域类型**

**1. 特权访问区域：**
- 只有特权任务可以访问
- 用于系统关键数据
- 内核和驱动程序内存区域

**2. 用户访问区域：**
- 特权任务和用户任务都可以访问
- 用于共享数据结构
- 通信缓冲区

**3. 只读区域：**
- 数据无法被修改
- 用于常量和配置
- 代码段

**4. 只执行区域：**
- 可以执行代码但无法读取
- 用于专有算法
- 安全敏感代码

### **区域大小计算**

**MPU 大小值：**
```c
uint32_t vCalculateMPUSize(uint32_t size_bytes) {
    if (size_bytes <= 32) return MPU_REGION_SIZE_32B;
    if (size_bytes <= 64) return MPU_REGION_SIZE_64B;
    if (size_bytes <= 128) return MPU_REGION_SIZE_128B;
    if (size_bytes <= 256) return MPU_REGION_SIZE_256B;
    if (size_bytes <= 512) return MPU_REGION_SIZE_512B;
    if (size_bytes <= 1*1024) return MPU_REGION_SIZE_1KB;
    if (size_bytes <= 2*1024) return MPU_REGION_SIZE_2KB;
    if (size_bytes <= 4*1024) return MPU_REGION_SIZE_4KB;
    if (size_bytes <= 8*1024) return MPU_REGION_SIZE_8KB;
    if (size_bytes <= 16*1024) return MPU_REGION_SIZE_16KB;
    if (size_bytes <= 32*1024) return MPU_REGION_SIZE_32KB;
    if (size_bytes <= 64*1024) return MPU_REGION_SIZE_64KB;
    if (size_bytes <= 128*1024) return MPU_REGION_SIZE_128KB;
    if (size_bytes <= 256*1024) return MPU_REGION_SIZE_256KB;
    if (size_bytes <= 512*1024) return MPU_REGION_SIZE_512KB;
    if (size_bytes <= 1*1024*1024) return MPU_REGION_SIZE_1MB;
    if (size_bytes <= 2*1024*1024) return MPU_REGION_SIZE_2MB;
    if (size_bytes <= 4*1024*1024) return MPU_REGION_SIZE_4MB;
    if (size_bytes <= 8*1024*1024) return MPU_REGION_SIZE_8MB;
    if (size_bytes <= 16*1024*1024) return MPU_REGION_SIZE_16MB;
    if (size_bytes <= 32*1024*1024) return MPU_REGION_SIZE_32MB;
    if (size_bytes <= 64*1024*1024) return MPU_REGION_SIZE_64MB;
    if (size_bytes <= 128*1024*1024) return MPU_REGION_SIZE_128MB;
    if (size_bytes <= 256*1024*1024) return MPU_REGION_SIZE_256MB;
    if (size_bytes <= 512*1024*1024) return MPU_REGION_SIZE_512MB;
    if (size_bytes <= 1*1024*1024*1024) return MPU_REGION_SIZE_1GB;
    if (size_bytes <= 2*1024*1024*1024) return MPU_REGION_SIZE_2GB;
    if (size_bytes <= 4*1024*1024*1024) return MPU_REGION_SIZE_4GB;
    
    return MPU_REGION_SIZE_4GB; // Maximum size
}
```

### **区域优先级与重叠**

**区域优先级规则：**
- 较低区域编号具有较高优先级
- 重叠区域使用较高优先级区域
- 子区域可以禁用特定区域

**子区域配置：**
```c
void vConfigureSubRegions(uint8_t region_num, uint32_t sub_region_mask) {
    MPU_Region_InitTypeDef mpu_region;
    
    // Get current region configuration
    HAL_MPU_GetRegionConfig(&mpu_region, region_num);
    
    // Configure sub-regions
    mpu_region.SubRegionDisable = sub_region_mask;
    
    // Reapply configuration
    HAL_MPU_ConfigRegion(&mpu_region);
}
```

---

## 💻 **实现示例**

### **完整的 MPU 管理系统**

```c
typedef struct {
    uint8_t region_count;
    MPU_Region_InitTypeDef regions[16];
    bool regions_enabled[16];
} mpu_manager_t;

mpu_manager_t g_mpu_manager = {0};

void vInitializeMPUManager(void) {
    // Enable MPU
    HAL_MPU_Enable(MPU_PRIVILEGED_DEFAULT);
    
    // Initialize region tracking
    memset(&g_mpu_manager, 0, sizeof(mpu_manager_t));
    
    printf("MPU Manager initialized\n");
}

uint8_t vAllocateMPURegion(void) {
    for (uint8_t i = 0; i < 16; i++) {
        if (!g_mpu_manager.regions_enabled[i]) {
            g_mpu_manager.regions_enabled[i] = true;
            g_mpu_manager.region_count++;
            return i;
        }
    }
    return 0xFF; // No free regions
}

void vFreeMPURegion(uint8_t region_num) {
    if (region_num < 16 && g_mpu_manager.regions_enabled[region_num]) {
        // Disable region
        HAL_MPU_DisableRegion(region_num);
        g_mpu_manager.regions_enabled[region_num] = false;
        g_mpu_manager.region_count--;
    }
}
```

### **任务特定的内存保护**

```c
typedef struct {
    TaskHandle_t task_handle;
    uint8_t stack_region;
    uint8_t data_region;
    uint8_t code_region;
    bool protection_enabled;
} task_memory_protection_t;

task_memory_protection_t task_protection[10];

void vEnableTaskMemoryProtection(TaskHandle_t task_handle) {
    // Find free slot
    int slot = -1;
    for (int i = 0; i < 10; i++) {
        if (task_protection[i].task_handle == NULL) {
            slot = i;
            break;
        }
    }
    
    if (slot >= 0) {
        task_protection[slot].task_handle = task_handle;
        
        // Allocate MPU regions
        task_protection[slot].stack_region = vAllocateMPURegion();
        task_protection[slot].data_region = vAllocateMPURegion();
        task_protection[slot].code_region = vAllocateMPURegion();
        
        // Configure protection
        vConfigureTaskStackProtection(task_handle, task_protection[slot].stack_region);
        
        task_protection[slot].protection_enabled = true;
        printf("Memory protection enabled for task\n");
    }
}
```

### **MPU 故障处理程序**

```c
void MemManage_Handler(void) {
    // Get fault information
    uint32_t mmfar = SCB->MMFAR;
    uint32_t cfsr = SCB->CFSR;
    uint32_t mmfault = (cfsr >> 7) & 0x1;
    uint32_t daccviol = (cfsr >> 1) & 0x1;
    uint32_t mmarvalid = (cfsr >> 7) & 0x1;
    
    printf("MPU Fault Detected!\n");
    printf("MMAR: 0x%08lx\n", mmfar);
    printf("MMFault: %lu\n", mmfault);
    printf("DACCViol: %lu\n", daccviol);
    printf("MMARValid: %lu\n", mmarvalid);
    
    // Get current task information
    TaskHandle_t current_task = xTaskGetCurrentTaskHandle();
    if (current_task != NULL) {
        printf("Fault in task: %s\n", pcTaskGetName(current_task));
    }
    
    // Handle fault based on type
    if (daccviol) {
        // Data access violation - terminate task
        printf("Data access violation - terminating task\n");
        vTaskDelete(current_task);
    } else if (mmfault) {
        // Memory management fault - log and continue
        printf("Memory management fault - logging\n");
    }
    
    // Clear fault flags
    SCB->CFSR = cfsr;
}
```

---

## 🔐 **安全考虑**

### **特权级别管理**

**特权分离：**
- 内核以特权模式运行
- 用户任务以非特权模式运行
- MPU 强制基于特权的访问

**实现：**
```c
void vSwitchToUserMode(void) {
    // Configure MPU for user mode
    __set_CONTROL(0x02); // User mode, no FPU
    
    // ISB to ensure context switch
    __ISB();
}

void vSwitchToPrivilegedMode(void) {
    // Configure MPU for privileged mode
    __set_CONTROL(0x00); // Privileged mode, no FPU
    
    // ISB to ensure context switch
    __ISB();
}
```

### **安全数据处理**

**敏感数据保护：**
- 加密密钥位于受保护区域
- 认证数据位于隔离内存
- 安全通信缓冲区

**实现：**
```c
void vProtectSensitiveData(uint32_t data_start, uint32_t data_size) {
    uint8_t region_num = vAllocateMPURegion();
    
    MPU_Region_InitTypeDef mpu_region;
    mpu_region.Number = region_num;
    mpu_region.BaseAddress = data_start;
    mpu_region.Size = vCalculateMPUSize(data_size);
    mpu_region.AccessPermission = MPU_REGION_PRIVILEGED_READ_WRITE;
    mpu_region.IsBufferable = MPU_ACCESS_NOT_BUFFERABLE;
    mpu_region.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
    mpu_region.IsShareable = MPU_ACCESS_NOT_SHAREABLE;
    mpu_region.DisableExec = MPU_INSTRUCTION_ACCESS_ENABLE;
    
    HAL_MPU_ConfigRegion(&mpu_region);
}
```

---

## ✅ **最佳实践**

### **设计原则**

1. **最小化区域使用**
   - 高效使用区域
   - 分组相关内存区域
   - 仔细规划区域分配

2. **一致的权限模型**
   - 定义清晰的访问规则
   - 记录权限要求
   - 在整个系统中一致应用

3. **故障处理策略**
   - 规划对 MPU 故障的响应
   - 实现优雅降级
   - 记录并监控违规

### **实现指南**

1. **区域配置**
   - 从保守权限开始
   - 生产前彻底测试
   - 监控性能影响

2. **任务管理**
   - 在任务创建期间配置保护
   - 在任务删除时清理区域
   - 处理动态内存分配

3. **调试支持**
   - 实现全面的故障处理程序
   - 提供调试信息
   - 支持运行时配置

---

## 🔬 **引导实验**

### **实验 1：基本 MPU 配置**
**目标**：设置用于任务隔离的基本 MPU 区域
**步骤**：
1. 启用 MPU 并配置基本区域
2. 创建具有不同内存访问权限的任务
3. 测试内存访问违规
4. 实现 MPU 故障处理程序

**预期结果**：任务被隔离，内存违规被捕获

### **实验 2：任务栈保护**
**目标**：保护各个任务栈免受破坏
**步骤**：
1. 为每个任务栈配置 MPU 区域
2. 实现栈溢出检测
3. 测试栈破坏场景
4. 验证故障隔离

**预期结果**：栈溢出被限制，不影响其他任务

### **实验 3：关键资源保护**
**目标**：保护系统关键资源和数据
**步骤**：
1. 识别关键系统资源
2. 配置具有合适权限的 MPU 区域
3. 在不同条件下测试访问控制
4. 实现优雅的故障处理

**预期结果**：关键资源免受未经授权的访问

---

## ✅ **自测**

### **理解检查**
- [ ] 你能解释为什么内存保护在实时系统中很重要吗？
- [ ] 你理解 MPU 与 MMU 的区别吗？
- [ ] 你能识别哪些内存区域需要保护吗？
- [ ] 你知道如何处理 MPU 故障吗？

### **实践技能检查**
- [ ] 你能为基本任务隔离配置 MPU 区域吗？
- [ ] 你知道如何保护任务栈免受破坏吗？
- [ ] 你能实现 MPU 故障处理程序吗？
- [ ] 你理解如何平衡保护与性能吗？

### **进阶概念检查**
- [ ] 你能解释如何实现动态内存保护吗？
- [ ] 你理解 MPU 区域配置中的权衡吗？
- [ ] 你能设计一个全面的内存保护策略吗？
- [ ] 你知道如何调试 MPU 相关问题吗？

---

## 🔗 **交叉链接**

### **相关主题**
- **[[FreeRTOS_Basics]]** - 理解 RTOS 上下文
- **[[Task_Creation_Management]]** - 保护任务资源
- **[[Real_Time_Debugging]]** - 调试内存保护问题
- **[[Memory_Management]]** - 理解内存概念

### **前置知识**
- **[[C_Language_Fundamentals]]** - 基础编程概念
- **[[Pointers_Memory_Addresses]]** - 内存概念
- **[[GPIO_Configuration]]** - 基础 I/O 设置

### **下一步**
- **[[Performance_Monitoring]]** - 监控内存保护性能
- **[[Power_Management]]** - MPU 的功耗考虑
- **[[Response_Time_Analysis]]** - 分析保护开销

---

## 📋 **速查表：关键要点**

### **内存保护基础**
- **目的**：硬件强制的内存隔离和访问控制
- **类型**：MPU（内存保护单元）、MMU（内存管理单元）
- **特性**：基于区域、权限控制、故障检测
- **好处**：任务隔离、故障隔离、安全增强

### **MPU 架构**
- **区域寄存器(Region Registers)**：定义内存区域及其权限
- **权限逻辑(Permission Logic)**：内存操作期间强制执行访问权限
- **故障检测(Fault Detection)**：访问违规时触发异常
- **区域选择(Region Selection)**：为每次内存访问选择合适的区域

### **任务隔离策略**
- **栈隔离(Stack Isolation)**：每个任务获得专用、受保护的栈区域
- **数据隔离(Data Isolation)**：为任务特定数据分隔内存区域
- **代码保护(Code Protection)**：为关键代码设置只执行区域
- **资源隔离(Resource Isolation)**：对共享资源的受保护访问

### **实现考虑**
- **区域限制(Region Limits)**：大多数 MPU 支持 8-16 个内存区域
- **性能影响(Performance Impact)**：MPU 检查增加极小的内存访问开销
- **配置(Configuration)**：区域在使用前必须配置
- **故障处理(Fault Handling)**：必须为访问违规实现处理程序

---

## ❓ **面试题**

### **基础概念**

1. **什么是 MPU，它与 MMU 有何区别？**
   - MPU 提供硬件内存保护
   - 处理物理地址
   - 区域数量有限
   - 比 MMU 简单

2. **如何使用 MPU 实现任务隔离？**
   - 每个任务有专用内存区域
   - 栈和数据保护
   - 访问权限强制
   - 故障处理

3. **MPU 区域的主要类型有哪些？**
   - 特权访问区域
   - 用户访问区域
   - 只读区域
   - 只执行区域

### **进阶主题**

1. **如何在实时系统中处理 MPU 故障？**
   - 实现故障处理程序
   - 记录违规信息
   - 终止违规任务
   - 保持系统稳定性

2. **解释 MPU 区域优先级和重叠处理。**
   - 较低编号具有较高优先级
   - 重叠区域使用优先级规则
   - 子区域用于精细控制

3. **如何使用 MPU 实现安全数据处理？**
   - 受保护的内存区域
   - 基于特权的访问
   - 安全通信缓冲区
   - 加密密钥保护

### **实际场景**

1. **设计一个基于 MPU 的任务隔离系统。**
   - 规划内存区域
   - 实现保护逻辑
   - 处理动态分配
   - 管理故障条件

2. **如何在嵌入式系统中保护敏感数据？**
   - 识别敏感数据
   - 配置受保护区域
   - 实现访问控制
   - 监控违规

3. **解释多任务 RTOS 的 MPU 配置。**
   - 区域分配策略
   - 权限管理
   - 动态配置
   - 性能优化

这份全面的内存保护文档为嵌入式工程师提供了实现使用 MPU 的健壮内存保护系统所需的理论基础、实践实现示例和最佳实践。
