---
tags:
  - 系统集成
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/System_Integration/Watchdog_Timers_System_Recovery.md
created: 2026-08-27
---

# 看门狗定时器与系统恢复(Watchdog Timers and System Recovery)

## 快速参考：关键要点(Quick Reference: Key Facts)

- **看门狗定时器(Watchdog timers)** 监控系统健康，并在系统无响应时自动触发恢复
- **硬件看门狗(Hardware watchdogs)** 在硅片中实现，相比软件看门狗提供最高的可靠性
- **窗口看门狗(Windowed watchdogs)** 防止过早和过晚的喂狗，确保正确的时间纪律
- **恢复层次(Recovery hierarchy)** 从软复位（任务重启）到冷复位（完整系统重启）
- **喂狗策略(Petting strategy)** 应有目的性并反映实际系统健康，而不仅仅是维护定时器
- **恢复前的错误日志记录(Error logging)** 为恢复后的分析提供关键的诊断信息
- **安全模式(Safe mode)** 在正常恢复失败时提供最小功能
- **看门狗超时(Watchdog timeout)** 必须在响应性和防止误触发之间取得平衡

## 概述(Overview)
看门狗定时器是嵌入式系统中的关键安全机制，用于监控系统健康，并在系统无响应或进入错误状态时自动触发恢复操作。本指南涵盖看门狗定时器实现、系统恢复策略，以及构建健壮嵌入式系统的最佳实践。

