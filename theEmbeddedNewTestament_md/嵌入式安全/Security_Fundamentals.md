---
tags:
  - 嵌入式安全
source: Embedded_Security/Security_Fundamentals.md
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 练习与深度学习
>
> 把这些安全概念作为排名面试题（附模型答案）来练习，并查看交互式深度学习指南。
>
> 👉 **[浏览安全与可靠性问题 →](https://embeddedinterviewlab.com/questions/domain/safety-security-reliability?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=embedded_security)** &nbsp;·&nbsp; **[阅读深入指南 →](https://embeddedinterviewlab.com/topics/secure-boot-and-crypto?utm_source=github&utm_medium=referral&utm_campaign=kb_topic&utm_content=embedded_security)**

---

# 嵌入式安全基础

> **在一个不安全的世界中构建可信系统**  
> 理解嵌入式安全的基本原则及其重要性

---

## 📋 **目录**

- [安全哲学](#安全哲学)
- [威胁模型](#威胁模型)
- [攻击向量](#攻击向量)
- [安全原则](#安全原则)
- [实践实现](#实践实现)

---

## 🛡️ **安全哲学**

### **为什么安全在嵌入式系统中很重要**

想象你在设计一个控制胰岛素给药的医疗设备。如果攻击者能攻破这个设备，他们可能：
- **让患者胰岛素过量**
- **通过禁用设备而拒绝治疗**
- **窃取敏感医疗数据**
- **造成危及生命的系统故障**

这不是科幻小说——这是现代嵌入式系统的现实。

**安全思维**

安全不是让你的系统“黑客免疫”——而是让攻击成本高到不值得尝试。把它想成一栋房子：你无法让它无法闯入，但你可以让它足够难，使盗贼转而寻找更容易的目标。

---

## 🎯 **威胁模型**

### **理解你的对手**

威胁模型（threat model）就像一种安全评估，它问：“谁可能攻击我们，以及如何攻击？”这种理解驱动你的所有安全决策。

#### **攻击者类型**

**1. 脚本小子（Script Kiddies）**
- **动机**（Motivation）：无聊、好奇、炫耀
- **能力**（Capability）：低 —— 使用现有工具与脚本
- **资源**（Resources）：极少 —— 只需一台电脑与互联网
- **风险**（Risk）：低到中 —— 能造成干扰，但无法发动复杂攻击

**2. 网络罪犯（Cybercriminals）**
- **动机**（Motivation）：通过窃取、勒索或出售被盗数据牟取钱财
- **能力**（Capability）：中到高 —— 能开发定制工具
- **资源**（Resources）：可观 —— 能投资于攻击开发
- **风险**（Risk）：高 —— 动机强且资金充足

**3. 国家级行为者（Nation-State Actors）**
- **动机**（Motivation）：间谍、破坏或军事优势
- **能力**（Capability）：极高 —— 能获取零日漏洞利用与先进技术
- **资源**（Resources）：庞大 —— 无上限的预算与时间
- **风险**（Risk）：极高 —— 复杂且持久

**4. 内部人员（Insiders）**
- **动机**（Motivation）：报复、牟利或意识形态
- **能力**（Capability）：高 —— 拥有对系统的合法访问权
- **资源**（Resources）：高 —— 了解系统内部并有访问权
- **风险**（Risk）：极高 —— 最难防范

#### **攻击场景**

**场景 1：医疗设备被攻破**
```
攻击者目标：禁用胰岛素泵
攻击向量：无线通信漏洞利用
影响：患者受害或死亡
难度：中等（需要无线访问）
```

**场景 2：工业控制系统被破坏**
```
攻击者目标：破坏制造过程
攻击向量：网络渗透
影响：生产停机、安全事故
难度：高（需要网络访问）
```

**场景 3：汽车系统被接管**
```
攻击者目标：控制车辆系统
攻击向量：OBD-II 端口或无线
影响：车辆劫持、安全受损
难度：中到高（取决于访问方式）
```

---

## 🚪 **攻击向量**

### **攻击者如何进入**

攻击向量（attack vector）是攻击者用来攻破你系统的路径。理解这些有助于你构建适当的防御。

#### **物理攻击向量**

**1. 直接硬件访问**
```
┌─────────────────────────────────────┐
│           设备外壳（Device Enclosure）│
│  ┌─────────────────────────────┐    │
│  │      电路板（Circuit Board） │    │
│  │  ┌─────────────────────┐    │    │
│  │  │  微控制器（MCU）     │    │    │
│  │  │  ┌───────────────┐  │    │    │
│  │  │  │   CPU 内核    │  │    │    │
│  │  │  └───────────────┘  │    │    │
│  │  └─────────────────────┘    │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

**攻击方法：**
- **芯片拆卸**（Chip removal）与更换
- **总线窃听**（Bus tapping）以监控通信
- **功耗分析**（Power analysis）以提取加密密钥
- **故障注入**（Fault injection）使用电压毛刺或激光脉冲

**2. 接口利用**
```
常见接口：
┌─────────────┬─────────────┬─────────────┐
│  USB 端口   │  串行端口   │  JTAG 端口   │
├─────────────┼─────────────┼─────────────┤
│   容易      │   中等      │   困难      │
│   访问      │   访问      │   访问      │
└─────────────┴─────────────┴─────────────┘
```

**USB 攻击示例：**
```c
// 脆弱的 USB 处理代码
void handle_usb_command(uint8_t* buffer, int length) {
    if (length > 0) {
        // 未校验命令类型
        uint8_t command = buffer[0];
        
        switch (command) {
            case CMD_READ_MEMORY:
                // 允许读取任意内存位置！
                read_memory(buffer[1], buffer[2]);
                break;
            case CMD_WRITE_MEMORY:
                // 允许写入任意内存位置！
                write_memory(buffer[1], buffer[2], buffer[3]);
                break;
        }
    }
}
```

**安全版本：**
```c
// 带校验的安全 USB 处理
void handle_usb_command_secure(uint8_t* buffer, int length) {
    if (length < MIN_COMMAND_LENGTH) {
        return; // 命令长度无效
    }
    
    uint8_t command = buffer[0];
    
    // 校验该命令在当前模式下是否被允许
    if (!is_command_allowed(command, current_security_mode)) {
        return; // 命令不被允许
    }
    
    // 执行前校验参数
    if (!validate_command_parameters(buffer, length)) {
        return; // 参数无效
    }
    
    switch (command) {
        case CMD_READ_MEMORY:
            if (is_memory_region_readable(buffer[1], buffer[2])) {
                read_memory(buffer[1], buffer[2]);
            }
            break;
        case CMD_WRITE_MEMORY:
            if (is_memory_region_writable(buffer[1], buffer[2])) {
                write_memory(buffer[1], buffer[2], buffer[3]);
            }
            break;
    }
}
```

#### **网络攻击向量**

**1. 无线通信**
```
无线攻击类型：
┌─────────────────┬─────────────────┬─────────────────┐
│   窃听（Eavesdropping）│ 重放攻击（Replay）│ 中间人（Man-in-Middle）│
├─────────────────┼─────────────────┼─────────────────┤
│  拦截             │ 记录并           │ 拦截并           │
│  通信             │ 之后重放         │ 修改流量         │
└─────────────────┴─────────────────┴─────────────────┘
```

**2. 网络协议漏洞**
```c
// 脆弱的网络代码
void handle_network_packet(uint8_t* packet, int length) {
    if (length >= 4) {
        uint32_t command = *(uint32_t*)packet;
        
        // 无认证 - 任何人都能发送命令！
        if (command == CMD_SHUTDOWN) {
            system_shutdown();
        } else if (command == CMD_FACTORY_RESET) {
            factory_reset();
        }
    }
}
```

**安全的网络处理：**
```c
// 带认证的安全网络处理
void handle_network_packet_secure(uint8_t* packet, int length) {
    if (length < MIN_PACKET_SIZE) {
        return; // 包太短
    }
    
    // 提取并验证认证
    uint8_t* auth_data = packet;
    uint8_t* payload = packet + AUTH_SIZE;
    
    if (!verify_authentication(auth_data, payload, length - AUTH_SIZE)) {
        return; // 认证失败
    }
    
    // 检查发送者是否被授权
    if (!is_sender_authorized(auth_data)) {
        return; // 发送者未被授权
    }
    
    // 现在处理经过认证的命令
    uint32_t command = *(uint32_t*)payload;
    if (command == CMD_SHUTDOWN && is_shutdown_allowed()) {
        system_shutdown();
    }
}
```

---

## 🔐 **安全原则**

### **安全设计的基础**

这些原则指导所有安全决策，并帮助你构建“设计即安全”的系统。

#### **1. 纵深防御（Defense in Depth）**

不要依赖单一安全措施。构建多层保护：

```
安全层级：
┌─────────────────────────────────────┐
│         应用安全（Application）      │ ← 输入校验、认证
├─────────────────────────────────────┤
│          运行时保护（Runtime）        │ ← 内存保护、沙盒
├─────────────────────────────────────┤
│          操作系统（Operating System） │ ← 进程隔离、文件权限
├─────────────────────────────────────┤
│          硬件安全（Hardware）         │ ← 安全启动、硬件隔离
├─────────────────────────────────────┤
│          物理安全（Physical）         │ ← 防篡改检测、安全外壳
└─────────────────────────────────────┘
```

**示例：多层认证**
```c
// 多重认证因素
typedef struct {
    uint8_t password_hash[32];      // 你知晓的
    uint8_t device_token[16];       // 你拥有的
    uint8_t biometric_template[64]; // 你本人的
} MultiFactorAuth;

bool authenticate_user(const char* password, 
                      const uint8_t* device_token,
                      const uint8_t* biometric_data) {
    
    // 第 1 层：密码验证
    if (!verify_password_hash(password, auth.password_hash)) {
        return false;
    }
    
    // 第 2 层：设备令牌验证
    if (!verify_device_token(device_token, auth.device_token)) {
        return false;
    }
    
    // 第 3 层：生物特征验证
    if (!verify_biometric(biometric_data, auth.biometric_template)) {
        return false;
    }
    
    return true; // 所有层都通过
}
```

#### **2. 最小权限原则（Principle of Least Privilege）**

只给每个组件授予其绝对需要的权限：

```c
// 之前：一切以全权限运行
void process_sensor_data() {
    // 可访问任意内存、任意外设
    read_sensor();
    write_to_display();
    modify_system_config();
    access_network();
}

// 之后：每个功能的权限受限
void process_sensor_data() {
    // 仅允许传感器访问
    read_sensor();
    
    // 请求其他操作的权限
    if (request_permission(PERM_DISPLAY_WRITE)) {
        write_to_display();
    }
    
    if (request_permission(PERM_CONFIG_MODIFY)) {
        modify_system_config();
    }
    
    if (request_permission(PERM_NETWORK_ACCESS)) {
        access_network();
    }
}
```

#### **3. 故障安全（Fail Secure）**

当安全机制失败时，系统应回到安全状态：

```c
// 故障安全认证
bool authenticate_user(const char* username, const char* password) {
    // 任何一步失败，都拒绝访问
    if (!username || !password) {
        return false; // 故障安全：拒绝访问
    }
    
    if (!validate_input(username) || !validate_input(password)) {
        return false; // 故障安全：拒绝访问
    }
    
    if (!check_credentials(username, password)) {
        return false; // 故障安全：拒绝访问
    }
    
    // 仅在一切成功时才授予访问
    return true;
}
```

---

## 🛠️ **实践实现**

### **把安全构建进你的系统**

#### **安全启动实现**

安全启动（Secure Boot）确保只有受信任的代码在你的系统上运行：

```c
// 安全启动序列
typedef struct {
    uint8_t signature[64];      // 数字签名
    uint8_t hash[32];          // 代码哈希
    uint32_t version;          // 版本号
    uint32_t flags;            // 安全标志
} SecureBootHeader;

bool verify_boot_image(uint8_t* image, uint32_t size) {
    SecureBootHeader* header = (SecureBootHeader*)image;
    
    // 第 1 步：验证签名
    if (!verify_signature(header->signature, 
                         header->hash, 
                         sizeof(header->hash))) {
        return false; // 签名验证失败
    }
    
    // 第 2 步：验证代码哈希
    uint8_t calculated_hash[32];
    calculate_hash(image + sizeof(SecureBootHeader), 
                  size - sizeof(SecureBootHeader), 
                  calculated_hash);
    
    if (memcmp(calculated_hash, header->hash, 32) != 0) {
        return false; // 代码哈希不匹配
    }
    
    // 第 3 步：检查版本与标志
    if (header->version < MIN_SUPPORTED_VERSION) {
        return false; // 版本太旧
    }
    
    if (header->flags & FLAG_DEBUG_ENABLED) {
        return false; // 生产环境不允许调试模式
    }
    
    return true; // 所有检查通过
}
```

#### **内存保护（Memory Protection）**

保护内存中的敏感数据：

```c
// 内存保护示例
typedef struct {
    uint8_t encryption_key[32];
    uint8_t user_credentials[64];
    uint8_t system_config[128];
} SecureData;

// 将敏感数据标记为不可交换并加密
void initialize_secure_memory() {
    // 锁定内存页（防止交换）
    mlock(&secure_data, sizeof(secure_data));
    
    // 加密敏感数据
    encrypt_data(&secure_data.encryption_key, sizeof(secure_data.encryption_key));
    encrypt_data(&secure_data.user_credentials, sizeof(secure_data.user_credentials));
    
    // 初始化后将内存设为只读
    mprotect(&secure_data, sizeof(secure_data), PROT_READ);
}

// 不再需要时清除敏感数据
void clear_secure_memory() {
    // 先解密数据
    decrypt_data(&secure_data.encryption_key, sizeof(secure_data.encryption_key));
    
    // 用随机数据覆盖
    secure_random_fill(&secure_data, sizeof(secure_data));
    
    // 用零覆盖
    memset(&secure_data, 0, sizeof(secure_data));
    
    // 解锁内存页
    munlock(&secure_data, sizeof(secure_data));
}
```

#### **输入校验（Input Validation）**

始终校验并清洗输入数据：

```c
// 全面的输入校验
typedef enum {
    VALIDATION_OK,
    VALIDATION_TOO_LONG,
    VALIDATION_INVALID_CHARS,
    VALIDATION_OUT_OF_RANGE,
    VALIDATION_NULL_POINTER
} ValidationResult;

ValidationResult validate_user_input(const char* input, int max_length) {
    // 检查空指针
    if (!input) {
        return VALIDATION_NULL_POINTER;
    }
    
    // 检查长度
    int length = strlen(input);
    if (length > max_length) {
        return VALIDATION_TOO_LONG;
    }
    
    // 检查有效字符（仅限字母数字）
    for (int i = 0; i < length; i++) {
        if (!isalnum(input[i]) && input[i] != '_' && input[i] != '-') {
            return VALIDATION_INVALID_CHARS;
        }
    }
    
    return VALIDATION_OK;
}

// 带校验的安全字符串复制
bool safe_string_copy(char* dest, const char* src, int dest_size) {
    if (!dest || !src || dest_size <= 0) {
        return false;
    }
    
    ValidationResult result = validate_user_input(src, dest_size - 1);
    if (result != VALIDATION_OK) {
        return false;
    }
    
    strncpy(dest, src, dest_size - 1);
    dest[dest_size - 1] = '\0'; // 确保以空字符结尾
    
    return true;
}
```

---

## 🎯 **关键要点**

### **基本原则**

1. **安全是一种思维**（Security is a mindset）—— 像攻击者一样思考
2. **纵深防御**（Defense in depth）—— 多层保护
3. **最小权限**（Least privilege）—— 只授予必要的权限
4. **故障安全**（Fail secure）—— 默认回到安全状态
5. **输入校验**（Input validation）—— 从不信任外部数据

### **安全检查清单**

- [ ] **威胁模型**（Threat model）已完成并记录
- [ ] **攻击向量**（Attack vectors）已识别并缓解
- [ ] **安全启动**（Secure boot）已实现
- [ ] **内存保护**（Memory protection）已启用
- [ ] **所有输入均已输入校验**
- [ ] **敏感操作需要认证**（Authentication）
- [ ] **针对静态与传输中的数据加密**（Encryption）
- [ ] **安全通信协议**（Secure communication）已实现
- [ ] **物理安全**（Physical security）措施已到位
- [ ] **安全测试**（Security testing）已执行

### **应避免的常见错误**

1. **通过隐藏实现安全**（Security through obscurity）—— 不要依赖隐藏事物
2. **单点故障**（Single point of failure）—— 在安全中构建冗余
3. **忽视物理安全**—— 硬件访问可绕过软件
4. **弱认证**（Weak authentication）—— 使用强、多因素认证
5. **无安全更新**—— 规划安全维护

---

## 📚 **更多资源**

- **《Security Engineering》by Ross Anderson** —— 全面安全指南
- **《Building Secure and Reliable Systems》by Google** —— 生产安全实践
- **OWASP 嵌入式应用安全** —— 嵌入式安全指南

---

**下一主题**：[[Secure_Boot_Chain_Trust]] → [[TPM2_Basics]]
