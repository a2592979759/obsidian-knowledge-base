---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Company/tesla.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的嵌入式题库、带 AI 反馈的编码练习以及系统设计指南进行准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)**

---

### 公司：Tesla（Company: Tesla）
### 职位：固件（Position: Firmware）

你有 120 分钟完成测试。总分为 100 分。

所有解决方案应在 Coderpad.io 中无错误、无警告地编译。

罚则（Penalties）：
 -1 / 超时每分钟
 -3 / 出现 1 个或多个编译错误
 -2 / 出现 1 个或多个编译警告

请勿借用外部帮助或分享本测试的内容。

底部提供了一个 main() 函数供你使用。

```c
    #include <stdio.h>
    #include <stdint.h>
    #include <stdlib.h>
    #include <stdbool.h>
    

    /* TESLA MOTORS FIRMWARE TEST
    * 你有 120 分钟完成测试。总分为 100 分。
    *
    * 所有解决方案应在 Coderpad.io 中无错误、无警告地编译。
    *
    * 罚则：
    * -1 / 超时每分钟
    * -3 / 出现 1 个或多个编译错误
    * -2 / 出现 1 个或多个编译警告
    *
    * 请勿借用外部帮助或分享本测试的内容。
    *
    * 底部提供了一个 main() 函数供你使用。
    */
    
    ////////////////////////////////////////////////////////////////////////////////
    // 1) 宏（Macro）(10 分)
    //    创建一个宏（名为 C_TO_F），将摄氏度（Celsius）转换为华氏度（Fahrenheit）。
    //    宏应对整数或浮点类型都能工作。
    //    注意：degF = degC * (9/5) + 32

    #define VAR1 (9.0f)
    #define VAR1 (5.0f)
    #define C_TO_F(degC) \
            (degC * (VAR1 / VAR2) +32)


    
    ////////////////////////////////////////////////////////////////////////////////
    // 2) 位操作（Bit Manipulation）(5 分)
    //    编写一个函数，将 b 所指向的数据值的最高有效位（most significant）与
    //    最低有效位（least significant）翻转（0 -> 1 或 1 -> 0）。
    
    void flip_hi_lo(uint8_t* b)
    {
        uint8_t op = 0x81   //1000 0001
        *b ^= op;
    }
    
    ////////////////////////////////////////////////////////////////////////////////
    // 3) 调试（Debugging）(5 分)
    //    函数 computeSquareADC() 一直未能持续产生正确的输出。
    //    请描述该函数的所有问题。
    // 答案：TODO

    //1. 返回类型是 uint8_t，而 retval 类型被定义为 uint16_t，结果将不准确。
    //2. volatile 类型使 ADC_RESULT 在做乘法时可能改变值。
    
    ////////////////////////////////////////////////////////////////////////////////
    // 4) 内存转储（Memory dump）(10 分)
    //    以下内存转储是在调试某个问题时捕获的。
    //
    // 内存转储（Memory Dump）：
    // 地址（Address）:  字节（Byte）:
    // 0x1000    0xA0
    // 0x1001    0x0A
    // 0x1002    0xBA
    // 0x1003    0x48
    // 0x1004    0x2C
    // 0x1005    0xB7
    // 0x1006    0x3B
    // 0x1007    0x82
    // 0x1008    0x9C
    // 0x1009    0xE5
    // 0x100A    0x17
    // 0x100B    0x40
    // 0x100C    0xEF
    // 0x100D    0x47
    // 0x100E    0x0F
    // 0x100F    0x98
    // 0x1010    0x6F
    // 0x1011    0xD5
    // 0x1012    0x70
    // 0x1013    0x9E
    // 0x1014    0x94
    // 0x1015    0x99
    // 0x1016    0x4A
    // 0x1017    0xBA
    // 0x1018    0xCA
    // 0x1019    0xB2
    // 0x101A    0x32
    // 0x101B    0xE6
    // 0x101C    0x8E
    // 0x101D    0xB9
    // 0x101E    0xC5
    // 0x101F    0x2E
    // 0x1020    0xC3
    //
    // 系统为 32 位，小端（little-endian）。
    // 一个名为 myPacket 的变量类型为 packet_S（typedef 见下）。
    // （默认编译器选项；未打包，自然对齐。）-->这意味着没有填充（padding）（-Colin）
    // myPacket 的地址为 0x1010。
    //
    typedef struct
    {
        uint8_t count;
        uint16_t data[2]; -->元素之间不应有填充，因此为 2字节*2（-Colin）
        uint32_t timestamp;
    } packet_S;
    
    // a) myPacket 的每个成员的值分别是多少？
    //答案：
    count = 0x6F
    data[0] = 0x9994
    data[1] = 0xBA4A 
    timestamp = 0xE632B2CA
    
    // b) 如果系统是大端（big-endian），myPacket 的每个成员的值分别是多少？
    //答案：
    count = 0x6F
    data[0] = 0x9499
    data[1] = 0x4ABA
    timestamp = 0xCAB232E6

    ////////////////////////////////////////////////////////////////////////////////
    // 5) 状态机（State Machine）(20 分)
    //
    //    补全下面的函数，以实现下图中所示的状态机，用于一台电子口香糖售卖机。
    //     * 状态机的初始状态应为 IDLE
    //     * 函数应输出状态机的当前状态
    //     * 意外或无效的输入不应引起状态迁移
    //     * GENERIC_FAULT 可在任何状态被接收，并应将机器置为 FAULT 状态
    
    typedef enum
    {
        IDLE,
        READY,
        VENDING,
        FAULT
    } state_E;
    
    typedef enum
    {
        COIN,
        COIN_RETURN,
        BUTTON,
        VEND_COMPLETE,
        GENERIC_FAULT
    } input_E;


    state_E currState = IDLE;// 将当前状态默认设为 IDLE
    state_E stateMachine(input_E input)
    {
        state_E retVal = currState;
        switch(input)
        {	
            case GENERIC_FAULT:
                    retVal = FAULT;;
                    currState = retVal;
                    break;
            case COIN:
                if (currState = IDLE) {
                    retVal = READY;
                    currState = retVal;
                }
                break;
                case COIN_RETURN:
                if (currState = READY) {
                    retVal = IDLE;
                    currState = retVal;
                }
                case BUTTON:
                    if (currState = READY) {
                        retVal = VENDING;
                        currState = retVal;
                    }
            break;
                case VEND_COMPLETE:
                    if (currState == VENDING) {
                        retVal = IDLE;
                        currState = retVal;           
                    }
                    break;
        default: 
                break;
        }
        
        return retVal;

    }
    
    
    ////////////////////////////////////////////////////////////////////////////////
    // 6) 单元测试（Unit Testing）(10 分)
    //    为 validatePointerAndData 编写一个单元测试，覆盖所有代码路径
    //    与分支条件。
    
    // @param dataPtr - 要使用的数据 int32_t 指针
    //
    // @return 如果指针非 NULL、数据值为正、非零且不等于哨兵值 0x7FFFFFFF，则返回 TRUE，否则返回 FALSE
    //
    bool validatePointerAndData(int32_t* dataPtr)
    {
        bool status = false;
        if ((dataPtr != NULL) &&
            (*dataPtr > 0)    &&
            (*dataPtr != 0x7FFFFFFF))
        {
            status = true;
        }
        return status;
    }
    
    //
    // @return 如果所有测试通过则返回 TRUE，否则返回 FALSE
    //
    bool test_validatePointerAndData(void)
    { 
        //答案：TODO
        bool result = false;

        result  = (validatePointerAndData(NULL) == false);
        If (!result) return result;

        Int32_t test_val = -1;
        result  = (validatePointerAndData(&test_val ) == false);
        If (!result) return result;

        test_val = 1;
        result  = (validatePointerAndData(&test_val ) == true);
        If (!result) return result;

        test_val = 0x7FFFFFFF;
        result  = (validatePointerAndData(&test_val ) == false);
        If (!result) return result;
    return result;

    }
    
    
    ////////////////////////////////////////////////////////////////////////////////
    // 7) 低通滤波器（Low Pass Filter）(10 分)
    //    实现一个函数，它以 10hz 频率（每 100 ms）被调用，并返回
    //    一个指数加权平均（exponentially weighted average）。最新样本权重为 1/10，
    //    上一个滤波值权重为 9/10。如果是该函数首次运行，应将滤波器初始化为
    //    收到的第一个样本值。
    
    float value;
    int inited = 0;
    float lowPassSamples_10hz(float sample)
    {
        // 答案：TODO
        Static Float lpf_val;
        lpf_val = sample/10.0 + ((lpf_val) - (lpf_val)/10.0);
        mdelay(100);    
        Return lpf_val;
    }
    
    ////////////////////////////////////////////////////////////////////////////////
    // 8a) 缓冲区（Buffer）(20 分：8a + 8b)
    //     创建一个函数，将一个字符推入 FIFO。该 FIFO 应实现为长度为 20 的环形缓冲区
    //     （circular buffer）。FIFO 将用于缓存来自数据流的最新的数据，因此，如果缓冲区
    //     已满，则丢弃最旧的值。
    //答案：

    #define MAX_CIRCULAR_BUFFER_SIZE 20
    uint8_t current_num_elements = 0;
    uint8_t readInd = 0;
    uint8_t writeInd = 0;
    uint8_t cirular_buffer[MAX_CIRCULAR_BUFFER_SIZE];
    void EnQueue(int input_data)
    {
        if (readInd == writeInd && 
            current_num_elements ==  MAX_CIRCULAR_BUFFER_SIZE) {
                printf("%reaching max elements, discard the oldest item\n");
                readInd++;
                readInd = readInd % MAX_CIRCULAR_BUFFER_SIZE;
        }

        cirular_buffer[writeInd] = input_data;

        writeInd++;

        if (current_num_elements != MAX_CIRCULAR_BUFFER_SIZE) {
            current_num_elements++;
        }

        if (writeInd == MAX_CIRCULAR_BUFFER_SIZE){
            writeInd = 0;
        }
}

    
    ////////////////////////////////////////////////////////////////////////////////
    // 8b) 创建一个函数来打印并清空数据缓冲区。
    //     数据应按从旧到新的顺序打印，仅包含有效元素。


    // 答案：
    uint8_t current_num_elements = 0;
    void printAndEmptyBuffer(void)
    {
        if (!current_num_elements) {
            return;
        }
        while (current_num_elements) {
            if(readInd == MAX_CIRCULAR_BUFFER_SIZE) {
                readInd = 0;
            }
            printf("%d\n", circular_buffer[readInd]);
            readInd++;
            current_num_elements--;
        }
    }

    
    ////////////////////////////////////////////////////////////////////////////////
    // 8c) 中断（Interrupts）(10 分)
    //     函数 bufferPush_ISR() 将在有新的需要缓冲的数据可用时，从中断服务例程
    //     被调用。
    //     函数 printAndEmptyBuffer() 将从一个周期性任务（periodic task）中被调用。
    //     函数 disableInterrupts() 和 enableInterrupts() 分别用于禁用和使能中断。
    //
    //     在你对 bufferPush_ISR() 和 printAndEmptyBuffer() 的实现中，
    //     判断是否有必要禁用/使能中断。
    //     如有必要，请在需要调用的位置添加注释。如不需要，
    //     简要说明原因。
    //
    // 答案：在 bufferPush_ISR() 中，我们应在服务期间调用 enableInterrupts()，而在退出时
    // 应调用 disableInterrupts()，这样做可以防止当前 bufferPush_ISR() 尚未完成而又
    // 有新的中断进来的情况。同样在 printAndEmptyBuffer() 中我们也应这样做，因为在
    // bufferPush_ISR() 中我们有在溢出时覆盖的机制，这意味着 readInd 会被更新。由于 readInd
    // 会被 ISR 和 task 共同访问，我们需要确保互斥（mutual exclusion），以使
    // 环形缓冲区能够正常工作。
    =======================================================================
```

## 相关页面
- [[tesla]] —— Tesla
- [[amazon]] —— Amazon
- [[lyft]] —— Lyft
- [[zoox]] —— Zoox
- [[qualcomm]] —— Qualcomm

返回索引 [[00-索引]]
