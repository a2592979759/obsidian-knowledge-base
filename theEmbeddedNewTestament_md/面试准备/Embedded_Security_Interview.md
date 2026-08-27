---
tags:
  - 面试准备
  - 嵌入式面试
source: "Interview_Preparation/Foundation_Level/Embedded_Security_Interview.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入
>
> 在网站上刷社区排名的题库、带 AI 反馈的编程练习，以及结构化的面试准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

# 🔒 嵌入式安全（Embedded Security）面试准备

## 🚀 **快速导航**
- [安全基础](#security-fundamentals)
- [密码学实现](#cryptographic-implementation)
- [安全启动与信任链](#secure-boot--chain-of-trust)
- [内存保护](#memory-protection)
- [安全测试](#security-testing)

## 📚 **速查表：核心概念**
- **安全原则**：机密性、完整性、可用性（CIA 三元组）
- **密码学（Cryptography）**：对称/非对称加密、哈希、数字签名
- **安全启动（Secure Boot）**：验证启动流程、信任链、TPM 集成
- **内存保护**：MPU、安全隔离区、缓冲区溢出防护
- **安全测试**：渗透测试、模糊测试、旁路分析

## 🔒 **安全基础**

### **安全原则**

**CIA 三元组**
```
1. 机密性（Confidentiality）：仅授权用户可访问数据
2. 完整性（Integrity）：数据保持未篡改且可信
3. 可用性（Availability）：需要时系统和数据可访问
```

**威胁模型**：
```
1. 攻击向量（Attack Vectors）
   - 物理访问硬件
   - 基于网络的攻击
   - 供应链妥协
   - 旁路攻击（side-channel attacks）

2. 攻击类型（Attack Types）
   - 中间人（Man-in-the-middle）
   - 重放攻击（Replay attacks）
   - 缓冲区溢出（Buffer overflows）
   - 时序攻击（Timing attacks）
```

### **安全架构**

**分层安全**：
```
┌─────────────────────────────────────┐
│           应用层（Application Layer）│
├─────────────────────────────────────┤
│           运行时安全（Runtime）      │
├─────────────────────────────────────┤
│           操作系统安全（OS）         │
├─────────────────────────────────────┤
│           硬件安全（Hardware）       │
├─────────────────────────────────────┤
│           物理安全（Physical）       │
└─────────────────────────────────────┘
```

## 🔒 **密码学实现**

### **哈希函数**

**SHA-256 实现**：
```c
#include <stdint.h>
#include <string.h>

// SHA-256 常量
static const uint32_t K[64] = {
    0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5,
    0x3956c25b, 0x59f111f1, 0x923f82a4, 0xab1c5ed5,
    // ... 更多常量
};

// SHA-256 上下文
typedef struct {
    uint32_t state[8];
    uint32_t count[2];
    uint8_t buffer[64];
} sha256_context_t;

// 初始化 SHA-256
void sha256_init(sha256_context_t *ctx) {
    ctx->state[0] = 0x6a09e667;
    ctx->state[1] = 0xbb67ae85;
    ctx->state[2] = 0x3c6ef372;
    ctx->state[3] = 0xa54ff53a;
    ctx->state[4] = 0x510e527f;
    ctx->state[5] = 0x9b05688c;
    ctx->state[6] = 0x1f83d9ab;
    ctx->state[7] = 0x5be0cd19;
    ctx->count[0] = ctx->count[1] = 0;
}

// SHA-256 变换
void sha256_transform(sha256_context_t *ctx, const uint8_t *data) {
    uint32_t a, b, c, d, e, f, g, h;
    uint32_t W[64];
    uint32_t temp1, temp2;
    
    // 准备消息调度
    for (int i = 0; i < 16; i++) {
        W[i] = (data[i*4] << 24) | (data[i*4+1] << 16) | 
                (data[i*4+2] << 8) | data[i*4+3];
    }
    
    for (int i = 16; i < 64; i++) {
        W[i] = W[i-16] + W[i-7] + 
                ((W[i-15] >> 7) | (W[i-15] << 25)) +
                ((W[i-2] >> 17) | (W[i-2] << 15));
    }
    
    // 初始化工作变量
    a = ctx->state[0]; b = ctx->state[1]; c = ctx->state[2]; d = ctx->state[3];
    e = ctx->state[4]; f = ctx->state[5]; g = ctx->state[6]; h = ctx->state[7];
    
    // 主循环
    for (int i = 0; i < 64; i++) {
        temp1 = h + ((e >> 6) | (e << 26)) + ((e >> 11) | (e << 21)) + 
                (e ^ f ^ g) + K[i] + W[i];
        temp2 = ((a >> 2) | (a << 30)) + ((a >> 13) | (a << 19)) + 
                (a ^ b ^ c) + ((b & c) ^ (a & (b ^ c)));
        
        h = g; g = f; f = e; e = d + temp1;
        d = c; c = b; b = a; a = temp1 + temp2;
    }
    
    // 更新状态
    ctx->state[0] += a; ctx->state[1] += b; ctx->state[2] += c; ctx->state[3] += d;
    ctx->state[4] += e; ctx->state[5] += f; ctx->state[6] += g; ctx->state[7] += h;
}

// 计算 SHA-256 哈希
void sha256_calculate(const uint8_t *data, size_t length, uint8_t *hash) {
    sha256_context_t ctx;
    sha256_init(&ctx);
    
    // 按块处理数据
    size_t remaining = length;
    while (remaining >= 64) {
        sha256_transform(&ctx, data);
        data += 64;
        remaining -= 64;
        ctx.count[0] += 64 * 8;
    }
    
    // 收尾
    // ... 填充与最终变换
}
```

### **AES 加密**

**AES-128 实现**：
```c
#include <stdint.h>

// AES S 盒
static const uint8_t SBOX[256] = {
    0x63, 0x7c, 0x77, 0x7b, 0xf2, 0x6b, 0x6f, 0xc5,
    // ... 完整 S 盒
};

// AES 轮常量
static const uint8_t RCON[10] = {
    0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1b, 0x36
};

// AES 密钥扩展
void aes_key_expansion(const uint8_t *key, uint8_t *round_keys) {
    uint8_t temp[4];
    
    // 复制初始密钥
    memcpy(round_keys, key, 16);
    
    for (int i = 4; i < 44; i++) {
        memcpy(temp, round_keys + (i-1) * 4, 4);
        
        if (i % 4 == 0) {
            // 循环移位并替换
            uint8_t t = temp[0];
            temp[0] = temp[1]; temp[1] = temp[2]; 
            temp[2] = temp[3]; temp[3] = t;
            
            for (int j = 0; j < 4; j++) {
                temp[j] = SBOX[temp[j]];
            }
            
            temp[0] ^= RCON[i/4 - 1];
        }
        
        // 与前一轮密钥异或
        for (int j = 0; j < 4; j++) {
            round_keys[i * 4 + j] = round_keys[(i-4) * 4 + j] ^ temp[j];
        }
    }
}

// AES 加密
void aes_encrypt(const uint8_t *plaintext, const uint8_t *round_keys, uint8_t *ciphertext) {
    uint8_t state[16];
    memcpy(state, plaintext, 16);
    
    // 初始轮密钥加
    for (int i = 0; i < 16; i++) {
        state[i] ^= round_keys[i];
    }
    
    // 主轮次
    for (int round = 1; round < 10; round++) {
        // SubBytes, ShiftRows, MixColumns, AddRoundKey
        aes_sub_bytes(state);
        aes_shift_rows(state);
        aes_mix_columns(state);
        aes_add_round_key(state, round_keys + round * 16);
    }
    
    // 最终轮
    aes_sub_bytes(state);
    aes_shift_rows(state);
    aes_add_round_key(state, round_keys + 160);
    
    memcpy(ciphertext, state, 16);
}
```

## 🔒 **安全启动与信任链**

### **安全启动实现**

**启动验证**：
```c
#include <stdint.h>
#include <stdbool.h>

// 启动镜像头
typedef struct {
    uint32_t magic;
    uint32_t version;
    uint32_t size;
    uint8_t hash[32];
    uint8_t signature[256];
} secure_boot_header_t;

// 验证启动镜像
bool verify_boot_image(const secure_boot_header_t *header, const uint8_t *image_data) {
    // 检查魔数
    if (header->magic != SECURE_BOOT_MAGIC) {
        return false;
    }
    
    // 计算镜像数据的哈希
    uint8_t calculated_hash[32];
    sha256_calculate(image_data, header->size, calculated_hash);
    
    // 验证哈希
    if (memcmp(calculated_hash, header->hash, 32) != 0) {
        return false;
    }
    
    // 验证签名
    return verify_signature(calculated_hash, header->signature, PUBLIC_KEY);
}

// 安全启动序列
bool secure_boot(void) {
    // 从安全存储加载引导加载程序
    secure_boot_header_t bootloader_header;
    uint8_t *bootloader_data;
    
    if (!load_bootloader(&bootloader_header, &bootloader_data)) {
        return false;
    }
    
    // 验证引导加载程序
    if (!verify_boot_image(&bootloader_header, bootloader_data)) {
        return false;
    }
    
    // 加载并验证应用程序
    secure_boot_header_t app_header;
    uint8_t *app_data;
    
    if (!load_application(&app_header, &app_data)) {
        return false;
    }
    
    if (!verify_boot_image(&app_header, app_data)) {
        return false;
    }
    
    // 跳转到已验证的应用程序
    jump_to_application(app_data);
    return true;
}
```

### **TPM 集成**

**TPM 2.0 操作**：
```c
#include <stdint.h>
#include <stdbool.h>

// TPM 上下文
typedef struct {
    uint32_t tpm_handle;
    bool initialized;
} tpm_context_t;

// 初始化 TPM
bool tpm_init(tpm_context_t *tpm) {
    // 初始化 TPM 硬件接口
    if (!init_tpm_hardware()) {
        return false;
    }
    
    // 启动 TPM
    if (!tpm_startup()) {
        return false;
    }
    
    tpm->initialized = true;
    return true;
}

// 扩展 PCR（平台配置寄存器）
bool tpm_extend_pcr(tpm_context_t *tpm, uint32_t pcr_index, const uint8_t *data, size_t data_size) {
    uint8_t digest[32];
    
    // 计算哈希
    sha256_calculate(data, data_size, digest);
    
    // TPM2_PCR_Extend 命令
    uint8_t command[256];
    uint16_t command_size = build_pcr_extend_command(pcr_index, digest, command);
    
    // 向 TPM 发送命令
    uint8_t response[256];
    uint16_t response_size;
    
    if (!tpm_send_command(command, command_size, response, &response_size)) {
        return false;
    }
    
    return parse_pcr_extend_response(response, response_size);
}

// 引用操作（认证）
bool tpm_quote(tpm_context_t *tpm, const uint8_t *nonce, uint8_t *quote, uint8_t *signature) {
    // 构建引用命令
    uint8_t command[512];
    uint16_t command_size = build_quote_command(nonce, command);
    
    // 向 TPM 发送命令
    uint8_t response[512];
    uint16_t response_size;
    
    if (!tpm_send_command(command, command_size, response, &response_size)) {
        return false;
    }
    
    // 解析响应
    return parse_quote_response(response, response_size, quote, signature);
}
```

## 🔒 **内存保护**

### **MPU 配置**

**内存保护单元（MPU）设置**：
```c
#include <stdint.h>
#include <stdbool.h>

// MPU 区域配置
typedef struct {
    uint32_t base_address;
    uint32_t size;
    uint8_t access_permissions;
    uint8_t attributes;
    bool enabled;
} mpu_region_t;

// 配置 MPU 区域
bool configure_mpu_region(uint8_t region_number, const mpu_region_t *config) {
    if (region_number >= MAX_MPU_REGIONS) {
        return false;
    }
    
    // 先禁用区域
    disable_mpu_region(region_number);
    
    // 配置区域基地址和大小
    set_mpu_region_base(region_number, config->base_address);
    set_mpu_region_size(region_number, config->size);
    
    // 配置访问权限
    set_mpu_region_access(region_number, config->access_permissions);
    
    // 配置内存属性
    set_mpu_region_attributes(region_number, config->attributes);
    
    // 使能区域
    if (config->enabled) {
        enable_mpu_region(region_number);
    }
    
    return true;
}

// 安全内存布局
void configure_secure_memory_layout(void) {
    // Flash 内存（可执行、只读）
    mpu_region_t flash_region = {
        .base_address = FLASH_BASE,
        .size = FLASH_SIZE,
        .access_permissions = MPU_READ_ONLY | MPU_EXECUTABLE,
        .attributes = MPU_NORMAL_MEMORY,
        .enabled = true
    };
    configure_mpu_region(0, &flash_region);
    
    // RAM（读-写、不可执行）
    mpu_region_t ram_region = {
        .base_address = RAM_BASE,
        .size = RAM_SIZE,
        .access_permissions = MPU_READ_WRITE,
        .attributes = MPU_NORMAL_MEMORY,
        .enabled = true
    };
    configure_mpu_region(1, &ram_region);
    
    // 外设（读-写、不可执行）
    mpu_region_t peripheral_region = {
        .base_address = PERIPHERAL_BASE,
        .size = PERIPHERAL_SIZE,
        .access_permissions = MPU_READ_WRITE,
        .attributes = MPU_DEVICE_MEMORY,
        .enabled = true
    };
    configure_mpu_region(2, &peripheral_region);
}
```

### **缓冲区溢出防护**

**栈金丝雀（Stack Canary）实现**：
```c
#include <stdint.h>
#include <stdbool.h>

// 栈金丝雀
static uint32_t stack_canary = 0xDEADBEEF;

// 初始化栈金丝雀
void init_stack_canary(void) {
    // 生成随机金丝雀
    stack_canary = generate_random_uint32();
}

// 检查栈金丝雀
bool check_stack_canary(void) {
    // 这会在汇编中实现
    // 检查栈上的金丝雀值
    return true;
}

// 带栈保护的函数
void secure_function(void) {
    uint32_t local_canary = stack_canary;
    
    // 函数体
    // ...
    
    // 返回前检查金丝雀
    if (local_canary != stack_canary) {
        // 检测到栈破坏
        handle_stack_corruption();
    }
}
```

## 🔒 **安全测试**

### **渗透测试**

**安全测试框架**：
```c
#include <stdint.h>
#include <stdbool.h>

// 安全测试结果
typedef struct {
    bool passed;
    char description[128];
    uint32_t test_duration;
    uint32_t vulnerabilities_found;
} security_test_result_t;

// 缓冲区溢出测试
security_test_result_t test_buffer_overflow(void) {
    security_test_result_t result = {0};
    strcpy(result.description, "Buffer Overflow Test");
    
    uint32_t start_time = get_system_time();
    
    // 测试各种缓冲区大小
    for (int size = 1; size <= 1024; size *= 2) {
        uint8_t *buffer = malloc(size);
        if (!buffer) continue;
        
        // 尝试溢出缓冲区
        uint8_t *overflow_data = malloc(size + 100);
        if (overflow_data) {
            memset(overflow_data, 0xAA, size + 100);
            
            // 这应导致溢出
            memcpy(buffer, overflow_data, size + 100);
            
            free(overflow_data);
        }
        
        free(buffer);
    }
    
    result.test_duration = get_system_time() - start_time;
    result.passed = true;  // 会检查实际的溢出检测
    
    return result;
}

// 时序攻击测试
security_test_result_t test_timing_attack(void) {
    security_test_result_t result = {0};
    strcpy(result.description, "Timing Attack Test");
    
    uint32_t start_time = get_system_time();
    
    // 测试恒定时间比较
    uint8_t secret[] = "secret123";
    uint8_t guess[] = "secret123";
    
    uint32_t time1 = get_system_time();
    bool match1 = constant_time_compare(secret, guess, strlen(secret));
    uint32_t time2 = get_system_time();
    
    uint32_t time_diff = time2 - time1;
    
    // 检查时序是否一致
    if (time_diff < MAX_TIMING_VARIATION) {
        result.passed = true;
    } else {
        result.vulnerabilities_found++;
    }
    
    result.test_duration = get_system_time() - start_time;
    return result;
}
```

## 🧪 **常见面试问题**

### **问题 1：实现安全通信**

**问题**：设计两个嵌入式设备之间的安全通信协议。

**求解思路**：
```
1. 协议设计：
   - 使用 Diffie-Hellman 交换密钥
   - 用 AES 加密数据
   - 用 HMAC 做消息认证
   - 用 nonce 防重放

2. 实现：
   - 安全随机数生成
   - 恒定时间密码学操作
   - 安全密钥存储
   - 错误处理
```

**实现**：
```c
// 安全通信上下文
typedef struct {
    uint8_t shared_key[32];
    uint8_t session_nonce[16];
    uint32_t message_counter;
    bool key_established;
} secure_comm_context_t;

// 建立安全连接
bool establish_secure_connection(secure_comm_context_t *ctx) {
    // 生成私钥
    uint8_t private_key[32];
    generate_random_bytes(private_key, 32);
    
    // 执行密钥交换
    uint8_t public_key[32];
    if (!diffie_hellman_key_exchange(private_key, public_key, ctx->shared_key)) {
        return false;
    }
    
    // 生成会话 nonce
    generate_random_bytes(ctx->session_nonce, 16);
    ctx->message_counter = 0;
    ctx->key_established = true;
    
    return true;
}

// 发送安全消息
bool send_secure_message(secure_comm_context_t *ctx, const uint8_t *data, uint16_t length) {
    if (!ctx->key_established) return false;
    
    // 准备消息
    uint8_t message[512];
    uint16_t message_size = 0;
    
    // 添加 nonce 和计数器
    memcpy(message, ctx->session_nonce, 16);
    memcpy(message + 16, &ctx->message_counter, 4);
    message_size = 20;
    
    // 加密数据
    uint8_t encrypted_data[length + 16];  // +16 用于填充
    uint16_t encrypted_size;
    
    if (!aes_encrypt_cbc(data, length, ctx->shared_key, ctx->session_nonce, 
                         encrypted_data, &encrypted_size)) {
        return false;
    }
    
    // 添加加密数据
    memcpy(message + message_size, encrypted_data, encrypted_size);
    message_size += encrypted_size;
    
    // 计算 HMAC
    uint8_t hmac[32];
    calculate_hmac(message, message_size, ctx->shared_key, hmac);
    
    // 添加 HMAC
    memcpy(message + message_size, hmac, 32);
    message_size += 32;
    
    // 发送消息
    bool success = send_message(message, message_size);
    
    if (success) {
        ctx->message_counter++;
    }
    
    return success;
}
```

### **问题 2：安全固件更新**

**问题**：实现一个安全固件更新机制。

**求解思路**：
```
1. 安全需求：
   - 固件真实性验证
   - 完整性检查
   - 回滚保护
   - 安全存储

2. 实现：
   - 数字签名验证
   - 安全启动集成
   - 版本管理
   - 错误恢复
```

**实现**：
```c
// 固件更新上下文
typedef struct {
    uint32_t current_version;
    uint32_t new_version;
    uint8_t firmware_hash[32];
    uint8_t signature[256];
    bool update_pending;
} firmware_update_context_t;

// 验证固件更新
bool verify_firmware_update(const uint8_t *firmware_data, size_t firmware_size,
                           const firmware_update_context_t *update_info) {
    // 检查版本
    if (update_info->new_version <= update_info->current_version) {
        return false;
    }
    
    // 验证哈希
    uint8_t calculated_hash[32];
    sha256_calculate(firmware_data, firmware_size, calculated_hash);
    
    if (memcmp(calculated_hash, update_info->firmware_hash, 32) != 0) {
        return false;
    }
    
    // 验证签名
    return verify_signature(calculated_hash, update_info->signature, UPDATE_PUBLIC_KEY);
}

// 应用固件更新
bool apply_firmware_update(const uint8_t *firmware_data, size_t firmware_size) {
    // 备份当前固件
    if (!backup_current_firmware()) {
        return false;
    }
    
    // 将新固件写入备份位置
    if (!write_firmware_to_backup(firmware_data, firmware_size)) {
        return false;
    }
    
    // 验证备份
    if (!verify_backup_firmware()) {
        restore_original_firmware();
        return false;
    }
    
    // 切换到新固件
    if (!switch_to_new_firmware()) {
        restore_original_firmware();
        return false;
    }
    
    return true;
}
```

## 🧪 **练习题**

### **问题 1：安全密钥存储**

**场景**：设计一个在嵌入式设备中安全存储加密密钥的系统。

**问题**：实现带硬件保护的安全密钥存储。

**预期分析**：
```
1. 安全需求：
   - 密钥机密性
   - 访问控制
   - 篡改检测
   - 安全删除

2. 实现：
   - 硬件安全模块
   - 密钥派生函数
   - 访问策略
   - 审计日志
```

### **问题 2：旁路攻击防护**

**场景**：实现抗时序和功率分析攻击的密码学函数。

**问题**：设计恒定时间的密码学操作。

**预期分析**：
```
1. 攻击向量：
   - 时序分析
   - 功率分析
   - 缓存攻击
   - 电磁分析

2. 对策：
   - 恒定时间算法
   - 随机延迟
   - 功率掩蔽
   - 缓存隔离
```

## ✅ **自我评估清单**

### **安全基础** ✅
- [ ] 能解释 CIA 三元组和安全原则
- [ ] 能识别常见攻击向量
- [ ] 能设计安全架构
- [ ] 能实现威胁建模

### **密码学** ✅
- [ ] 能实现哈希函数
- [ ] 能实现加密算法
- [ ] 能处理密钥管理
- [ ] 能防止旁路攻击

### **安全启动** ✅
- [ ] 能实现安全启动流程
- [ ] 能验证固件完整性
- [ ] 能集成 TPM 功能
- [ ] 能建立信任链

### **内存保护** ✅
- [ ] 能配置 MPU 区域
- [ ] 能防止缓冲区溢出
- [ ] 能实现安全内存布局
- [ ] 能处理内存访问违规

### **安全测试** ✅
- [ ] 能执行渗透测试
- [ ] 能识别漏洞
- [ ] 能实现安全测试框架
- [ ] 能验证安全措施

## 🔗 **相关主题**
- [[C_Programming_Interview]]
- [[Embedded_Security_Interview]]
- [[System_Integration_Interview]]
- [[Performance_Optimization_Interview]]

## 📚 **附加资源**
- **安全标准**：[NIST 网络安全框架](https://www.nist.gov/cyberframework)
- **密码学**：[密码学标准](https://www.nist.gov/cryptography)
- **安全启动**：[UEFI 安全启动](https://uefi.org/secureboot)
- **安全测试**：[OWASP 测试指南](https://owasp.org/www-project-web-security-testing-guide/)

## 相关页面

- [[C_Programming_Interview]]
- [[RTOS_Interview]]
- [[Bus_Protocols_Interview]]
- [[Problem_Solving_Approach]]
- [[00-索引]]

返回索引 [[00-索引]]
