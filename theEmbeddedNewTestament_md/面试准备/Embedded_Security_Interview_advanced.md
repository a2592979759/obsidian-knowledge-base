---
tags:
  - 面试准备
  - 嵌入式面试
source: "Interview_Preparation/Advanced_Level/Embedded_Security_Interview.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入
>
> 在网站上刷社区排名的题库、带 AI 反馈的编程练习，以及结构化的面试准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_preparation)**

---

# 🎯 嵌入式安全面试准备

## 🚀 **快速导航**
- [常见问题](#常见问题)
- [问题求解示例](#问题求解示例)
- [练习题](#练习题)
- [资源](#资源)

## 📚 **速查表：核心概念**
- **安全启动（Secure Boot）**：信任链（chain of trust）、签名验证（signature verification）、反回滚保护（anti-rollback protection）
- **密码学（Cryptography）**：对称/非对称算法（symmetric/asymmetric algorithms）、密钥管理（key management）、安全存储（secure storage）
- **侧信道攻击（Side-Channel Attacks）**：时序攻击（timing attacks）、功耗分析（power analysis）、故障注入（fault injection）
- **平台安全（Platform Security）**：ARM TrustZone、安全隔离区（secure enclaves）、内存保护（memory protection）
- **安全协议（Security Protocols）**：TLS、安全通信、认证机制（authentication mechanisms）

## 🎯 **常见面试问题**

### **问题 1：实现一个集成 TPM 的安全启动系统**

**为什么这很重要**：安全启动对于防止未授权固件、维持系统完整性至关重要。

**问题**：设计一个使用 TPM 2.0 验证固件的安全启动系统。

**需求**：
- 验证引导加载程序（bootloader）签名
- 将固件度量（measure）到 TPM PCR 中
- 防止回滚攻击
- 处理更新场景

**方案设计**：
```c
typedef struct {
    uint32_t magic_number;
    uint32_t version;
    uint32_t size;
    uint8_t hash[SHA256_DIGEST_SIZE];
    uint8_t signature[RSA_SIGNATURE_SIZE];
    uint32_t flags;
} secure_firmware_header_t;

typedef struct {
    TPM2_HANDLE tpm_handle;
    uint8_t pcr_values[24][SHA256_DIGEST_SIZE];
    bool pcr_extended;
} tpm_context_t;

// TPM 2.0 PCR extension
bool extend_pcr(tpm_context_t* tpm, uint32_t pcr_index, const uint8_t* data, size_t data_size) {
    uint8_t digest[SHA256_DIGEST_SIZE];
    
    // Calculate SHA256 hash
    if (!sha256_calculate(data, data_size, digest)) {
        return false;
    }
    
    // TPM2_PCR_Extend command
    TPM2_PCR_EXTEND_CMD cmd = {
        .pcrHandle = pcr_index,
        .digests.count = 1,
        .digests.digests[0].hashAlg = TPM2_ALG_SHA256,
        .digests.digests[0].digest = {0}
    };
    memcpy(cmd.digests.digests[0].digest, digest, SHA256_DIGEST_SIZE);
    
    // Send command to TPM
    TPM2_PCR_EXTEND_RSP rsp;
    if (!tpm2_send_command(TPM2_CC_PCR_Extend, &cmd, sizeof(cmd), &rsp, sizeof(rsp))) {
        return false;
    }
    
    // Update local PCR values
    memcpy(tpm->pcr_values[pcr_index], digest, SHA256_DIGEST_SIZE);
    tpm->pcr_extended = true;
    
    return true;
}

// Secure boot sequence
boot_result_t secure_boot_with_tpm(void) {
    tpm_context_t tpm = {0};
    
    // Initialize TPM
    if (!tpm2_init(&tpm.tpm_handle)) {
        return BOOT_TPM_INIT_FAILED;
    }
    
    // Measure bootloader into PCR 0
    if (!extend_pcr(&tpm, 0, (uint8_t*)BOOTLOADER_START, BOOTLOADER_SIZE)) {
        return BOOT_TPM_MEASUREMENT_FAILED;
    }
    
    // Validate bootloader signature
    if (!verify_bootloader_signature()) {
        return BOOT_INVALID_SIGNATURE;
    }
    
    // Check for firmware update
    if (check_firmware_update()) {
        if (!secure_firmware_update(&tpm)) {
            return BOOT_UPDATE_FAILED;
        }
    }
    
    // Measure application firmware into PCR 1
    secure_firmware_header_t* header = (secure_firmware_header_t*)APP_START_ADDRESS;
    
    if (!extend_pcr(&tpm, 1, (uint8_t*)header, header->size)) {
        return BOOT_TPM_MEASUREMENT_FAILED;
    }
    
    // Verify firmware signature
    if (!verify_firmware_signature(header)) {
        return BOOT_INVALID_SIGNATURE;
    }
    
    // Verify firmware hash
    uint8_t calculated_hash[SHA256_DIGEST_SIZE];
    if (!sha256_calculate((uint8_t*)header, header->size, calculated_hash)) {
        return BOOT_HASH_CALCULATION_FAILED;
    }
    
    if (memcmp(calculated_hash, header->hash, SHA256_DIGEST_SIZE) != 0) {
        return BOOT_HASH_MISMATCH;
    }
    
    // Check rollback protection
    if (!check_rollback_protection(&tpm, header->version)) {
        return BOOT_VERSION_ROLLBACK;
    }
    
    // Final PCR measurement for boot success
    if (!extend_pcr(&tpm, 2, (uint8_t*)"BOOT_SUCCESS", 12)) {
        return BOOT_TPM_MEASUREMENT_FAILED;
    }
    
    return BOOT_SUCCESS;
}

// Rollback protection using TPM
bool check_rollback_protection(tpm_context_t* tpm, uint32_t firmware_version) {
    // Store version in TPM NV index
    uint32_t stored_version = 0;
    
    if (!tpm2_nv_read(tpm->tpm_handle, VERSION_NV_INDEX, (uint8_t*)&stored_version, sizeof(stored_version))) {
        // First boot, store current version
        return tpm2_nv_write(tpm->tpm_handle, VERSION_NV_INDEX, (uint8_t*)&firmware_version, sizeof(firmware_version));
    }
    
    // Check if new version is higher
    if (firmware_version <= stored_version) {
        return false;  // Rollback detected
    }
    
    // Update stored version
    return tpm2_nv_write(tpm->tpm_handle, VERSION_NV_INDEX, (uint8_t*)&firmware_version, sizeof(firmware_version));
}
```

**安全特性**：
- **TPM 集成**：基于硬件的安全度量
- **PCR 扩展**：信任链验证
- **回滚保护**：使用 TPM NV 存储进行版本控制
- **签名验证**：对所有组件进行密码学校验

**追问**：
- 你会如何处理 TPM 故障？
- 如果 TPM 被攻破，怎么办？

### **问题 2：实现侧信道攻击的对策**

**问题**：设计一个能抵抗时序与功耗分析攻击的密码学实现。

**需求**：
- 恒定时间操作（constant-time operations）
- 抗功耗分析（power analysis resistance）
- 故障注入保护
- 安全密钥存储

**方案设计**：
```c
// Constant-time AES implementation
void aes_encrypt_constant_time(const uint8_t* key, const uint8_t* plaintext, uint8_t* ciphertext) {
    // Use lookup tables with constant memory access patterns
    static const uint8_t sbox[256] = AES_SBOX;
    static const uint8_t inv_sbox[256] = AES_INV_SBOX;
    
    // Key expansion (constant time)
    uint32_t round_keys[44];
    aes_key_expansion(key, round_keys);
    
    // Initial round
    uint8_t state[16];
    memcpy(state, plaintext, 16);
    add_round_key(state, round_keys, 0);
    
    // Main rounds (constant time)
    for (int round = 1; round < 10; round++) {
        // SubBytes - constant time using lookup
        for (int i = 0; i < 16; i++) {
            state[i] = sbox[state[i]];
        }
        
        // ShiftRows - constant time
        shift_rows_constant_time(state);
        
        // MixColumns - constant time
        mix_columns_constant_time(state);
        
        // AddRoundKey
        add_round_key(state, round_keys, round);
    }
    
    // Final round
    for (int i = 0; i < 16; i++) {
        state[i] = sbox[state[i]];
    }
    shift_rows_constant_time(state);
    add_round_key(state, round_keys, 10);
    
    memcpy(ciphertext, state, 16);
}

// Constant-time shift rows
void shift_rows_constant_time(uint8_t* state) {
    // Row 0: no shift
    // Row 1: shift left by 1
    uint8_t temp = state[1];
    state[1] = state[5];
    state[5] = state[9];
    state[9] = state[13];
    state[13] = temp;
    
    // Row 2: shift left by 2
    temp = state[2];
    state[2] = state[10];
    state[10] = temp;
    temp = state[6];
    state[6] = state[14];
    state[14] = temp;
    
    // Row 3: shift left by 3
    temp = state[3];
    state[3] = state[15];
    state[15] = state[11];
    state[11] = state[7];
    state[7] = temp;
}

// Power analysis resistant key comparison
bool secure_key_compare(const uint8_t* key1, const uint8_t* key2, size_t length) {
    uint8_t result = 0;
    
    // Constant-time comparison
    for (size_t i = 0; i < length; i++) {
        result |= key1[i] ^ key2[i];
    }
    
    // Return true if all bytes match (result == 0)
    return (result == 0);
}

// Fault injection protection
bool verify_checksum_with_protection(const uint8_t* data, size_t data_size, uint32_t expected_checksum) {
    // Calculate checksum multiple times
    uint32_t checksum1 = calculate_crc32(data, data_size);
    uint32_t checksum2 = calculate_crc32(data, data_size);
    uint32_t checksum3 = calculate_crc32(data, data_size);
    
    // All three must match
    if (checksum1 != checksum2 || checksum2 != checksum3) {
        return false;  // Fault injection detected
    }
    
    // Compare with expected value
    return (checksum1 == expected_checksum);
}
```

**侧信道对策**：
- **恒定时间操作**：消除时序变化
- **抗功耗分析**：使用查找表与恒定模式
- **故障注入保护**：多次计算与验证
- **安全密钥存储**：尽可能使用硬件安全模块

### **问题 3：为嵌入式设备设计一个安全通信协议**

**问题**：创建保护通信免受中间人攻击（man-in-the-middle attacks）、确保数据完整性的安全通信协议。

**需求**：
- 双向认证（mutual authentication）
- 加密通信
- 完整性验证
- 会话管理（session management）

**方案设计**：
```c
typedef struct {
    uint8_t device_id[32];
    uint8_t public_key[64];
    uint32_t session_id;
    uint32_t sequence_number;
} device_identity_t;

typedef struct {
    uint8_t session_key[32];
    uint8_t iv[16];
    uint32_t session_id;
    uint32_t sequence_number;
    uint64_t timestamp;
} session_context_t;

// Secure handshake protocol
bool establish_secure_session(device_identity_t* local_device, 
                            device_identity_t* remote_device,
                            session_context_t* session) {
    // Generate random challenge
    uint8_t challenge[32];
    if (!generate_random_bytes(challenge, sizeof(challenge))) {
        return false;
    }
    
    // Send challenge to remote device
    secure_message_t challenge_msg = {
        .type = MSG_TYPE_CHALLENGE,
        .data = challenge,
        .data_size = sizeof(challenge)
    };
    
    if (!send_secure_message(&challenge_msg)) {
        return false;
    }
    
    // Receive challenge response
    secure_message_t response_msg;
    if (!receive_secure_message(&response_msg)) {
        return false;
    }
    
    // Verify challenge response
    if (!verify_challenge_response(challenge, &response_msg, remote_device)) {
        return false;
    }
    
    // Generate session key
    uint8_t session_key[32];
    if (!generate_session_key(local_device, remote_device, challenge, session_key)) {
        return false;
    }
    
    // Initialize session context
    session->session_id = generate_random_uint32();
    session->sequence_number = 0;
    memcpy(session->session_key, session_key, sizeof(session_key));
    
    // Generate random IV
    if (!generate_random_bytes(session->iv, sizeof(session->iv))) {
        return false;
    }
    
    return true;
}

// Encrypted message transmission
bool send_encrypted_message(session_context_t* session, 
                           const uint8_t* data, 
                           size_t data_size) {
    // Create message header
    message_header_t header = {
        .session_id = session->session_id,
        .sequence_number = session->sequence_number++,
        .timestamp = get_system_time(),
        .data_size = data_size
    };
    
    // Calculate HMAC for integrity
    uint8_t hmac[32];
    if (!calculate_hmac(session->session_key, sizeof(session->session_key),
                       (uint8_t*)&header, sizeof(header),
                       data, data_size, hmac)) {
        return false;
    }
    
    // Encrypt data
    uint8_t encrypted_data[data_size + 16];  // +16 for padding
    size_t encrypted_size;
    
    if (!aes_encrypt_cbc(session->session_key, session->iv,
                         data, data_size,
                         encrypted_data, &encrypted_size)) {
        return false;
    }
    
    // Create final message
    secure_message_t secure_msg = {
        .type = MSG_TYPE_ENCRYPTED,
        .header = header,
        .hmac = hmac,
        .data = encrypted_data,
        .data_size = encrypted_size
    };
    
    // Send message
    return send_secure_message(&secure_msg);
}

// Message verification and decryption
bool receive_encrypted_message(session_context_t* session,
                              uint8_t* data,
                              size_t* data_size) {
    secure_message_t secure_msg;
    
    if (!receive_secure_message(&secure_msg)) {
        return false;
    }
    
    // Verify session ID
    if (secure_msg.header.session_id != session->session_id) {
        return false;
    }
    
    // Verify sequence number
    if (secure_msg.header.sequence_number <= session->sequence_number) {
        return false;  // Replay attack detected
    }
    
    // Verify HMAC
    uint8_t calculated_hmac[32];
    if (!calculate_hmac(session->session_key, sizeof(session->session_key),
                       (uint8_t*)&secure_msg.header, sizeof(secure_msg.header),
                       secure_msg.data, secure_msg.data_size, calculated_hmac)) {
        return false;
    }
    
    if (memcmp(calculated_hmac, secure_msg.hmac, sizeof(hmac)) != 0) {
        return false;  // Integrity check failed
    }
    
    // Decrypt data
    if (!aes_decrypt_cbc(session->session_key, session->iv,
                         secure_msg.data, secure_msg.data_size,
                         data, data_size)) {
        return false;
    }
    
    // Update session sequence number
    session->sequence_number = secure_msg.header.sequence_number;
    
    return true;
}
```

**安全协议特性**：
- **双向认证**：挑战-响应协议（challenge-response protocol）
- **会话管理**：唯一的会话密钥与 ID
- **加密**：带随机 IV 的 AES-CBC
- **完整性**：所有消息的 HMAC 验证
- **重放保护**：序列号验证

## 🧪 **练习题**

### **问题 1：TPM PCR 分析**

**场景**：分析安全启动序列之后的 TPM PCR 值。

**启动后的 PCR 值**：
- PCR 0：引导加载程序哈希
- PCR 1：应用固件哈希
- PCR 2：启动成功指示
- PCR 3-7：平台配置

**问题**：你会如何使用这些 PCR 值验证系统完整性？

**预期方案**：
```
1. PCR 验证：
   - 将 PCR 0 与预期的引导加载程序哈希比较
   - 验证 PCR 1 与固件哈希匹配
   - 确认 PCR 2 指示启动成功

2. 认证（Attestation）：
   - 使用 TPM 身份密钥对 PCR 值签名
   - 将签名后的 PCR 发送给远程验证方
   - 远程验证方与已知良好值比较

3. 完整性检查：
   - 固件的任何改动都会改变 PCR 值
   - 被攻破的系统会有不同的 PCR
   - 远程验证可检测篡改
```

### **问题 2：侧信道攻击分析**

**场景**：分析一个存在漏洞的密码学实现。

**漏洞代码**：
```c
bool check_password(const char* input, const char* correct) {
    for (int i = 0; input[i] != '\0' && correct[i] != '\0'; i++) {
        if (input[i] != correct[i]) {
            return false;  // Early exit reveals password length
        }
    }
    return (input[i] == '\0' && correct[i] == '\0');
}
```

**问题**：存在哪些侧信道漏洞，你会如何修复？

**预期分析**：
```
漏洞：
1. 时序攻击：提前退出泄露密码长度
2. 功耗分析：不同的执行路径
3. 缓存时序：内存访问模式

修复：
1. 恒定时间比较
2. 始终遍历完整密码
3. 使用安全的比较函数
4. 添加随机延迟（不推荐）
```

### **问题 3：安全通信设计**

**场景**：为物联网设备网络设计安全方案。

**需求**：
- 100 个物联网设备
- 中央网关
- 无线通信
- 安全的固件更新

**预期方案**：
```
1. 设备认证：
   - 唯一的设备证书
   - 证书验证链
   - 设备注册协议

2. 通信安全：
   - 所有连接使用 TLS 1.3
   - 证书固定（certificate pinning）
   - 安全密钥交换

3. 固件更新：
   - 签名固件包
   - 安全启动验证
   - 回滚保护
   - 更新认证

4. 网络安全：
   - 设备隔离
   - 流量监控
   - 入侵检测
```

## ✅ **自我评估清单**

### **安全启动** ✅
- [ ] 能实现信任链
- [ ] 理解 TPM 集成
- [ ] 能防止回滚攻击
- [ ] 掌握签名验证

### **密码学** ✅
- [ ] 能实现恒定时间操作
- [ ] 理解侧信道攻击
- [ ] 能设计安全协议
- [ ] 掌握密钥管理原则

### **平台安全** ✅
- [ ] 能使用 ARM TrustZone
- [ ] 理解安全隔离区
- [ ] 能实现内存保护
- [ ] 掌握安全通信

### **安全测试** ✅
- [ ] 能执行安全审计
- [ ] 理解攻击向量
- [ ] 能实现对策
- [ ] 掌握安全最佳实践

## 🔗 **相关主题**
- [[Security_Fundamentals]]
- [[Secure_Boot_Chain_Trust]]
- [[TPM2_Basics]]
- [[Cryptographic_Foundations]]
- [[Platform_Security]]

## 📚 **附加资源**
- **书籍**：《Applied Cryptography》作者 Bruce Schneier
- **在线**：[ARM TrustZone 文档](https://developer.arm.com/ip-products/security-ip/trustzone)
- **练习**：[TPM 2.0 Tools](https://github.com/tpm2-software/tpm2-tools)
- **标准**：[TPM 2.0 规范](https://trustedcomputinggroup.org/resource/tpm-library-specification/)

## 相关页面

- [[Advanced_Hardware_Interview]]
- [[Operating_Systems_Interview]]
- [[IoT_Wireless_Interview]]
- [[System_Integration_Interview]]
- [[00-索引]]

返回索引 [[00-索引]]
