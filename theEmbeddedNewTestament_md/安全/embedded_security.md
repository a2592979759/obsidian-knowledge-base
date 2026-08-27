---
tags:
  - 安全
  - 系统集成
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Security/embedded_security.md
created: 2026-08-27
---

# 嵌入式安全指南(Embedded Security Guide)

## 概述(Overview)

嵌入式安全对于保护设备、数据和系统免遭未经授权的访问、篡改和攻击至关重要。本指南涵盖了嵌入式系统必不可少的核心安全概念、实现和最佳实践。

---

## 概念 → 为什么重要 → 最小示例 → 动手试试 → 要点(Takeaways)

**概念(Concept)**：嵌入式安全是关于保护设备和数据免遭未经授权的访问、篡改和攻击。它不只是加密，而是构建一个全面的安全架构，包括安全启动(secure boot)、加密实现、侧信道攻击(side-channel attack)防护和安全的通信协议。

**为什么重要(Why it matters)**：嵌入式系统中的安全漏洞可能产生严重后果，从数据窃取到人身伤害。许多嵌入式设备运行在关键基础设施、医疗设备或汽车系统中，安全故障可能是灾难性的。理解嵌入式安全对于构建值得信赖的系统至关重要。

**最小示例(Minimal example)**：一个简单的安全启动实现，在固件执行前验证其完整性，演示信任链(chain of trust)的概念。

**动手试试(Try it)**：实现一个基本的 AES 加密函数，然后修改它对时序攻击(timing attacks)具有抵抗力，观察安全考量如何影响实现复杂度。

**要点(Takeaways)**：嵌入式安全需要纵深防御(defense-in-depth)的方法、对加密原理的理解以及对各种攻击向量的认知。安全性应从系统设计之初就纳入，而不是事后添加。

---

## 目录(Table of Contents)