## 目录(Table of Contents)
1. [核心概念(Core Concepts)](#core-concepts)
2. [看门狗定时器类型(Watchdog Timer Types)](#watchdog-timer-types)
3. [看门狗实现(Watchdog Implementation)](#watchdog-implementation)
4. [系统恢复策略(System Recovery Strategies)](#system-recovery-strategies)
5. [错误检测与处理(Error Detection and Handling)](#error-detection-and-handling)
6. [恢复机制(Recovery Mechanisms)](#recovery-mechanisms)
7. [实现示例(Implementation Examples)](#implementation-examples)
8. [常见陷阱(Common Pitfalls)](#common-pitfalls)
9. [最佳实践(Best Practices)](#best-practices)
10. [面试问题(Interview Questions)](#interview-questions)

---

## 核心概念(Core Concepts)

### 什么是看门狗定时器?(What are Watchdog Timers?)
看门狗定时器是硬件或软件定时器，能够：
- **监控系统健康(Monitor System Health)**：跟踪系统是否正常运行
- **检测故障(Detect Failures)**：识别系统何时变得无响应
- **触发恢复(Trigger Recovery)**：自动重启或恢复系统
- **防止死锁(Prevent Lockups)**：确保系统不会保持在故障状态

### 看门狗定时器操作(Watchdog Timer Operation)
```
Watchdog Operation Cycle:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   System    │───▶│   Watchdog  │───▶│   Recovery  │
│  Running    │    │   Timer     │    │   Action    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Kick      │    │   Countdown │    │   System    │
│  Watchdog   │    │   Timer     │    │   Restart   │
└─────────────┘    └─────────────┘    └─────────────┘
```

### 恢复层次(Recovery Hierarchy)
```
System Recovery Hierarchy
┌─────────────────────────────────────────────────────────────┐
│ Level 1: Soft Reset                                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Restart application tasks                               │ │
│ │ Reinitialize peripherals                                │ │
│ │ Preserve critical data                                  │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ Level 2: Warm Reset                                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Restart RTOS and tasks                                  │ │
│ │ Reinitialize hardware                                   │ │
│ │ Load configuration from non-volatile memory             │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ Level 3: Cold Reset                                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Complete system restart                                 │ │
│ │ Full hardware initialization                            │ │
│ │ Load firmware from bootloader                           │ │
│ └─────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│ Level 4: Safe Mode                                         │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Minimal functionality                                   │ │
│ │ Error reporting and diagnostics                         │ │
│ │ Manual intervention required                            │ │
│ └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 窗口看门狗时序(Windowed Watchdog Timing)
```
Windowed Watchdog Timing
┌─────────────────────────────────────────────────────────────┐
│ Timeout Period (e.g., 1000ms)                             │
│ ┌─────────┬─────────────┬─────────┬─────────────────────┐ │
│ │ 0ms     │ 200ms       │ 800ms   │ 1000ms              │ │
│ │         │ Window      │ Window  │                     │ │
│ │ Invalid │ Start       │ End     │ Invalid              │ │
│ │ Kick    │             │         │ Kick                 │ │
│ └─────────┴─────────────┴─────────┴─────────────────────┘ │
│         │             │             │                     │
│         ▼             ▼             ▼                     │
│     Early Kick    Valid Kick   Late Kick                 │
│     (Violation)   (Success)    (Violation)               │
└─────────────────────────────────────────────────────────────┘
```

### 错误检测与恢复流程(Error Detection and Recovery Flow)
```
Error Detection and Recovery Flow
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Error       │───▶│ Error       │───▶│ Recovery    │
│ Detection   │    │ Analysis    │    │ Strategy    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Watchdog    │    │ Error       │    │ Execute     │
│ Timeout     │    │ Logging     │    │ Recovery    │
└─────────────┘    └─────────────┘    └─────────────┘
                                                    │
                                                    ▼
                                            ┌─────────────┐
                                            │ Verify      │
                                            │ Recovery    │
                                            └─────────────┘
```
---

## 看门狗定时器类型(Watchdog Timer Types)

### 硬件看门狗定时器(Hardware Watchdog Timers)
硬件看门狗在硅片中实现，提供最高的可靠性：

```c
// Hardware watchdog configuration
typedef struct {
    uint32_t timeout_ms;               // Watchdog timeout in milliseconds
    uint32_t window_start_ms;          // Window start time (for windowed mode)
    uint32_t window_end_ms;            // Window end time (for windowed mode)
    bool     window_mode_enabled;      // Enable windowed operation
    bool     debug_mode_enabled;       // Enable debug mode (stops during debug)
    uint32_t prescaler;                // Clock prescaler value
} hw_watchdog_config_t;

// Hardware watchdog registers (example for STM32)
#define IWDG_KR_KEY_RELOAD            0xAAAA
#define IWDG_KR_KEY_ENABLE            0xCCCC
#define IWDG_KR_KEY_WRITE_ACCESS      0x5555

// Hardware watchdog timeout calculation
uint32_t calculate_hw_watchdog_timeout(uint32_t prescaler, uint32_t reload_value) {
    // IWDG clock = LSI / 4 = 40kHz / 4 = 10kHz
    uint32_t iwdg_clock = 10000;  // 10kHz
    
    // Timeout = (reload_value + 1) * prescaler / iwdg_clock
    return ((reload_value + 1) * prescaler * 1000) / iwdg_clock;
}
```

### 软件看门狗定时器(Software Watchdog Timers)
软件看门狗提供灵活性，可以监控特定的应用程序任务：

```c
// Software watchdog structure
typedef struct {
    uint32_t task_id;                  // Task identifier
    uint32_t last_kick_time;           // Last kick timestamp
    uint32_t timeout_ms;               // Task timeout
    bool     enabled;                  // Watchdog enabled flag
    void     (*recovery_callback)(void); // Recovery function pointer
} sw_watchdog_t;

// Software watchdog array
#define MAX_SW_WATCHDOGS 16
sw_watchdog_t sw_watchdogs[MAX_SW_WATCHDOGS];
uint32_t sw_watchdog_count = 0;

// Register software watchdog
int register_sw_watchdog(uint32_t task_id, uint32_t timeout_ms, 
                        void (*recovery_callback)(void)) {
    if (sw_watchdog_count >= MAX_SW_WATCHDOGS) {
        return -1; // No more slots available
    }
    
    sw_watchdogs[sw_watchdog_count].task_id = task_id;
    sw_watchdogs[sw_watchdog_count].timeout_ms = timeout_ms;
    sw_watchdogs[sw_watchdog_count].recovery_callback = recovery_callback;
    sw_watchdogs[sw_watchdog_count].enabled = true;
    sw_watchdogs[sw_watchdog_count].last_kick_time = get_system_time_ms();
    
    sw_watchdog_count++;
    return 0;
}
```

### 窗口看门狗定时器(Windowed Watchdog Timers)
窗口看门狗防止过早喂狗，并确保正确的时序：

```c
// Windowed watchdog configuration
typedef struct {
    uint32_t window_start_ms;          // Start of kick window
    uint32_t window_end_ms;            // End of kick window
    uint32_t total_timeout_ms;         // Total watchdog timeout
    bool     early_kick_detected;      // Early kick detection flag
    uint32_t early_kick_count;         // Count of early kicks
} windowed_watchdog_t;

// Check if kick is within valid window
bool is_kick_within_window(windowed_watchdog_t *wdog, uint32_t current_time) {
    uint32_t time_since_start = current_time % wdog->total_timeout_ms;
    
    if (time_since_start < wdog->window_start_ms) {
        // Too early - kick before window opens
        wdog->early_kick_detected = true;
        wdog->early_kick_count++;
        return false;
    } else if (time_since_start > wdog->window_end_ms) {
        // Too late - window has closed
        return false;
    }
    
    return true;
}
```

---

## 看门狗实现(Watchdog Implementation)

### 硬件看门狗初始化(Hardware Watchdog Initialization)
```c
// Initialize hardware watchdog
int init_hardware_watchdog(hw_watchdog_config_t *config) {
    // 1. Enable write access to IWDG registers
    IWDG->KR = IWDG_KR_KEY_WRITE_ACCESS;
    
    // 2. Configure prescaler
    uint32_t prescaler_value;
    switch (config->prescaler) {
        case 4:   prescaler_value = IWDG_PR_PR_0; break;      // /4
        case 8:   prescaler_value = IWDG_PR_PR_1; break;      // /8
        case 16:  prescaler_value = IWDG_PR_PR_0 | IWDG_PR_PR_1; break; // /16
        case 32:  prescaler_value = IWDG_PR_PR_2; break;      // /32
        case 64:  prescaler_value = IWDG_PR_PR_0 | IWDG_PR_PR_2; break; // /64
        case 128: prescaler_value = IWDG_PR_PR_1 | IWDG_PR_PR_2; break; // /128
        case 256: prescaler_value = IWDG_PR_PR_0 | IWDG_PR_PR_1 | IWDG_PR_PR_2; break; // /256
        default:  return -1; // Invalid prescaler
    }
    IWDG->PR = prescaler_value;
    
    // 3. Calculate reload value for desired timeout
    uint32_t reload_value = (config->timeout_ms * 10000) / (config->prescaler * 1000) - 1;
    IWDG->RLR = reload_value;
    
    // 4. Enable watchdog
    IWDG->KR = IWDG_KR_KEY_ENABLE;
    
    return 0;
}

// Kick hardware watchdog
void kick_hardware_watchdog(void) {
    IWDG->KR = IWDG_KR_KEY_RELOAD;
}
```

### 软件看门狗管理(Software Watchdog Management)
```c
// Software watchdog task
void software_watchdog_task(void *pvParameters) {
    uint32_t current_time;
    uint32_t elapsed_time;
    
    while (1) {
        current_time = get_system_time_ms();
        
        // Check all registered watchdogs
        for (uint32_t i = 0; i < sw_watchdog_count; i++) {
            if (sw_watchdogs[i].enabled) {
                elapsed_time = current_time - sw_watchdogs[i].last_kick_time;
                
                if (elapsed_time > sw_watchdogs[i].timeout_ms) {
                    // Watchdog timeout - trigger recovery
                    log_error("Software watchdog timeout for task %d", 
                             sw_watchdogs[i].task_id);
                    
                    if (sw_watchdogs[i].recovery_callback != NULL) {
                        sw_watchdogs[i].recovery_callback();
                    }
                    
                    // Reset kick time
                    sw_watchdogs[i].last_kick_time = current_time;
                }
            }
        }
        
        // Check every 100ms
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

// Kick software watchdog
void kick_software_watchdog(uint32_t task_id) {
    for (uint32_t i = 0; i < sw_watchdog_count; i++) {
        if (sw_watchdogs[i].task_id == task_id && sw_watchdogs[i].enabled) {
            sw_watchdogs[i].last_kick_time = get_system_time_ms();
            break;
        }
    }
}
```

### 多层看门狗系统(Multi-Level Watchdog System)
```c
// Multi-level watchdog system
typedef struct {
    hw_watchdog_t hw_watchdog;         // Hardware watchdog (last resort)
    sw_watchdog_t app_watchdog;        // Application-level watchdog
    sw_watchdog_t task_watchdog;       // Task-level watchdog
    sw_watchdog_t comm_watchdog;       // Communication watchdog
    uint32_t recovery_level;           // Current recovery level
    uint32_t failure_count;            // Count of consecutive failures
} multi_level_watchdog_t;

// Initialize multi-level watchdog system
int init_multi_level_watchdog(multi_level_watchdog_t *mlwd) {
    // 1. Initialize hardware watchdog (longest timeout)
    hw_watchdog_config_t hw_config = {
        .timeout_ms = 5000,            // 5 second timeout
        .window_mode_enabled = false,
        .debug_mode_enabled = true
    };
    
    if (init_hardware_watchdog(&hw_config) != 0) {
        return -1;
    }
    
    // 2. Initialize application watchdog (medium timeout)
    if (register_sw_watchdog(APP_TASK_ID, 2000, app_recovery_callback) != 0) {
        return -1;
    }
    
    // 3. Initialize task watchdog (short timeout)
    if (register_sw_watchdog(TASK_MONITOR_ID, 500, task_recovery_callback) != 0) {
        return -1;
    }
    
    // 4. Initialize communication watchdog
    if (register_sw_watchdog(COMM_TASK_ID, 1000, comm_recovery_callback) != 0) {
        return -1;
    }
    
    mlwd->recovery_level = 0;
    mlwd->failure_count = 0;
    
    return 0;
}
```

---

## 系统恢复策略(System Recovery Strategies)

### 恢复级别管理(Recovery Level Management)
```c
// Recovery level definitions
typedef enum {
    RECOVERY_LEVEL_NONE = 0,
    RECOVERY_LEVEL_SOFT_RESET,
    RECOVERY_LEVEL_WARM_RESET,
    RECOVERY_LEVEL_COLD_RESET,
    RECOVERY_LEVEL_SAFE_MODE,
    RECOVERY_LEVEL_FACTORY_RESET
} recovery_level_t;

// Recovery level configuration
typedef struct {
    recovery_level_t level;            // Recovery level
    uint32_t max_attempts;             // Maximum attempts at this level
    uint32_t attempt_count;            // Current attempt count
    uint32_t cooldown_period_ms;       // Cooldown period between attempts
    uint32_t last_attempt_time;        // Timestamp of last attempt
    bool     enabled;                  // Level enabled flag
} recovery_level_config_t;

// Recovery level manager
typedef struct {
    recovery_level_config_t levels[RECOVERY_LEVEL_FACTORY_RESET + 1];
    recovery_level_t current_level;    // Current active level
    uint32_t total_failure_count;      // Total failures across all levels
    bool     recovery_in_progress;     // Recovery operation status
} recovery_manager_t;

// Initialize recovery manager
int init_recovery_manager(recovery_manager_t *manager) {
    // Configure each recovery level
    manager->levels[RECOVERY_LEVEL_SOFT_RESET] = (recovery_level_config_t){
        .level = RECOVERY_LEVEL_SOFT_RESET,
        .max_attempts = 3,
        .cooldown_period_ms = 1000,
        .enabled = true
    };
    
    manager->levels[RECOVERY_LEVEL_WARM_RESET] = (recovery_level_config_t){
        .level = RECOVERY_LEVEL_WARM_RESET,
        .max_attempts = 2,
        .cooldown_period_ms = 5000,
        .enabled = true
    };
    
    manager->levels[RECOVERY_LEVEL_COLD_RESET] = (recovery_level_config_t){
        .level = RECOVERY_LEVEL_COLD_RESET,
        .max_attempts = 1,
        .cooldown_period_ms = 30000,
        .enabled = true
    };
    
    manager->levels[RECOVERY_LEVEL_SAFE_MODE] = (recovery_level_config_t){
        .level = RECOVERY_LEVEL_SAFE_MODE,
        .max_attempts = 1,
        .cooldown_period_ms = 0,
        .enabled = true
    };
    
    manager->current_level = RECOVERY_LEVEL_NONE;
    manager->total_failure_count = 0;
    manager->recovery_in_progress = false;
    
    return 0;
}
```

### 恢复动作实现(Recovery Action Implementation)
```c
// Recovery action functions
int perform_soft_reset(void) {
    log_info("Performing soft reset");
    
    // 1. Stop all application tasks
    vTaskSuspendAll();
    
    // 2. Reinitialize critical peripherals
    if (reinit_critical_peripherals() != 0) {
        log_error("Critical peripheral reinitialization failed");
        return -1;
    }
    
    // 3. Restart application tasks
    xTaskResumeAll();
    
    // 4. Reset watchdog timers
    reset_all_watchdogs();
    
    log_info("Soft reset completed");
    return 0;
}

int perform_warm_reset(void) {
    log_info("Performing warm reset");
    
    // 1. Save critical data to non-volatile memory
    if (save_critical_data() != 0) {
        log_error("Failed to save critical data");
        return -1;
    }
    
    // 2. Stop RTOS
    vTaskEndScheduler();
    
    // 3. Reinitialize hardware
    if (reinit_hardware() != 0) {
        log_error("Hardware reinitialization failed");
        return -1;
    }
    
    // 4. Restart RTOS
    if (restart_rtos() != 0) {
        log_error("RTOS restart failed");
        return -1;
    }
    
    log_info("Warm reset completed");
    return 0;
}

int perform_cold_reset(void) {
    log_info("Performing cold reset");
    
    // 1. Save diagnostic information
    save_diagnostic_info();
    
    // 2. Perform complete system reset
    NVIC_SystemReset();
    
    // This function should never return
    return 0;
}
```

---

## 错误检测与处理(Error Detection and Handling)

### 错误分类系统(Error Classification System)
```c
// Error severity levels
typedef enum {
    ERROR_SEVERITY_LOW = 0,
    ERROR_SEVERITY_MEDIUM,
    ERROR_SEVERITY_HIGH,
    ERROR_SEVERITY_CRITICAL,
    ERROR_SEVERITY_FATAL
} error_severity_t;

// Error categories
typedef enum {
    ERROR_CATEGORY_HARDWARE = 0,
    ERROR_CATEGORY_SOFTWARE,
    ERROR_CATEGORY_COMMUNICATION,
    ERROR_CATEGORY_POWER,
    ERROR_CATEGORY_TIMING,
    ERROR_CATEGORY_MEMORY
} error_category_t;

// Error information structure
typedef struct {
    uint32_t error_id;                 // Unique error identifier
    error_severity_t severity;         // Error severity level
    error_category_t category;         // Error category
    uint32_t timestamp;                // Error occurrence time
    uint32_t task_id;                  // Task that detected the error
    uint32_t error_code;               // Specific error code
    char     description[64];          // Error description
    uint32_t context_data[4];          // Additional context data
} error_info_t;

// Error handler function type
typedef void (*error_handler_t)(error_info_t *error);

// Error handler registration
typedef struct {
    error_category_t category;          // Category to handle
    error_severity_t min_severity;     // Minimum severity to handle
    error_handler_t handler;            // Handler function
} error_handler_registration_t;

#define MAX_ERROR_HANDLERS 16
error_handler_registration_t error_handlers[MAX_ERROR_HANDLERS];
uint32_t error_handler_count = 0;

// Register error handler
int register_error_handler(error_category_t category, error_severity_t min_severity,
                          error_handler_t handler) {
    if (error_handler_count >= MAX_ERROR_HANDLERS) {
        return -1;
    }
    
    error_handlers[error_handler_count].category = category;
    error_handlers[error_handler_count].min_severity = min_severity;
    error_handlers[error_handler_count].handler = handler;
    
    error_handler_count++;
    return 0;
}
```

### 错误检测机制(Error Detection Mechanisms)
```c
// Memory corruption detection
typedef struct {
    uint32_t start_magic;              // Start magic number
    uint32_t end_magic;                // End magic number
    uint32_t checksum;                 // Data checksum
    uint32_t size;                     // Data size
} memory_guard_t;

// Check memory integrity
bool check_memory_integrity(void *data, uint32_t size) {
    memory_guard_t *guard = (memory_guard_t*)data;
    
    // Check magic numbers
    if (guard->start_magic != MEMORY_GUARD_MAGIC_START ||
        guard->end_magic != MEMORY_GUARD_MAGIC_END) {
        return false;
    }
    
    // Check size
    if (guard->size != size) {
        return false;
    }
    
    // Check checksum
    uint32_t calculated_checksum = calculate_checksum(
        (uint8_t*)data + sizeof(memory_guard_t),
        size - sizeof(memory_guard_t)
    );
    
    return (calculated_checksum == guard->checksum);
}

// Stack overflow detection
bool check_stack_overflow(void) {
    extern uint32_t _estack;           // Stack end address
    extern uint32_t __stack_start__;   // Stack start address
    
    uint32_t current_sp = __get_MSP(); // Current stack pointer
    
    // Check if stack pointer is within valid range
    if (current_sp < (uint32_t)&__stack_start__ || 
        current_sp > (uint32_t)&_estack) {
        return true; // Stack overflow detected
    }
    
    // Check stack usage pattern
    uint32_t *stack_ptr = (uint32_t*)current_sp;
    for (int i = 0; i < 16; i++) {
        if (stack_ptr[i] == STACK_PATTERN) {
            return true; // Stack overflow detected
        }
    }
    
    return false;
}
```

---

## 恢复机制(Recovery Mechanisms)

### 自动恢复策略(Automatic Recovery Strategies)
```c
// Automatic recovery manager
typedef struct {
    uint32_t recovery_attempts;         // Number of recovery attempts
    uint32_t max_recovery_attempts;     // Maximum recovery attempts
    uint32_t recovery_cooldown_ms;      // Cooldown period between attempts
    uint32_t last_recovery_time;        // Timestamp of last recovery
    bool     automatic_recovery_enabled; // Automatic recovery flag
} auto_recovery_manager_t;

// Perform automatic recovery
int perform_automatic_recovery(auto_recovery_manager_t *manager, 
                              error_info_t *error) {
    uint32_t current_time = get_system_time_ms();
    
    // Check if enough time has passed since last recovery
    if (current_time - manager->last_recovery_time < manager->recovery_cooldown_ms) {
        log_warning("Recovery cooldown period not expired");
        return -1;
    }
    
    // Check if maximum attempts exceeded
    if (manager->recovery_attempts >= manager->max_recovery_attempts) {
        log_error("Maximum recovery attempts exceeded");
        return -1;
    }
    
    // Perform recovery based on error type
    int result = 0;
    switch (error->category) {
        case ERROR_CATEGORY_SOFTWARE:
            result = perform_soft_reset();
            break;
            
        case ERROR_CATEGORY_COMMUNICATION:
            result = restart_communication_tasks();
            break;
            
        case ERROR_CATEGORY_MEMORY:
            result = perform_memory_recovery();
            break;
            
        default:
            result = perform_soft_reset();
            break;
    }
    
    if (result == 0) {
        manager->recovery_attempts++;
        manager->last_recovery_time = current_time;
        log_info("Automatic recovery successful (attempt %d/%d)", 
                manager->recovery_attempts, manager->max_recovery_attempts);
    }
    
    return result;
}
```

### 手动恢复接口(Manual Recovery Interface)
```c
// Manual recovery commands
typedef enum {
    RECOVERY_CMD_SOFT_RESET,
    RECOVERY_CMD_WARM_RESET,
    RECOVERY_CMD_COLD_RESET,
    RECOVERY_CMD_SAFE_MODE,
    RECOVERY_CMD_FACTORY_RESET,
    RECOVERY_CMD_GET_STATUS,
    RECOVERY_CMD_CLEAR_ERRORS
} recovery_command_t;

// Manual recovery handler
int handle_manual_recovery_command(recovery_command_t command) {
    switch (command) {
        case RECOVERY_CMD_SOFT_RESET:
            return perform_soft_reset();
            
        case RECOVERY_CMD_WARM_RESET:
            return perform_warm_reset();
            
        case RECOVERY_CMD_COLD_RESET:
            return perform_cold_reset();
            
        case RECOVERY_CMD_SAFE_MODE:
            return enter_safe_mode();
            
        case RECOVERY_CMD_FACTORY_RESET:
            return perform_factory_reset();
            
        case RECOVERY_CMD_GET_STATUS:
            return get_recovery_status();
            
        case RECOVERY_CMD_CLEAR_ERRORS:
            return clear_error_logs();
            
        default:
            return -1;
    }
}
```

---

## 实现示例(Implementation Examples)

### 完整看门狗与恢复系统(Complete Watchdog and Recovery System)
```c
// Complete watchdog and recovery system
typedef struct {
    multi_level_watchdog_t watchdog;    // Multi-level watchdog system
    recovery_manager_t recovery;        // Recovery level manager
    auto_recovery_manager_t auto_recovery; // Automatic recovery manager
    error_logger_t error_logger;        // Error logging system
    bool system_healthy;                // System health status
} watchdog_recovery_system_t;

// Initialize complete system
int init_watchdog_recovery_system(watchdog_recovery_system_t *system) {
    int result = 0;
    
    // 1. Initialize multi-level watchdog
    result = init_multi_level_watchdog(&system->watchdog);
    if (result != 0) {
        log_error("Failed to initialize multi-level watchdog");
        return result;
    }
    
    // 2. Initialize recovery manager
    result = init_recovery_manager(&system->recovery);
    if (result != 0) {
        log_error("Failed to initialize recovery manager");
        return result;
    }
    
    // 3. Initialize automatic recovery
    system->auto_recovery.recovery_attempts = 0;
    system->auto_recovery.max_recovery_attempts = 5;
    system->auto_recovery.recovery_cooldown_ms = 10000;
    system->auto_recovery.automatic_recovery_enabled = true;
    
    // 4. Initialize error logger
    result = init_error_logger(&system->error_logger);
    if (result != 0) {
        log_error("Failed to initialize error logger");
        return result;
    }
    
    system->system_healthy = true;
    
    log_info("Watchdog and recovery system initialized successfully");
    return 0;
}

// Main watchdog monitoring task
void watchdog_monitoring_task(void *pvParameters) {
    watchdog_recovery_system_t *system = (watchdog_recovery_system_t*)pvParameters;
    
    while (1) {
        // 1. Check system health
        if (!check_system_health()) {
            system->system_healthy = false;
            
            // 2. Log error
            error_info_t error = {
                .error_id = generate_error_id(),
                .severity = ERROR_SEVERITY_HIGH,
                .category = ERROR_CATEGORY_SOFTWARE,
                .timestamp = get_system_time_ms(),
                .task_id = WATCHDOG_TASK_ID,
                .error_code = ERROR_CODE_SYSTEM_UNHEALTHY,
                .description = "System health check failed"
            };
            
            log_error(&system->error_logger, &error);
            
            // 3. Attempt automatic recovery
            if (system->auto_recovery.automatic_recovery_enabled) {
                perform_automatic_recovery(&system->auto_recovery, &error);
            }
        } else {
            system->system_healthy = true;
        }
        
        // 4. Kick hardware watchdog
        kick_hardware_watchdog();
        
        // 5. Check every 100ms
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

---

## 常见陷阱(Common Pitfalls)

### 1. **超时值不足(Insufficient Timeout Values)**
- **问题(Problem)**：看门狗超时过短，导致误触发
- **解决方案(Solution)**：基于任务执行时间计算合适的超时
- **示例(Example)**：使用最坏情况执行时间分析来计算超时

### 2. **缺少看门狗喂狗(Missing Watchdog Kicks)**
- **问题(Problem)**：关键任务忘记喂狗
- **解决方案(Solution)**：在任务调度器中实现自动喂狗
- **示例(Example)**：在空闲任务或定时器中断中喂狗

### 3. **恢复策略不足(Inadequate Recovery Strategies)**
- **问题(Problem)**：恢复动作未解决根本原因
- **解决方案(Solution)**：实现带正确错误分析的渐进式恢复
- **示例(Example)**：从软复位开始，必要时升级到冷复位

### 4. **错误日志记录不佳(Poor Error Logging)**
- **问题(Problem)**：调试恢复问题信息不足
- **解决方案(Solution)**：实现带上下文的全面错误日志记录
- **示例(Example)**：记录错误细节、系统状态和恢复动作

---

## 最佳实践(Best Practices)

### 1. **分层看门狗设计(Layered Watchdog Design)**
- 为不同故障模式使用多个看门狗级别
- 硬件看门狗作为最后手段
- 软件看门狗用于应用程序监控

### 2. **渐进式恢复(Progressive Recovery)**
- 从干扰最小的恢复动作开始
- 基于故障持续性升级恢复级别
- 在恢复尝试之间实现冷却期

### 3. **全面的错误处理(Comprehensive Error Handling)**
- 按严重性和类别对错误分类
- 为每种错误类型实现适当的恢复动作
- 记录所有错误和恢复动作以供分析

### 4. **测试与验证(Testing and Validation)**
- 使用各种故障场景测试恢复机制
- 验证看门狗超时计算
- 在不同系统条件下测试恢复

### 5. **文档与培训(Documentation and Training)**
- 记录所有恢复流程
- 提供清晰的故障排查指南
- 培训操作人员掌握恢复流程

---

## 面试问题(Interview Questions)

### 基础级别(Basic Level)
1. **看门狗定时器的用途是什么?(What is the purpose of a watchdog timer?)**
   - 监控系统健康、检测故障、触发恢复动作

2. **看门狗定时器的主要类型有哪些?(What are the main types of watchdog timers?)**
   - 硬件看门狗、软件看门狗、窗口看门狗

3. **软复位和冷复位有什么区别?(What is the difference between soft reset and cold reset?)**
   - 软复位重启应用程序，冷复位重启整个系统

### 中级级别(Intermediate Level)
1. **如何实现多层看门狗系统?(How would you implement a multi-level watchdog system?)**
   - 硬件看门狗作为最后手段，软件看门狗用于特定监控

2. **实现自动恢复有哪些挑战?(What are the challenges in implementing automatic recovery?)**
   - 错误分类、恢复策略选择、故障升级

3. **如何防止看门狗误触发?(How do you prevent watchdog false triggers?)**
   - 正确的超时计算、窗口操作、任务监控

### 高级级别(Advanced Level)
1. **如何为分布式嵌入式网络设计恢复系统?(How would you design a recovery system for a distributed embedded network?)**
   - 协调恢复、网络拓扑感知、故障传播

2. **不同恢复策略的性能影响有哪些?(What are the performance implications of different recovery strategies?)**
   - 恢复时间、系统可用性、资源使用、数据保护

3. **如何在安全关键系统中实现容错恢复?(How do you implement fault-tolerant recovery in safety-critical systems?)**
   - 冗余系统、投票机制、安全失败操作

### 实用编码问题(Practical Coding Questions)
1. **实现一个基本的硬件看门狗驱动(Implement a basic hardware watchdog driver)**
2. **设计一个软件看门狗任务监控系统(Design a software watchdog task monitoring system)**
3. **创建一个带升级的恢复级别管理器(Create a recovery level manager with escalation)**
4. **实现错误分类和处理系统(Implement error classification and handling system)**
5. **设计一个带恢复协调的多层看门狗(Design a multi-level watchdog with recovery coordination)**

---

## 引导实验(Guided Labs)

### 实验 1：基本看门狗实现(Lab 1: Basic Watchdog Implementation)
1. **搭建(Setup)**：在你的目标 MCU 上配置硬件看门狗
2. **实现(Implement)**：在主循环中实现基本的看门狗喂狗机制
3. **测试(Test)**：验证未喂狗时看门狗触发恢复
4. **添加(Add)**：看门狗活动日志记录和监控

### 实验 2：恢复策略实现(Lab 2: Recovery Strategy Implementation)
1. **设计(Design)**：为你的系统设计恢复层次
2. **实现(Implement)**：不同的恢复级别（软、温、冷）
3. **添加(Add)**：恢复执行前的错误日志记录
4. **测试(Test)**：针对不同故障场景的恢复有效性

### 实验 3：窗口看门狗(Lab 3: Windowed Watchdog)
1. **实现(Implement)**：带可配置时序的窗口看门狗
2. **添加(Add)**：时序违规检测和日志记录
3. **创建(Create)**：过早和过晚喂狗的测试场景
4. **验证(Validate)**：窗口执行和违规处理

## 自我检查(Check Yourself)

### 理解检查(Understanding Check)
- [ ] 你能解释为什么看门狗定时器对嵌入式系统至关重要吗?
- [ ] 你理解硬件和软件看门狗之间的区别吗?
- [ ] 你能解释窗口看门狗的好处吗?
- [ ] 你知道如何选择合适的恢复策略吗?

### 应用检查(Application Check)
- [ ] 你能为你的目标实现基本的看门狗系统吗?
- [ ] 你能实现不同的恢复策略吗?
- [ ] 你能在恢复执行前添加错误日志记录吗?
- [ ] 你能实现带时序验证的窗口看门狗吗?

### 分析检查(Analysis Check)
- [ ] 你能分析看门狗超时模式以识别系统问题吗?
- [ ] 你能测量恢复时间并优化过程吗?
- [ ] 你能为关键故障实现安全模式功能吗?
- [ ] 你能设计全面的错误检测和恢复系统吗?

## 交叉链接(Cross-links)

- **[[Error_Handling_Logging]]** - 错误检测和日志记录策略
- **[[Bootloader_Development]]** - 系统恢复和重启机制
- **[[Real_Time_Debugging]]** - 实时系统监控和恢复
- **[[Watchdog_Timers]]** - 硬件看门狗特性
- **[[Error_Handling_Logging]]** - 系统级错误处理

## 结论(Conclusion)

看门狗定时器和系统恢复机制对于构建健壮的嵌入式系统至关重要。一个设计良好的看门狗和恢复系统提供：

- **可靠性(Reliability)**：自动检测并从系统故障中恢复
- **安全性(Safety)**：防止系统死锁和不安全状态
- **可维护性(Maintainability)**：清晰的错误报告和恢复流程
- **用户体验(User Experience)**：最少用户干预的自动恢复

成功实现看门狗和恢复的关键在于：
- **带多个看门狗级别的分层设计(Layered design)**
- **适当升级的渐进式恢复策略(Progressive recovery)**
- **带正确分类的全面错误处理(Comprehensive error handling)**
- **对所有恢复场景的彻底测试(Thorough testing)**
- **恢复流程和故障排查的清晰文档(Clear documentation)**

通过遵循这些原则并实现本指南中讨论的技术，开发人员可以创建健壮、可靠且可维护的嵌入式系统，自动从故障中恢复并提供出色的用户体验。
