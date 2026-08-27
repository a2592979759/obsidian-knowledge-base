---
tags:
  - 调试
source: Debugging/README.md
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 练习与深度学习
>
> 把这些调试 / 测试概念作为排名面试题（附模型答案）来练习，并查看交互式深度学习指南。
>
> 👉 **[浏览调试与测试问题 →](https://embeddedinterviewlab.com/questions/domain/debugging-testing-tools?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=debugging)** &nbsp;·&nbsp; **[浏览调试与测试指南 →](https://embeddedinterviewlab.com/categories/debugging-testing-tools?utm_source=github&utm_medium=referral&utm_campaign=kb_domain&utm_content=debugging)**

---

# 嵌入式调试指南

## 概述
与桌面应用相比，调试嵌入式系统需要一套独特的工具与技术。本指南涵盖嵌入式软件工程师使用的必备调试方法、工具与策略。

## 目录
1. [JTAG 调试](#jtag-调试)
2. [逻辑分析仪的使用](#逻辑分析仪的使用)
3. [示波器测量](#示波器测量)
4. [代码覆盖率与静态分析](#代码覆盖率与静态分析)
5. [嵌入式系统的单元测试](#嵌入式系统的单元测试)
6. [硬件在环测试](#硬件在环测试)
7. [性能剖析](#性能剖析)

---

## JTAG 调试

### 什么是 JTAG？
JTAG（联合测试行动组，Joint Test Action Group）是调试嵌入式系统的标准接口。它允许你：
- 在代码中设置断点（breakpoint）
- 检查并修改寄存器与内存
- 单步执行代码
- 将固件下载到目标设备

### JTAG 调试设置

#### 硬件要求
- JTAG 调试器（例如 J-Link、ST-Link、OpenOCD）
- 带 JTAG 接口的目标板
- 调试线缆与连接器

#### 软件设置
```bash
# 安装 OpenOCD（开源 JTAG 调试器）
sudo apt-get install openocd

# 安装 GDB（GNU 调试器）
sudo apt-get install gdb-multiarch
```

#### 基本 JTAG 命令
```bash
# 启动 OpenOCD 服务器
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg

# 连接 GDB 到 OpenOCD
gdb-multiarch
(gdb) target remote localhost:3333
(gdb) monitor reset halt
(gdb) load firmware.elf
(gdb) continue
```

### 常见的 JTAG 调试场景

#### 1. 设置断点
```c
// 示例：调试 UART 通信问题
void uart_send_byte(uint8_t data) {
    // 在此处设置断点以检查数据
    while (!(UART->SR & UART_SR_TXE));  // 等待 TX 空
    UART->DR = data;  // 发送数据
}
```

#### 2. 检查寄存器
```bash
(gdb) info registers
(gdb) print/x $r0
(gdb) print/x $pc
(gdb) print/x $sp
```

#### 3. 内存检查
```bash
(gdb) x/16x 0x20000000  # 检查地址处的 16 个字节
(gdb) print *((int*)0x20000000)  # 按整数打印
```

### JTAG 调试最佳实践
1. **谨慎使用硬件断点** - 它们是有限资源
2. **在函数入口/出口设置断点** 以获得更好的控制
3. **使用条件断点** 以避免在每次迭代时停下
4. **监控栈使用** 以防止栈溢出
5. **使用监视点（watchpoint）** 以监控变量变化

---

## 逻辑分析仪的使用

### 什么是逻辑分析仪？
逻辑分析仪是一种用于捕获与分析数字信号的工具。它对调试以下内容至关重要：
- 通信协议（I2C、SPI、UART）
- 时序问题
- 信号完整性问题
- 协议违规

### 逻辑分析仪设置

#### 硬件设置
1. 将逻辑分析仪连接到目标信号
2. 设置正确的电压电平与阈值
3. 配置触发条件
4. 设置采样率与缓冲区大小

#### 软件配置
```python
# 示例：使用 Python 控制 Saleae 逻辑分析仪
import saleae

# 连接到逻辑分析仪
analyzer = saleae.Saleae()

# 配置通道
analyzer.set_active_channels([0, 1, 2, 3])

# 设置触发
analyzer.set_trigger_one_channel(0, saleae.Trigger.RISING_EDGE)

# 开始捕获
analyzer.capture_start()
analyzer.capture_wait_until_finished()

# 导出数据
analyzer.export_data2('capture.csv')
```

### 常见的逻辑分析仪场景

#### 1. I2C 协议分析
```c
// 示例：调试 I2C 通信
void i2c_write_byte(uint8_t device_addr, uint8_t reg_addr, uint8_t data) {
    // 逻辑分析仪将显示：
    // 1. 起始条件（Start condition）
    // 2. 设备地址 + 写位
    // 3. 寄存器地址
    // 4. 数据字节
    // 5. 停止条件（Stop condition）
    
    i2c_start();
    i2c_write_byte(device_addr << 1);
    i2c_write_byte(reg_addr);
    i2c_write_byte(data);
    i2c_stop();
}
```

#### 2. SPI 协议分析
```c
// 示例：调试 SPI 通信
void spi_transfer(uint8_t *data, uint16_t length) {
    // 逻辑分析仪将显示：
    // 1. 片选激活
    // 2. 时钟信号
    // 3. MOSI/MISO 上的数据
    // 4. 片选取消激活
    
    cs_low();
    for (int i = 0; i < length; i++) {
        data[i] = spi_transfer_byte(data[i]);
    }
    cs_high();
}
```

### 逻辑分析仪最佳实践
1. **设置合适的采样率** - 至少为信号频率的 4 倍
2. **使用正确的触发条件** 以捕获相关事件
3. **同时监控多个信号** 以进行协议分析
4. **保存捕获数据** 以便后续分析与文档化
5. **使用协议解码器** 以进行自动分析

---

## 示波器测量

### 什么是示波器？
示波器是一种用于测量与分析模拟和数字信号的工具。它对以下内容至关重要：
- 信号完整性分析
- 时序测量
- 电源分析
- 噪声与干扰检测

### 示波器设置

#### 基本测量
```c
// 示例：测量 GPIO 信号时序
void gpio_toggle_test(void) {
    // 示波器将显示：
    // 1. 信号上升时间
    // 2. 信号下降时间
    // 3. 频率
    // 4. 占空比
    
    while (1) {
        GPIO_SetBits(GPIOA, GPIO_Pin_0);
        delay_ms(100);
        GPIO_ResetBits(GPIOA, GPIO_Pin_0);
        delay_ms(100);
    }
}
```

#### 电源分析
```c
// 示例：测量电源纹波
void measure_power_supply(void) {
    // 示波器测量：
    // 1. 直流电压电平
    // 2. 交流纹波电压
    // 3. 纹波频率
    // 4. 瞬态响应
    
    // 将示波器连接到电源输出
    // 为纹波测量设置交流耦合
    // 使用合适的电压与时间刻度
}
```

### 常见的示波器测量

#### 1. 信号时序
- **上升时间**（Rise time）：信号从最终值的 10% 到 90% 所需时间
- **下降时间**（Fall time）：信号从初始值的 90% 到 10% 所需时间
- **脉宽**（Pulse width）：脉冲在 50% 幅度处的持续时间
- **频率**（Frequency）：每秒的周期数

#### 2. 信号完整性
- **过冲**（Overshoot）：信号超过最终值
- **下冲**（Undershoot）：信号低于最终值
- **回振**（Ringback）：围绕最终值的振荡
- **噪声**（Noise）：信号中的随机变化

#### 3. 功率分析
- **直流电压**（DC voltage）：平均电压电平
- **交流纹波**（AC ripple）：围绕直流电平的变化
- **瞬态响应**（Transient response）：对负载变化的响应
- **效率**（Efficiency）：输出功率与输入功率之比

### 示波器最佳实践
1. **为信号类型与频率使用合适的探头**
2. **设置正确的触发条件** 以实现稳定显示
3. **使用合适的时间与电压刻度**
4. **测量多个参数** 以进行综合分析
5. **记录测量结果** 以便将来参考

---

## 代码覆盖率与静态分析

### 代码覆盖率

#### 什么是代码覆盖率？
代码覆盖率（Code Coverage）衡量测试期间你的代码被执行了多少。它有助于识别：
- 未测试的代码路径
- 死代码（dead code）
- 缺失的测试用例
- 代码质量问题

#### 代码覆盖率工具
```bash
# 使用 gcov 进行代码覆盖率
gcc -fprofile-arcs -ftest-coverage -o program program.c
./program
gcov program.c

# 使用 lcov 生成 HTML 报告
lcov --capture --directory . --output-file coverage.info
genhtml coverage.info --output-directory coverage_report
```

#### 示例：代码覆盖率分析
```c
// 示例：带代码覆盖率的测试
int calculate_average(int *array, int size) {
    if (array == NULL || size <= 0) {
        return -1;  // 错误条件
    }
    
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += array[i];
    }
    
    return sum / size;
}

// 实现 100% 覆盖率的测试用例
void test_calculate_average(void) {
    int array1[] = {1, 2, 3, 4, 5};
    int array2[] = {0};
    int *array3 = NULL;
    
    assert(calculate_average(array1, 5) == 3);
    assert(calculate_average(array2, 1) == 0);
    assert(calculate_average(array3, 5) == -1);
    assert(calculate_average(array1, 0) == -1);
}
```

### 静态分析

#### 什么是静态分析？
静态分析（Static Analysis）不执行代码即检查代码，以发现：
- 潜在缺陷
- 代码质量问题
- 安全漏洞
- 性能问题

#### 静态分析工具
```bash
# 使用 cppcheck 进行静态分析
cppcheck --enable=all --xml --xml-version=2 . 2> report.xml

# 使用 clang-tidy 进行额外检查
clang-tidy --checks=* source_file.c

# 使用 splint 进行额外检查
splint source_file.c
```

#### 示例：静态分析结果
```c
// 示例：带潜在问题的代码
void process_data(int *data, int size) {
    int sum = 0;
    
    // 潜在问题：未进行边界检查
    for (int i = 0; i < size; i++) {
        sum += data[i];  // 可能导致缓冲区溢出
    }
    
    // 潜在问题：未使用的变量
    int unused_var = 42;
    
    return sum;
}

// 改良版本
void process_data(int *data, int size) {
    if (data == NULL || size <= 0) {
        return -1;
    }
    
    int sum = 0;
    for (int i = 0; i < size; i++) {
        sum += data[i];
    }
    
    return sum;
}
```

---

## 嵌入式系统的单元测试

### 单元测试框架

#### 设置单元测试
```c
// 示例：简单的单元测试框架
#include <assert.h>
#include <stdio.h>

#define TEST_ASSERT(condition) \
    do { \
        if (!(condition)) { \
            printf("FAIL: %s:%d - %s\n", __FILE__, __LINE__, #condition); \
            return -1; \
        } \
    } while(0)

#define TEST_RUN(test_func) \
    do { \
        printf("Running %s...\n", #test_func); \
        if (test_func() == 0) { \
            printf("PASS: %s\n", #test_func); \
        } else { \
            printf("FAIL: %s\n", #test_func); \
        } \
    } while(0)
```

#### 示例：单元测试
```c
// 示例：测试环形缓冲区实现
int test_ring_buffer_empty(void) {
    ring_buffer_t buffer;
    ring_buffer_init(&buffer);
    
    TEST_ASSERT(ring_buffer_is_empty(&buffer) == true);
    TEST_ASSERT(ring_buffer_is_full(&buffer) == false);
    
    return 0;
}

int test_ring_buffer_full(void) {
    ring_buffer_t buffer;
    ring_buffer_init(&buffer);
    
    // 填满缓冲区
    for (int i = 0; i < RING_BUFFER_SIZE; i++) {
        TEST_ASSERT(ring_buffer_push(&buffer, i) == 0);
    }
    
    TEST_ASSERT(ring_buffer_is_full(&buffer) == true);
    TEST_ASSERT(ring_buffer_push(&buffer, 100) == -1);  // 应当失败
    
    return 0;
}

int test_ring_buffer_wrap_around(void) {
    ring_buffer_t buffer;
    ring_buffer_init(&buffer);
    
    // 填满并清空缓冲区以测试回绕
    for (int i = 0; i < RING_BUFFER_SIZE; i++) {
        ring_buffer_push(&buffer, i);
    }
    
    for (int i = 0; i < RING_BUFFER_SIZE; i++) {
        int value;
        TEST_ASSERT(ring_buffer_pop(&buffer, &value) == 0);
        TEST_ASSERT(value == i);
    }
    
    return 0;
}

// 运行所有测试
int main(void) {
    TEST_RUN(test_ring_buffer_empty);
    TEST_RUN(test_ring_buffer_full);
    TEST_RUN(test_ring_buffer_wrap_around);
    
    printf("所有测试已完成！\n");
    return 0;
}
```

### 用于硬件的模拟对象（Mock Objects）

#### 示例：模拟 UART
```c
// 用于测试的模拟 UART
typedef struct {
    uint8_t tx_buffer[256];
    uint8_t rx_buffer[256];
    int tx_index;
    int rx_index;
    int tx_count;
    int rx_count;
} mock_uart_t;

static mock_uart_t mock_uart;

void mock_uart_init(void) {
    memset(&mock_uart, 0, sizeof(mock_uart_t));
}

int mock_uart_send(uint8_t data) {
    if (mock_uart.tx_count < 256) {
        mock_uart.tx_buffer[mock_uart.tx_count++] = data;
        return 0;
    }
    return -1;
}

int mock_uart_receive(uint8_t *data) {
    if (mock_uart.rx_index < mock_uart.rx_count) {
        *data = mock_uart.rx_buffer[mock_uart.rx_index++];
        return 0;
    }
    return -1;
}

// 测试 UART 通信
int test_uart_communication(void) {
    mock_uart_init();
    
    // 测试发送数据
    TEST_ASSERT(mock_uart_send(0x55) == 0);
    TEST_ASSERT(mock_uart.tx_buffer[0] == 0x55);
    TEST_ASSERT(mock_uart.tx_count == 1);
    
    // 测试接收数据
    mock_uart.rx_buffer[0] = 0xAA;
    mock_uart.rx_count = 1;
    
    uint8_t received_data;
    TEST_ASSERT(mock_uart_receive(&received_data) == 0);
    TEST_ASSERT(received_data == 0xAA);
    
    return 0;
}
```

---

## 硬件在环测试

### HIL 测试设置

#### 什么是 HIL 测试？
硬件在环（Hardware-in-the-Loop，HIL）测试涉及用模拟硬件测试嵌入式软件。它允许：
- 在无物理硬件的情况下测试
- 自动化测试
- 可重复的测试场景
- 经济高效的测试

#### HIL 测试框架
```c
// 示例：HIL 测试框架
typedef struct {
    uint32_t gpio_state;
    uint32_t timer_value;
    uint8_t uart_rx_data;
    uint8_t uart_tx_data;
    uint32_t adc_value;
} hil_environment_t;

static hil_environment_t hil_env;

void hil_init(void) {
    memset(&hil_env, 0, sizeof(hil_environment_t));
}

// 模拟硬件函数
uint32_t hil_gpio_read(uint32_t pin) {
    return (hil_env.gpio_state >> pin) & 0x01;
}

void hil_gpio_write(uint32_t pin, uint32_t value) {
    if (value) {
        hil_env.gpio_state |= (1 << pin);
    } else {
        hil_env.gpio_state &= ~(1 << pin);
    }
}

uint32_t hil_timer_get_value(void) {
    return hil_env.timer_value;
}

void hil_timer_set_value(uint32_t value) {
    hil_env.timer_value = value;
}
```

#### 示例：HIL 测试
```c
// 示例：测试 GPIO 功能
int test_gpio_functionality(void) {
    hil_init();
    
    // 测试 GPIO 写入
    hil_gpio_write(5, 1);
    TEST_ASSERT(hil_gpio_read(5) == 1);
    
    // 测试 GPIO 读取
    hil_gpio_write(3, 1);
    TEST_ASSERT(hil_gpio_read(3) == 1);
    
    // 测试 GPIO 清除
    hil_gpio_write(5, 0);
    TEST_ASSERT(hil_gpio_read(5) == 0);
    
    return 0;
}

// 示例：测试定时器功能
int test_timer_functionality(void) {
    hil_init();
    
    // 测试定时器值
    hil_timer_set_value(1000);
    TEST_ASSERT(hil_timer_get_value() == 1000);
    
    // 测试定时器递增
    hil_timer_set_value(hil_timer_get_value() + 100);
    TEST_ASSERT(hil_timer_get_value() == 1100);
    
    return 0;
}
```

---

## 性能剖析

### 性能剖析工具

#### 使用 GProf
```bash
# 启用剖析进行编译
gcc -pg -o program program.c

# 运行程序
./program

# 生成剖析报告
gprof program gmon.out > profile.txt
```

#### 使用 Perf
```bash
# 剖析 CPU 使用
perf record ./program
perf report

# 剖析特定事件
perf record -e cache-misses ./program
perf report
```

#### 示例：性能分析
```c
// 示例：性能关键函数
void performance_critical_function(void) {
    // 剖析此函数以进行优化
    for (int i = 0; i < 1000000; i++) {
        // 昂贵的操作
        complex_calculation(i);
    }
}

// 优化版本
void optimized_function(void) {
    // 使用查找表或算法优化
    for (int i = 0; i < 1000000; i++) {
        // 优化的操作
        optimized_calculation(i);
    }
}
```

### 内存剖析

#### 使用 Valgrind
```bash
# 内存泄漏检测
valgrind --leak-check=full ./program

# 内存使用剖析
valgrind --tool=massif ./program
ms_print massif.out.* > memory_profile.txt
```

#### 示例：内存分析
```c
// 示例：内存泄漏检测
void memory_leak_example(void) {
    // 这将导致内存泄漏
    int *data = malloc(1000 * sizeof(int));
    // 缺少 free(data);
}

// 修正版本
void correct_memory_usage(void) {
    int *data = malloc(1000 * sizeof(int));
    if (data != NULL) {
        // 使用数据
        process_data(data, 1000);
        free(data);  // 正确的清理
    }
}
```

---

## 调试最佳实践

### 通用调试技巧
1. **从最简单的解释开始** - 问题往往很基础
2. **使用系统化方法** - 逐一排除可能性
3. **记录一切** - 记录你尝试过的方法
4. **使用版本控制** - 若需要可回退到已知良好状态
5. **增量测试** - 一次添加一个功能

### 调试清单
- [ ] 检查电源与电压电平
- [ ] 验证时钟配置与时序
- [ ] 确认通信协议设置
- [ ] 验证内存分配与使用
- [ ] 检查中断配置与优先级
- [ ] 验证外设初始化
- [ ] 用已知良好的参考设计测试

### 常见调试错误
1. **假设硬件正常工作** - 始终先验证硬件
2. **不查数据手册** - 规格至关重要
3. **忽略时序问题** - 嵌入式系统对时序敏感
4. **忽视内存约束** - 嵌入式系统资源有限
5. **不考虑实时约束** - 时序至关重要

---

## 资源

### 工具与软件
- [OpenOCD](http://openocd.org/) - 开源 JTAG 调试器
- [GDB](https://www.gnu.org/software/gdb/) - GNU 调试器
- [Saleae Logic](https://www.saleae.com/) - 逻辑分析仪软件
- [Sigrok](https://sigrok.org/) - 开源信号分析
- [Valgrind](http://valgrind.org/) - 内存剖析工具

### 书籍与参考文献
- 《Debugging Embedded Microprocessor Systems》by Stuart Ball
- 《The Art of Debugging》by Norman Matloff
- 《Embedded Systems Design》by Arnold Berger

### 在线资源
- [Embedded.com Debugging Section](https://www.embedded.com/category/debugging/)
- [Stack Overflow Embedded Tag](https://stackoverflow.com/questions/tagged/embedded)
- [EEVblog](https://www.eevblog.com/) - 电子工程视频
