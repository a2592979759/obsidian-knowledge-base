---
tags:
  - 嵌入式安全
source: Embedded_Security/Secure_Boot_Chain_Trust.md
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 练习与深度学习
>
> 把这些安全概念作为排名面试题（附模型答案）来练习，并查看交互式深度学习指南。
>
> 👉 **[浏览安全与可靠性问题 →](https://embeddedinterviewlab.com/questions/domain/safety-security-reliability?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=embedded_security)** &nbsp;·&nbsp; **[阅读深入指南 →](https://embeddedinterviewlab.com/topics/secure-boot-and-crypto?utm_source=github&utm_medium=referral&utm_campaign=kb_topic&utm_content=embedded_security)**

---

# 安全启动与信任链（Secure Boot and Chain of Trust）

> **从上电到运行时构建牢不可破的信任**  
> 理解安全启动如何创建一个保护整个系统的信任基础

---

## 📋 **目录**

- [信任基础](#信任基础)
- [信任链](#信任链)
- [安全启动过程](#安全启动过程)
- [实现细节](#实现细节)
- [高级特性](#高级特性)

---

## 🏗️ **信任基础**

### **为什么信任从第一条指令开始就至关重要**

想象你在建造一栋房子。你不会从墙壁或屋顶开始——你会从一个坚实的地基开始。安全启动就是你嵌入式系统安全性的那个地基。

**信任问题**

当你的设备上电时，它处于完全不受信任的状态。任何人都可能：
- **用恶意代码替换固件**
- **修改引导加载程序以跳过安全检查**
- **篡改硬件以绕过保护**
- **拦截启动过程以注入恶意代码**

**解决方案：安全启动**

安全启动确保只有受信任、经过验证的代码能在你的系统上运行。这就像有一个保安在每个人进入大楼之前检查他们的身份证。

---

## 🔗 **信任链**

### **一步步构建信任**

信任链就像一场接力赛，每个跑者必须在传递接力棒之前验证下一位跑者。如果链条中的任何一环断裂，信任就丢失了。

#### **信任链可视化**

```
上电 → ROM → 引导加载程序 → 内核 → 应用
   ↓         ↓        ↓         ↓         ↓
不受信任 → 受信任 → 受信任 → 受信任 → 受信任
  状态     代码     代码     代码     代码
```

**它是如何工作的：**

1. **ROM（只读存储器，Read-Only Memory）** - 包含初始的可信代码
2. **引导加载程序（Bootloader）** - 在执行前由 ROM 验证
3. **内核（Kernel）** - 在执行前由引导加载程序验证
4. **应用（Applications）** - 在执行前由内核验证

#### **信任验证过程**

```
┌─────────────────────────────────────┐
│           ROM（受信任）               │
│  ┌─────────────────────────────┐    │
│  │   公钥哈希（Public Key Hash） │    │
│  │   （硬编码在 ROM 中）         │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
              ↓
        验证引导加载程序
              ↓
┌─────────────────────────────────────┐
│      引导加载程序（已验证）           │
│  ┌─────────────────────────────┐    │
│  │   数字签名（Digital Signature）│    │
│  │   （由可信密钥签名）           │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
              ↓
           验证内核
              ↓
┌─────────────────────────────────────┐
│          内核（已验证）              │
│  ┌─────────────────────────────┐    │
│  │   数字签名（Digital Signature）│    │
│  │   （由可信密钥签名）           │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

## 🚀 **安全启动过程**

### **分步启动验证**

#### **阶段 1：硬件初始化**

当电源接通时，系统从完全不受信任的状态开始：

```c
// 硬件复位序列
void hardware_reset() {
    // 清除所有寄存器与内存
    clear_all_registers();
    clear_all_memory();
    
    // 禁用除必要之外的所有外设
    disable_non_essential_peripherals();
    
    // 将安全模式设为最高级别
    set_security_mode(SECURITY_MODE_HIGHEST);
    
    // 跳转到 ROM 代码
    jump_to_rom();
}
```

#### **阶段 2：ROM 代码执行**

ROM 包含无法修改的第一段可信代码：

```c
// ROM 代码（不可变，可信）
void rom_entry_point() {
    // 初始化最小硬件
    init_minimal_hardware();
    
    // 从存储加载引导加载程序
    uint8_t* bootloader = load_bootloader_from_storage();
    
    // 验证引导加载程序签名
    if (!verify_bootloader_signature(bootloader)) {
        // 引导加载程序验证失败
        enter_secure_failure_mode();
        return;
    }
    
    // 计算引导加载程序哈希
    uint8_t calculated_hash[32];
    calculate_hash(bootloader, BOOTLOADER_SIZE, calculated_hash);
    
    // 与可信哈希比较
    if (!compare_hashes(calculated_hash, TRUSTED_BOOTLOADER_HASH)) {
        // 哈希不匹配 —— 引导加载程序已损坏
        enter_secure_failure_mode();
        return;
    }
    
    // 引导加载程序可信 —— 执行它
    execute_bootloader(bootloader);
}
```

#### **阶段 3：引导加载程序验证**

引导加载程序必须验证自身与下一组件：

```c
// 引导加载程序验证
typedef struct {
    uint8_t signature[64];      // 数字签名
    uint8_t hash[32];          // 代码哈希
    uint32_t version;          // 版本号
    uint32_t flags;            // 安全标志
    uint8_t reserved[16];      // 保留供将来使用
} BootloaderHeader;

bool verify_bootloader_integrity(uint8_t* bootloader) {
    BootloaderHeader* header = (BootloaderHeader*)bootloader;
    
    // 步骤 1：使用 ROM 公钥验证签名
    uint8_t* code_start = bootloader + sizeof(BootloaderHeader);
    uint32_t code_size = BOOTLOADER_SIZE - sizeof(BootloaderHeader);
    
    if (!verify_signature(header->signature, 
                         code_start, 
                         code_size, 
                         ROM_PUBLIC_KEY)) {
        return false; // 签名验证失败
    }
    
    // 步骤 2：验证代码哈希
    uint8_t calculated_hash[32];
    calculate_hash(code_start, code_size, calculated_hash);
    
    if (memcmp(calculated_hash, header->hash, 32) != 0) {
        return false; // 哈希不匹配
    }
    
    // 步骤 3：检查版本兼容性
    if (header->version < MIN_SUPPORTED_VERSION) {
        return false; // 版本太旧
    }
    
    // 步骤 4：检查安全标志
    if (header->flags & FLAG_DEBUG_ENABLED) {
        return false; // 不允许调试模式
    }
    
    return true; // 所有检查通过
}
```

#### **阶段 4：内核验证**

引导加载程序在加载内核之前验证它：

```c
// 内核验证过程
bool verify_kernel(uint8_t* kernel_image) {
    KernelHeader* header = (KernelHeader*)kernel_image;
    
    // 验证内核签名
    if (!verify_signature(header->signature, 
                         kernel_image + sizeof(KernelHeader),
                         header->code_size,
                         BOOTLOADER_PUBLIC_KEY)) {
        return false;
    }
    
    // 验证内核哈希
    uint8_t calculated_hash[32];
    calculate_hash(kernel_image + sizeof(KernelHeader),
                  header->code_size,
                  calculated_hash);
    
    if (memcmp(calculated_hash, header->hash, 32) != 0) {
        return false;
    }
    
    // 检查内核安全属性
    if (!verify_kernel_security_attributes(header)) {
        return false;
    }
    
    return true;
}
```

---

## 🛠️ **实现细节**

### **构建你的安全启动系统**

#### **1. 密码学原语**

你需要几个密码学函数：

```c
// 密码学函数原型
typedef struct {
    uint8_t modulus[256];      // RSA 模数
    uint8_t exponent[4];       // RSA 指数
} RSAPublicKey;

// 哈希计算（SHA-256）
void calculate_sha256(const uint8_t* data, 
                     uint32_t length, 
                     uint8_t* hash);

// RSA 签名验证
bool verify_rsa_signature(const uint8_t* signature,
                         const uint8_t* data,
                         uint32_t data_length,
                         const RSAPublicKey* public_key);

// 用于完整性的 HMAC 计算
void calculate_hmac(const uint8_t* key,
                   uint32_t key_length,
                   const uint8_t* data,
                   uint32_t data_length,
                   uint8_t* hmac);
```

#### **2. 密钥管理**

管理密码学密钥至关重要：

```c
// 密钥存储结构
typedef struct {
    uint8_t key_id[16];        // 唯一密钥标识符
    uint8_t public_key[256];   // 公钥数据
    uint32_t key_type;         // 密钥类型（RSA、ECC 等）
    uint32_t key_size;         // 以位为单位的密钥大小
    uint32_t permissions;      // 此密钥可以验证什么
    uint8_t reserved[32];      // 保留供将来使用
} TrustedKey;

// 密钥数据库
typedef struct {
    uint32_t num_keys;         // 可信密钥数量
    TrustedKey keys[MAX_KEYS]; // 可信密钥数组
    uint8_t reserved[64];      // 保留供将来使用
} KeyDatabase;

// 按 ID 查找密钥
TrustedKey* find_trusted_key(const uint8_t* key_id) {
    for (int i = 0; i < key_database.num_keys; i++) {
        if (memcmp(key_database.keys[i].key_id, key_id, 16) == 0) {
            return &key_database.keys[i];
        }
    }
    return NULL; // 未找到密钥
}
```

#### **3. 安全存储**

保护密钥与敏感数据：

```c
// 安全存储实现
typedef struct {
    uint8_t data[256];         // 加密数据
    uint8_t iv[16];            // 初始化向量
    uint8_t tag[16];           // 认证标签
} EncryptedData;

// 加密敏感数据
bool encrypt_secure_data(const uint8_t* plaintext,
                        uint32_t length,
                        const uint8_t* key,
                        EncryptedData* encrypted) {
    
    // 生成随机 IV
    if (!generate_random_bytes(encrypted->iv, 16)) {
        return false;
    }
    
    // 使用 AES-GCM 加密数据
    if (!aes_gcm_encrypt(plaintext, length,
                         key, encrypted->iv,
                         encrypted->data, encrypted->tag)) {
        return false;
    }
    
    return true;
}

// 解密敏感数据
bool decrypt_secure_data(const EncryptedData* encrypted,
                        const uint8_t* key,
                        uint8_t* plaintext,
                        uint32_t* length) {
    
    // 使用 AES-GCM 解密
    if (!aes_gcm_decrypt(encrypted->data, encrypted->tag,
                         key, encrypted->iv,
                         plaintext, length)) {
        return false;
    }
    
    return true;
}
```

---

## 🚀 **高级特性**

### **超越基本安全启动**

#### **1. 回滚保护**

防止攻击者降级到有漏洞的版本：

```c
// 版本管理
typedef struct {
    uint32_t major_version;    // 主版本号
    uint32_t minor_version;    // 次版本号
    uint32_t patch_version;    // 修订版本号
    uint32_t build_number;     // 构建编号
} VersionInfo;

// 检查回滚尝试
bool check_version_rollback(const VersionInfo* new_version) {
    VersionInfo current_version = get_current_version();
    
    // 比较版本号
    if (new_version->major_version < current_version.major_version) {
        return false; // 主版本回滚
    }
    
    if (new_version->major_version == current_version.major_version) {
        if (new_version->minor_version < current_version.minor_version) {
            return false; // 次版本回滚
        }
        
        if (new_version->minor_version == current_version.minor_version) {
            if (new_version->patch_version < current_version.patch_version) {
                return false; // 修订版本回滚
            }
        }
    }
    
    return true; // 未检测到回滚
}
```

#### **2. 安全更新过程**

确保更新保持安全性：

```c
// 更新验证
typedef struct {
    uint8_t signature[64];     // 更新签名
    uint8_t hash[32];          // 更新哈希
    VersionInfo version;       // 更新版本
    uint32_t update_size;      // 更新大小
    uint8_t update_type;       // 更新类型
    uint8_t reserved[15];      // 保留
} UpdateHeader;

bool verify_update_package(const uint8_t* update_data) {
    UpdateHeader* header = (UpdateHeader*)update_data;
    
    // 验证更新签名
    if (!verify_signature(header->signature,
                         update_data + sizeof(UpdateHeader),
                         header->update_size,
                         UPDATE_SIGNING_KEY)) {
        return false;
    }
    
    // 检查回滚
    if (!check_version_rollback(&header->version)) {
        return false;
    }
    
    // 验证更新哈希
    uint8_t calculated_hash[32];
    calculate_hash(update_data + sizeof(UpdateHeader),
                  header->update_size,
                  calculated_hash);
    
    if (memcmp(calculated_hash, header->hash, 32) != 0) {
        return false;
    }
    
    return true;
}
```

#### **3. 安全恢复模式**

在保持安全性的同时提供恢复选项：

```c
// 恢复模式处理
typedef enum {
    RECOVERY_MODE_NONE,
    RECOVERY_MODE_UPDATE,
    RECOVERY_MODE_FACTORY_RESET,
    RECOVERY_MODE_DEBUG
} RecoveryMode;

bool enter_recovery_mode(RecoveryMode mode) {
    // 检查是否允许恢复模式
    if (!is_recovery_mode_allowed(mode)) {
        return false;
    }
    
    // 验证恢复模式请求
    if (!verify_recovery_request(mode)) {
        return false;
    }
    
    // 设置恢复模式
    set_recovery_mode(mode);
    
    // 重启进入恢复模式
    system_reboot();
    
    return true;
}
```

---

## 🎯 **关键要点**

### **基本原则**

1. **信任从上电开始** —— 每个组件都必须被验证
2. **信任链牢不可破** —— 一个环节断裂则一切断裂
3. **密码学验证** —— 为所有验证使用强密码学
4. **安全密钥管理** —— 像保护代码一样保护你的密钥
5. **回滚保护** —— 防止降级攻击

### **实现检查清单**

- [ ] **ROM 代码**包含可信公钥
- [ ] **引导加载程序**经过密码学签名
- [ ] **内核**验证已实现
- [ ] **应用**验证已实现
- [ ] **密钥管理**系统已就位
- [ ] **回滚保护**已启用
- [ ] **安全更新**过程已实现
- [ ] **恢复模式**安全已实现
- [ ] **篡改检测**已启用
- [ ] **安全测试**已完成

### **常见陷阱**

1. **弱密码学** —— 使用行业标准算法
2. **密钥暴露** —— 绝不暴露私钥
3. **无回滚保护** —— 攻击者可以降级到有漏洞的版本
4. **验证不足** —— 验证一切，信任无物
5. **无恢复计划** —— 为安全恢复场景做规划

---

## 📚 **更多资源**

- **NIST《可信平台模块（TPM）摘要》** —— TPM 规范
- **ARM《安全启动实现指南》** —— ARM 安全启动指南
- **UEFI 论坛《UEFI 安全启动》** —— UEFI 安全启动规范

---

**下一主题**：[[TPM2_Basics]] → [[Cryptographic_Foundations]]