1. [安全启动与信任链](#安全启动与信任链)
2. [加密实现](#加密实现)
3. [侧信道攻击防护](#侧信道攻击防护)
4. [硬件安全模块（HSM）](#硬件安全模块hsm)
5. [安全通信协议](#安全通信协议)
6. [ARM TrustZone](#arm-trustzone)
7. [加密算法](#加密算法)

---

## 安全启动与信任链(Secure Boot and Chain of Trust)

### 什么是安全启动？

安全启动确保只有可信的、经过认证的代码才能在设备上执行。它从硬件信任根到应用软件建立一条信任链。

### 信任链组件

#### 1. 硬件信任根
```c
// Example: Hardware root of trust implementation
typedef struct {
    uint8_t public_key[256];
    uint8_t device_id[32];
    uint8_t secure_flags;
} hw_root_of_trust_t;

// Hardware root of trust verification
int verify_hw_root_of_trust(void) {
    hw_root_of_trust_t *hw_rot = (hw_root_of_trust_t*)HW_ROT_ADDRESS;
    
    // Verify hardware root of trust signature
    if (!verify_signature(hw_rot->public_key, hw_rot, sizeof(hw_root_of_trust_t))) {
        return -1;  // Verification failed
    }
    
    return 0;  // Verification successful
}
```

#### 2. Bootloader 验证
```c
// Example: Secure bootloader implementation
int secure_bootloader(void) {
    // 1. Verify hardware root of trust
    if (verify_hw_root_of_trust() != 0) {
        return -1;
    }
    
    // 2. Verify bootloader signature
    if (verify_bootloader_signature() != 0) {
        return -1;
    }
    
    // 3. Verify application signature
    if (verify_application_signature() != 0) {
        return -1;
    }
    
    // 4. Jump to verified application
    jump_to_application();
    
    return 0;
}

// Bootloader signature verification
int verify_bootloader_signature(void) {
    uint8_t *bootloader_addr = (uint8_t*)BOOTLOADER_ADDRESS;
    uint8_t *signature = (uint8_t*)BOOTLOADER_SIGNATURE_ADDRESS;
    
    // Calculate hash of bootloader
    uint8_t hash[SHA256_DIGEST_SIZE];
    sha256_calculate(bootloader_addr, BOOTLOADER_SIZE, hash);
    
    // Verify signature using public key
    return verify_signature(hash, signature, SHA256_DIGEST_SIZE);
}
```

#### 3. 应用验证
```c
// Example: Application signature verification
int verify_application_signature(void) {
    uint8_t *app_addr = (uint8_t*)APPLICATION_ADDRESS;
    uint8_t *signature = (uint8_t*)APP_SIGNATURE_ADDRESS;
    
    // Calculate hash of application
    uint8_t hash[SHA256_DIGEST_SIZE];
    sha256_calculate(app_addr, APPLICATION_SIZE, hash);
    
    // Verify signature
    return verify_signature(hash, signature, SHA256_DIGEST_SIZE);
}
```

### 安全启动实现

#### 启动序列
```c
// Example: Complete secure boot sequence
void secure_boot_sequence(void) {
    // 1. Initialize security hardware
    security_hw_init();
    
    // 2. Verify hardware root of trust
    if (verify_hw_root_of_trust() != 0) {
        security_failure_handler();
    }
    
    // 3. Verify bootloader
    if (verify_bootloader_signature() != 0) {
        security_failure_handler();
    }
    
    // 4. Verify application
    if (verify_application_signature() != 0) {
        security_failure_handler();
    }
    
    // 5. Set secure flags
    set_secure_flags();
    
    // 6. Jump to application
    jump_to_application();
}

// Security failure handler
void security_failure_handler(void) {
    // Clear sensitive data
    clear_sensitive_data();
    
    // Disable debug interfaces
    disable_debug_interfaces();
    
    // Enter secure failure mode
    while (1) {
        // Wait for reset or secure recovery
        watchdog_reset();
    }
}
```

---

## 加密实现(Cryptographic Implementations)

### 对称加密

#### AES 实现
```c
// Example: AES-128 encryption implementation
typedef struct {
    uint32_t round_keys[44];
} aes_context_t;

// AES key expansion
void aes_key_expansion(const uint8_t *key, aes_context_t *ctx) {
    uint32_t *rk = ctx->round_keys;
    uint32_t temp;
    int i = 0;
    
    // Copy initial key
    for (i = 0; i < 4; i++) {
        rk[i] = ((uint32_t)key[4*i] << 24) |
                ((uint32_t)key[4*i+1] << 16) |
                ((uint32_t)key[4*i+2] << 8) |
                ((uint32_t)key[4*i+3]);
    }
    
    // Generate round keys
    for (i = 4; i < 44; i++) {
        temp = rk[i-1];
        if (i % 4 == 0) {
            temp = sub_word(rot_word(temp)) ^ rcon[i/4];
        }
        rk[i] = rk[i-4] ^ temp;
    }
}

// AES encryption
void aes_encrypt(const aes_context_t *ctx, const uint8_t *input, uint8_t *output) {
    uint8_t state[16];
    int round;
    
    // Copy input to state
    memcpy(state, input, 16);
    
    // Add round key
    add_round_key(state, ctx->round_keys, 0);
    
    // Main rounds
    for (round = 1; round < 10; round++) {
        sub_bytes(state);
        shift_rows(state);
        mix_columns(state);
        add_round_key(state, ctx->round_keys, round);
    }
    
    // Final round
    sub_bytes(state);
    shift_rows(state);
    add_round_key(state, ctx->round_keys, 10);
    
    // Copy state to output
    memcpy(output, state, 16);
}
```

#### AES 解密
```c
// Example: AES-128 decryption implementation
void aes_decrypt(const aes_context_t *ctx, const uint8_t *input, uint8_t *output) {
    uint8_t state[16];
    int round;
    
    // Copy input to state
    memcpy(state, input, 16);
    
    // Add round key
    add_round_key(state, ctx->round_keys, 10);
    
    // Main rounds
    for (round = 9; round > 0; round--) {
        inv_shift_rows(state);
        inv_sub_bytes(state);
        add_round_key(state, ctx->round_keys, round);
        inv_mix_columns(state);
    }
    
    // Final round
    inv_shift_rows(state);
    inv_sub_bytes(state);
    add_round_key(state, ctx->round_keys, 0);
    
    // Copy state to output
    memcpy(output, state, 16);
}
```

### 非对称加密

#### RSA 实现
```c
// Example: RSA key generation and encryption
typedef struct {
    uint32_t n[64];  // Modulus
    uint32_t e[64];  // Public exponent
    uint32_t d[64];  // Private exponent
    uint32_t p[32];  // Prime p
    uint32_t q[32];  // Prime q
    uint32_t dp[32]; // d mod (p-1)
    uint32_t dq[32]; // d mod (q-1)
    uint32_t qinv[32]; // q^(-1) mod p
} rsa_context_t;

// RSA key generation
int rsa_generate_key(rsa_context_t *ctx, int bits) {
    // Generate two large prime numbers
    if (generate_prime(ctx->p, bits/2) != 0) {
        return -1;
    }
    
    if (generate_prime(ctx->q, bits/2) != 0) {
        return -1;
    }
    
    // Calculate n = p * q
    multiply(ctx->n, ctx->p, ctx->q);
    
    // Calculate φ(n) = (p-1) * (q-1)
    uint32_t phi[64];
    subtract(phi, ctx->p, 1);
    uint32_t temp[64];
    subtract(temp, ctx->q, 1);
    multiply(phi, phi, temp);
    
    // Choose public exponent e
    ctx->e[0] = 65537;  // Common choice
    
    // Calculate private exponent d
    mod_inverse(ctx->d, ctx->e, phi);
    
    return 0;
}

// RSA encryption
int rsa_encrypt(const rsa_context_t *ctx, const uint8_t *input, uint8_t *output) {
    uint32_t message[64];
    uint32_t result[64];
    
    // Convert input to integer
    bytes_to_int(message, input, 32);
    
    // Encrypt: c = m^e mod n
    mod_exp(result, message, ctx->e, ctx->n);
    
    // Convert result to bytes
    int_to_bytes(output, result, 64);
    
    return 0;
}
```

### 哈希函数

#### SHA-256 实现
```c
// Example: SHA-256 hash function
typedef struct {
    uint32_t state[8];
    uint32_t count[2];
    uint8_t buffer[64];
} sha256_context_t;

// SHA-256 initialization
void sha256_init(sha256_context_t *ctx) {
    ctx->state[0] = 0x6a09e667;
    ctx->state[1] = 0xbb67ae85;
    ctx->state[2] = 0x3c6ef372;
    ctx->state[3] = 0xa54ff53a;
    ctx->state[4] = 0x510e527f;
    ctx->state[5] = 0x9b05688c;
    ctx->state[6] = 0x1f83d9ab;
    ctx->state[7] = 0x5be0cd19;
    
    ctx->count[0] = 0;
    ctx->count[1] = 0;
}

// SHA-256 update
void sha256_update(sha256_context_t *ctx, const uint8_t *data, uint32_t len) {
    uint32_t i;
    
    for (i = 0; i < len; i++) {
        ctx->buffer[ctx->count[0]] = data[i];
        ctx->count[0]++;
        
        if (ctx->count[0] == 64) {
            sha256_transform(ctx, ctx->buffer);
            ctx->count[0] = 0;
        }
    }
}

// SHA-256 finalization
void sha256_final(sha256_context_t *ctx, uint8_t *hash) {
    uint8_t finalcount[8];
    int i;
    
    // Convert count to bytes
    for (i = 0; i < 8; i++) {
        finalcount[i] = (uint8_t)((ctx->count[(i >= 4 ? 0 : 1)] >> ((3-(i & 3)) * 8)) & 255);
    }
    
    // Add padding
    sha256_update(ctx, (uint8_t*)"\x80", 1);
    while (ctx->count[0] != 56) {
        sha256_update(ctx, (uint8_t*)"\x00", 1);
    }
    
    // Add length
    sha256_update(ctx, finalcount, 8);
    
    // Convert state to hash
    for (i = 0; i < 8; i++) {
        hash[i*4] = (uint8_t)((ctx->state[i] >> 24) & 255);
        hash[i*4+1] = (uint8_t)((ctx->state[i] >> 16) & 255);
        hash[i*4+2] = (uint8_t)((ctx->state[i] >> 8) & 255);
        hash[i*4+3] = (uint8_t)(ctx->state[i] & 255);
    }
}
```

---

## 侧信道攻击防护(Side-channel Attack Prevention)

### 什么是侧信道攻击？

侧信道攻击利用通过系统物理特征泄露的信息，例如：
- 功耗
- 电磁辐射
- 时序变化
- 缓存行为

### 功耗分析攻击

#### 简单功耗分析（SPA）
```c
// Vulnerable implementation - leaks information through power consumption
int vulnerable_aes_encrypt(const uint8_t *key, const uint8_t *input, uint8_t *output) {
    aes_context_t ctx;
    aes_key_expansion(key, &ctx);
    
    // This leaks information about the key through power consumption
    for (int i = 0; i < 16; i++) {
        output[i] = input[i] ^ key[i];  // Power consumption varies with key bits
    }
    
    return 0;
}

// Protected implementation - constant power consumption
int protected_aes_encrypt(const uint8_t *key, const uint8_t *input, uint8_t *output) {
    aes_context_t ctx;
    aes_key_expansion(key, &ctx);
    
    // Use constant-time operations
    for (int i = 0; i < 16; i++) {
        // Constant-time XOR operation
        output[i] = constant_time_xor(input[i], key[i]);
    }
    
    return 0;
}
```

#### 差分功耗分析（DPA）
```c
// DPA-resistant implementation
int dpa_resistant_aes_encrypt(const uint8_t *key, const uint8_t *input, uint8_t *output) {
    aes_context_t ctx;
    uint8_t masked_key[16];
    uint8_t masked_input[16];
    
    // Apply random masking
    for (int i = 0; i < 16; i++) {
        uint8_t mask = generate_random_byte();
        masked_key[i] = key[i] ^ mask;
        masked_input[i] = input[i] ^ mask;
    }
    
    // Perform encryption with masked values
    aes_key_expansion(masked_key, &ctx);
    aes_encrypt(&ctx, masked_input, output);
    
    // Remove masking from output
    for (int i = 0; i < 16; i++) {
        output[i] ^= masked_key[i];
    }
    
    return 0;
}
```

### 时序攻击防护

#### 恒定时间操作
```c
// Vulnerable comparison - leaks timing information
int vulnerable_compare(const uint8_t *a, const uint8_t *b, int len) {
    for (int i = 0; i < len; i++) {
        if (a[i] != b[i]) {
            return 0;  // Early exit leaks timing information
        }
    }
    return 1;
}

// Constant-time comparison
int constant_time_compare(const uint8_t *a, const uint8_t *b, int len) {
    uint8_t result = 0;
    
    for (int i = 0; i < len; i++) {
        result |= a[i] ^ b[i];  // Always perform all operations
    }
    
    return result == 0;
}

// Constant-time conditional copy
void constant_time_conditional_copy(uint8_t *dst, const uint8_t *src, int len, int condition) {
    uint8_t mask = condition ? 0xFF : 0x00;
    
    for (int i = 0; i < len; i++) {
        dst[i] = (dst[i] & ~mask) | (src[i] & mask);
    }
}
```

### 缓存攻击防护

#### 抗缓存时序的实现
```c
// Cache-timing resistant AES S-box lookup
uint8_t cache_resistant_sbox_lookup(uint8_t index) {
    uint8_t sbox[256];
    uint8_t result = 0;
    
    // Load entire S-box into cache
    for (int i = 0; i < 256; i++) {
        // Always access all elements to prevent cache timing
        uint8_t temp = sbox[i];
        if (i == index) {
            result = temp;
        }
    }
    
    return result;
}
```

---

## 硬件安全模块（HSM）

### 什么是 HSM？

硬件安全模块（HSM，Hardware Security Module）是一种物理设备，提供对加密密钥和操作的安全存储和处理。

### HSM 集成

#### HSM 接口
```c
// Example: HSM interface implementation
typedef struct {
    uint32_t hsm_id;
    uint8_t public_key[256];
    uint8_t certificate[1024];
} hsm_info_t;

// HSM initialization
int hsm_init(void) {
    // Initialize HSM hardware
    if (hsm_hw_init() != 0) {
        return -1;
    }
    
    // Verify HSM integrity
    if (hsm_verify_integrity() != 0) {
        return -1;
    }
    
    // Load HSM configuration
    if (hsm_load_config() != 0) {
        return -1;
    }
    
    return 0;
}

// HSM key generation
int hsm_generate_key(uint32_t key_id, uint32_t key_type, uint32_t key_size) {
    hsm_command_t cmd;
    hsm_response_t resp;
    
    cmd.command = HSM_CMD_GENERATE_KEY;
    cmd.key_id = key_id;
    cmd.key_type = key_type;
    cmd.key_size = key_size;
    
    return hsm_send_command(&cmd, &resp);
}

// HSM encryption
int hsm_encrypt(uint32_t key_id, const uint8_t *input, uint8_t *output, uint32_t len) {
    hsm_command_t cmd;
    hsm_response_t resp;
    
    cmd.command = HSM_CMD_ENCRYPT;
    cmd.key_id = key_id;
    cmd.data_len = len;
    memcpy(cmd.data, input, len);
    
    if (hsm_send_command(&cmd, &resp) != 0) {
        return -1;
    }
    
    memcpy(output, resp.data, resp.data_len);
    return 0;
}
```

### TPM 集成

#### TPM 2.0 实现
```c
// Example: TPM 2.0 integration
typedef struct {
    uint32_t tpm_id;
    uint8_t ek_public[256];
    uint8_t srk_public[256];
} tpm_context_t;

// TPM initialization
int tpm_init(tpm_context_t *ctx) {
    // Initialize TPM hardware
    if (tpm_hw_init() != 0) {
        return -1;
    }
    
    // Start TPM
    if (tpm_startup() != 0) {
        return -1;
    }
    
    // Get TPM capabilities
    if (tpm_get_capabilities() != 0) {
        return -1;
    }
    
    return 0;
}

// TPM key creation
int tpm2_create_key(tpm_context_t *ctx, uint32_t key_handle, uint32_t key_type) {
    tpm2_command_t cmd;
    tpm2_response_t resp;
    
    cmd.header.tag = TPM2_ST_NO_SESSIONS;
    cmd.header.command_code = TPM2_CC_CreatePrimary;
    cmd.header.param_size = sizeof(cmd);
    
    cmd.primary_handle = TPM2_RH_ENDORSEMENT;
    cmd.in_sensitive.sensitive.user_auth.size = 0;
    cmd.in_public.public_area.type = key_type;
    cmd.in_public.public_area.parameters.rsa_detail.key_bits = 2048;
    
    return tpm2_send_command(&cmd, &resp);
}
```

---

## 安全通信协议

### TLS/SSL 实现

#### TLS 握手
```c
// Example: Simplified TLS handshake
typedef struct {
    uint8_t client_random[32];
    uint8_t server_random[32];
    uint8_t master_secret[48];
    uint8_t session_key[32];
} tls_session_t;

// TLS handshake
int tls_handshake(tls_session_t *session) {
    // 1. Client Hello
    if (tls_client_hello(session) != 0) {
        return -1;
    }
    
    // 2. Server Hello
    if (tls_server_hello(session) != 0) {
        return -1;
    }
    
    // 3. Certificate exchange
    if (tls_certificate_exchange(session) != 0) {
        return -1;
    }
    
    // 4. Key exchange
    if (tls_key_exchange(session) != 0) {
        return -1;
    }
    
    // 5. Finished
    if (tls_finished(session) != 0) {
        return -1;
    }
    
    return 0;
}

// TLS encryption
int tls_encrypt(tls_session_t *session, const uint8_t *input, uint8_t *output, uint32_t len) {
    uint8_t iv[16];
    uint8_t mac[32];
    
    // Generate IV
    generate_random(iv, 16);
    
    // Calculate MAC
    hmac_calculate(session->session_key, input, len, mac);
    
    // Encrypt data
    aes_encrypt_cbc(session->session_key, iv, input, output, len);
    
    // Append MAC
    memcpy(output + len, mac, 32);
    
    return 0;
}
```

### 安全引导(Secure Bootstrapping)

#### 设备认证
```c
// Example: Device authentication protocol
typedef struct {
    uint8_t device_id[32];
    uint8_t challenge[32];
    uint8_t response[64];
    uint8_t session_key[32];
} device_auth_t;

// Device authentication
int device_authenticate(device_auth_t *auth) {
    // 1. Generate challenge
    generate_random(auth->challenge, 32);
    
    // 2. Send challenge to device
    if (send_challenge(auth->challenge) != 0) {
        return -1;
    }
    
    // 3. Receive response
    if (receive_response(auth->response) != 0) {
        return -1;
    }
    
    // 4. Verify response
    if (verify_device_response(auth->challenge, auth->response) != 0) {
        return -1;
    }
    
    // 5. Generate session key
    generate_session_key(auth->session_key);
    
    return 0;
}
```

---

## ARM TrustZone

### 什么是 ARM TrustZone？

ARM TrustZone 通过在同一处理器内创建安全世界和非安全世界，提供基于硬件的安全性。

### TrustZone 实现

#### 安全世界设置
```c
// Example: TrustZone secure world implementation
typedef struct {
    uint32_t secure_flags;
    uint8_t secure_key[32];
    uint8_t secure_data[1024];
} secure_world_t;

// Secure world initialization
void secure_world_init(void) {
    // Set up secure world
    configure_secure_world();
    
    // Initialize secure peripherals
    init_secure_peripherals();
    
    // Load secure applications
    load_secure_applications();
}

// Secure world entry
void secure_world_entry(void) {
    // Switch to secure world
    smc_instruction(SMC_SECURE_ENTRY);
    
    // Handle secure world operations
    while (1) {
        uint32_t command = get_secure_command();
        handle_secure_command(command);
    }
}

// Secure world command handler
void handle_secure_command(uint32_t command) {
    switch (command) {
        case SECURE_CMD_ENCRYPT:
            secure_encrypt();
            break;
        case SECURE_CMD_DECRYPT:
            secure_decrypt();
            break;
        case SECURE_CMD_KEY_GEN:
            secure_key_generation();
            break;
        default:
            secure_error_handler();
            break;
    }
}
```

#### 非安全世界接口
```c
// Example: Non-secure world interface
int non_secure_encrypt(const uint8_t *input, uint8_t *output, uint32_t len) {
    // Prepare parameters for secure world
    secure_params_t params;
    params.command = SECURE_CMD_ENCRYPT;
    params.input = input;
    params.output = output;
    params.length = len;
    
    // Call secure world
    smc_instruction(SMC_SECURE_CALL, &params);
    
    return params.result;
}

// SMC (Secure Monitor Call) implementation
void smc_instruction(uint32_t function_id, void *params) {
    // Set up registers for SMC
    __asm volatile (
        "mov r0, %0\n"
        "mov r1, %1\n"
        "smc #0\n"
        : : "r" (function_id), "r" (params) : "r0", "r1"
    );
}
```

---

## 加密算法

### AES-GCM 实现

#### GCM 模式
```c
// Example: AES-GCM implementation
typedef struct {
    aes_context_t aes_ctx;
    uint8_t h[16];
    uint8_t j0[16];
    uint8_t s[16];
} aes_gcm_context_t;

// AES-GCM initialization
void aes_gcm_init(aes_gcm_context_t *ctx, const uint8_t *key, const uint8_t *iv) {
    // Initialize AES context
    aes_key_expansion(key, &ctx->aes_ctx);
    
    // Calculate H = E(K, 0^128)
    uint8_t zero[16] = {0};
    aes_encrypt(&ctx->aes_ctx, zero, ctx->h);
    
    // Calculate J0
    if (iv_len == 12) {
        memcpy(ctx->j0, iv, 12);
        ctx->j0[15] = 1;
    } else {
        // GHASH(H, 0^s || IV || 0^t)
        ghash(ctx->h, iv, iv_len, ctx->j0);
    }
    
    // Initialize S
    memset(ctx->s, 0, 16);
}

// AES-GCM encryption
void aes_gcm_encrypt(aes_gcm_context_t *ctx, const uint8_t *input, uint8_t *output, uint32_t len) {
    uint8_t y[16];
    uint32_t i;
    
    // Y0 = E(K, J0)
    aes_encrypt(&ctx->aes_ctx, ctx->j0, y);
    
    // Encrypt data
    for (i = 0; i < len; i++) {
        if (i % 16 == 0 && i > 0) {
            // Increment counter
            increment_counter(ctx->j0);
            aes_encrypt(&ctx->aes_ctx, ctx->j0, y);
        }
        output[i] = input[i] ^ y[i % 16];
    }
    
    // Calculate authentication tag
    ghash_calculate(ctx->h, output, len, ctx->s);
    aes_encrypt(&ctx->aes_ctx, ctx->j0, y);
    xor_bytes(ctx->s, y, ctx->s, 16);
}
```

### ChaCha20-Poly1305 实现

#### ChaCha20 流密码
```c
// Example: ChaCha20 implementation
typedef struct {
    uint32_t state[16];
    uint8_t key[32];
    uint8_t nonce[12];
} chacha20_context_t;

// ChaCha20 initialization
void chacha20_init(chacha20_context_t *ctx, const uint8_t *key, const uint8_t *nonce) {
    // Initialize state
    ctx->state[0] = 0x61707865;  // "expa"
    ctx->state[1] = 0x3320646e;  // "nd 3"
    ctx->state[2] = 0x79622d32;  // "2-by"
    ctx->state[3] = 0x6b206574;  // "te k"
    
    // Load key
    for (int i = 0; i < 8; i++) {
        ctx->state[4+i] = ((uint32_t)key[4*i] << 24) |
                          ((uint32_t)key[4*i+1] << 16) |
                          ((uint32_t)key[4*i+2] << 8) |
                          ((uint32_t)key[4*i+3]);
    }
    
    // Load nonce
    ctx->state[12] = 0;
    ctx->state[13] = 0;
    ctx->state[14] = ((uint32_t)nonce[0] << 24) |
                     ((uint32_t)nonce[1] << 16) |
                     ((uint32_t)nonce[2] << 8) |
                     ((uint32_t)nonce[3]);
    ctx->state[15] = ((uint32_t)nonce[4] << 24) |
                     ((uint32_t)nonce[5] << 16) |
                     ((uint32_t)nonce[6] << 8) |
                     ((uint32_t)nonce[7]);
}

// ChaCha20 encryption
void chacha20_encrypt(chacha20_context_t *ctx, const uint8_t *input, uint8_t *output, uint32_t len) {
    uint8_t keystream[64];
    uint32_t block_counter = 0;
    
    for (uint32_t i = 0; i < len; i += 64) {
        // Generate keystream block
        chacha20_block(ctx, block_counter, keystream);
        
        // XOR with input
        uint32_t block_len = (len - i < 64) ? (len - i) : 64;
        for (uint32_t j = 0; j < block_len; j++) {
            output[i + j] = input[i + j] ^ keystream[j];
        }
        
        block_counter++;
    }
}
```

---

## 安全最佳实践

### 通用安全准则
1. **使用安全的随机数生成器** - 避免可预测的值
2. **实现正确的密钥管理** - 安全的存储和轮换
3. **使用恒定时间操作** - 防止时序攻击
4. **验证所有输入** - 防止缓冲区溢出和注入攻击
5. **实现安全通信** - 使用 TLS/SSL 进行数据传输

### 安全检查清单
- [ ] 实现安全启动和信任链
- [ ] 正确使用加密算法
- [ ] 保护免受侧信道攻击
- [ ] 实现安全的密钥存储
- [ ] 使用安全的通信协议
- [ ] 验证所有输入和输出
- [ ] 实现安全更新机制
- [ ] 监控安全事件

### 常见安全错误
1. **使用弱加密算法** - 避免已废弃的算法
2. **硬编码密钥** - 使用安全的密钥存储
3. **忽视侧信道攻击** - 实现恒定时间操作
4. **不验证输入** - 始终验证并净化输入
5. **使用可预测的随机数** - 使用加密安全 RNG

---

## 资源(Resources)

### 标准与规范
- [NIST Cryptographic Standards](https://www.nist.gov/cryptography)
- [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) - 加密模块的安全要求
- [Common Criteria](https://www.commoncriteriaportal.org/) - 安全评估框架

### 工具与库
- [mbed TLS](https://tls.mbed.org/) - 开源 SSL/TLS 库
- [OpenSSL](https://www.openssl.org/) - 开源加密库
- [wolfSSL](https://www.wolfssl.com/) - 嵌入式 SSL/TLS 库

### 书籍与参考资料
- "Applied Cryptography" by Bruce Schneier
- "Cryptography Engineering" by Ferguson, Schneier, and Kohno
- "The Art of Deception" by Kevin Mitnick

### 在线资源
- [Cryptography Stack Exchange](https://crypto.stackexchange.com/)
- [Security Stack Exchange](https://security.stackexchange.com/)
- [OWASP Embedded Application Security](https://owasp.org/www-project-embedded-application-security/)

---

## 引导实验(Guided Labs)

### 实验 1：安全启动实现
**目标**：实现一个基本的、在执行前验证固件完整性的安全启动机制。

**设置**：创建一个简单的 bootloader，在跳转到主应用前验证数字签名。

**步骤**：
1. 实现一个基本的 SHA-256 哈希函数
2. 创建一个简单的数字签名验证函数
3. 修改 bootloader 以验证应用签名
4. 使用有效和无效签名进行测试
5. 观察验证失败时系统行为

**预期结果**：理解安全启动如何建立信任链并防止未授权代码执行。

### 实验 2：侧信道攻击防护
**目标**：学习如何实现抵抗时序攻击的加密函数。

**设置**：实现 AES 加密，然后修改为恒定时间。

**步骤**：
1. 使用查找表实现基本的 AES 加密
2. 剖析函数以识别时序变化
3. 对敏感操作实现恒定时间操作
4. 重新剖析以验证时序一致性
5. 使用各种输入模式测试以确保抵抗力

**预期结果**：理解侧信道攻击如何工作以及如何实现对策。

### 实验 3：安全通信协议
**目标**：使用对称加密实现基本的安全通信协议。

**设置**：创建带加密和认证的简单客户端-服务器通信系统。

**步骤**：
1. 实现 AES 加密以保护数据机密性
2. 添加 HMAC 进行消息认证
3. 实现简单的密钥交换协议
4. 使用各种攻击场景测试协议
5. 分析实现的安全属性

**预期结果**：理解如何构建安全通信协议以及正确密钥管理的重要性。

---

## 自我检查(Check Yourself)

### 理解检查
- [ ] 你能解释安全启动中信任链的概念吗？
- [ ] 你理解对称和非对称加密的区别吗？
- [ ] 你能解释侧信道攻击如何工作以及如何防范吗？
- [ ] 你理解加密哈希函数在安全中的作用吗？
- [ ] 你能解释安全通信协议的概念吗？

### 应用检查
- [ ] 你能实现基本的安全启动机制吗？
- [ ] 你知道如何实现恒定时间加密操作吗？
- [ ] 你能设计安全的密钥管理系统吗？
- [ ] 你理解如何实现安全通信协议吗？
- [ ] 你能评估不同设计选择的安全影响吗？

### 分析检查
- [ ] 你能分析安全架构的潜在漏洞吗？
- [ ] 你理解安全与性能之间的权衡吗？
- [ ] 你能评估不同加密算法的安全强度吗？
- [ ] 你知道如何实现安全更新机制吗？
- [ ] 你能评估安全故障对系统可靠性的影响吗？

---

## 交叉链接

### 相关主题
- **[[Build_Systems]]**：将安全集成到构建和部署过程
- **[[performance_optimization]]**：在安全需求与性能约束之间取得平衡
- **[[Secure_Communication]]**：实现安全通信协议
- **[[FreeRTOS_Basics]]**：将安全与实时约束相结合

### 进一步阅读
- **NIST 加密标准**：官方的加密标准和指南
- **Common Criteria**：安全评估的国际标准
- **OWASP 嵌入式安全**：嵌入式系统的安全指南
- **ARM 安全文档**：TrustZone 和安全特性文档

### 行业标准
- **FIPS 140-2**：加密模块的安全要求
- **Common Criteria**：安全评估框架
- **ISO 27001**：信息安全管理体系
- **IEC 62443**：工业自动化和控制系统的安全
- **SAE J3061**：信息物理车辆系统的网络安全指南
