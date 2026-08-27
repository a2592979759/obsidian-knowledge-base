---
tags: [状态机, 有限状态机, 嵌入式, 算法]
source: Data_Struct_Implementation/stateMachine
created: 2026-08-27
---

# 有限状态机（FSM）实现

## 概述
有限状态机（Finite State Machine, FSM）是一种计算模型，用于设计计算机程序和数字逻辑电路。它由有限数量的状态、状态之间的转移以及动作组成。FSM 在嵌入式系统中尤为重要，用于管理系统行为、用户界面和协议实现。

## 关键概念

### 状态
- **空闲状态（Idle State）**：未插入卡片时的初始状态
- **已插卡状态（Card Inserted State）**：插入卡片后的状态
- **已输入 PIN 状态（PIN Entered State）**：输入 PIN 之后的状态
- **已选择选项状态（Option Selected State）**：用户选择选项后的状态
- **已输入金额状态（Amount Entered State）**：输入金额后的状态

### 事件
- **插卡事件（Card Insert Event）**：插入卡片时触发
- **输入 PIN 事件（PIN Enter Event）**：输入 PIN 时触发
- **选择选项事件（Option Selection Event）**：用户选择选项时触发
- **输入金额事件（Amount Enter Event）**：输入金额时触发
- **金额分发事件（Amount Dispatch Event）**：金额被分发时触发

### 转移
FSM 根据事件在状态之间转移。每个转移由特定的事件处理函数处理。

## 实现

### FSM 示意图
![[_assets/stateMachine.png]]

### 用法
```bash
make
./stateMachine
```

### 代码实现

#### 头文件（`state_machine.h`）
```c
#ifndef STATE_MACHINE_H
#define STATE_MACHINE_H

#include <stdio.h>
#include <stdlib.h>

// 系统状态枚举
typedef enum {
    IDLE_STATE,
    CARD_INSERTED_STATE,
    PIN_ENTERED_STATE,
    OPTION_SELECTED_STATE,
    AMOUNT_ENTERED_STATE,
    MAX_STATES
} eSystemState;

// 系统事件枚举
typedef enum {
    CARD_INSERT_EVENT,
    PIN_ENTER_EVENT,
    OPTION_SELECTION_EVENT,
    AMOUNT_ENTER_EVENT,
    AMOUNT_DISPATCH_EVENT,
    MAX_EVENTS
} eSystemEvent;

// 函数原型
eSystemState amountDispatchHandler(void);
eSystemState enterAmountHandler(void);
eSystemState optionSelectionHandler(void);
eSystemState enterPinHandler(void);
eSystemState insertCardHandler(void);

// 状态机函数
void stateMachineInit(void);
eSystemState processEvent(eSystemState currentState, eSystemEvent event);
const char* getStateName(eSystemState state);
const char* getEventName(eSystemEvent event);

#endif // STATE_MACHINE_H
```

#### 实现文件（`state_machine.c`）
```c
#include "state_machine.h"

// 事件处理函数
eSystemState amountDispatchHandler(void) {
    printf("Amount dispatched. Returning to idle state.\n");
    return IDLE_STATE;
}

eSystemState enterAmountHandler(void) {
    printf("Amount entered successfully.\n");
    return AMOUNT_ENTERED_STATE;
}

eSystemState optionSelectionHandler(void) {
    printf("Option selected successfully.\n");
    return OPTION_SELECTED_STATE;
}

eSystemState enterPinHandler(void) {
    printf("PIN entered successfully.\n");
    return PIN_ENTERED_STATE;
}

eSystemState insertCardHandler(void) {
    printf("Card inserted successfully.\n");
    return CARD_INSERTED_STATE;
}

// 状态机处理函数
eSystemState processEvent(eSystemState currentState, eSystemEvent event) {
    switch(currentState) {
        case IDLE_STATE:
            if(event == CARD_INSERT_EVENT) {
                return insertCardHandler();
            }
            break;
            
        case CARD_INSERTED_STATE:
            if(event == PIN_ENTER_EVENT) {
                return enterPinHandler();
            }
            break;
            
        case PIN_ENTERED_STATE:
            if(event == OPTION_SELECTION_EVENT) {
                return optionSelectionHandler();
            }
            break;
            
        case OPTION_SELECTED_STATE:
            if(event == AMOUNT_ENTER_EVENT) {
                return enterAmountHandler();
            }
            break;
            
        case AMOUNT_ENTERED_STATE:
            if(event == AMOUNT_DISPATCH_EVENT) {
                return amountDispatchHandler();
            }
            break;
            
        default:
            printf("Invalid state: %d\n", currentState);
            break;
    }
    
    printf("Invalid event %s for state %s\n", getEventName(event), getStateName(currentState));
    return currentState;
}

// 工具函数
const char* getStateName(eSystemState state) {
    static const char* stateNames[] = {
        "IDLE_STATE",
        "CARD_INSERTED_STATE", 
        "PIN_ENTERED_STATE",
        "OPTION_SELECTED_STATE",
        "AMOUNT_ENTERED_STATE"
    };
    return (state < MAX_STATES) ? stateNames[state] : "UNKNOWN_STATE";
}

const char* getEventName(eSystemEvent event) {
    static const char* eventNames[] = {
        "CARD_INSERT_EVENT",
        "PIN_ENTER_EVENT",
        "OPTION_SELECTION_EVENT", 
        "AMOUNT_ENTER_EVENT",
        "AMOUNT_DISPATCH_EVENT"
    };
    return (event < MAX_EVENTS) ? eventNames[event] : "UNKNOWN_EVENT";
}
```

#### 主应用（`main.c`）
```c
#include "state_machine.h"

int main(int argc, char *argv[]) {
    eSystemState currentState = IDLE_STATE;
    eSystemEvent newEvent;
    char input;
    
    printf("=== ATM State Machine Demo ===\n");
    printf("Available events:\n");
    printf("0 = Card Insert Event\n");
    printf("1 = PIN Enter Event\n");
    printf("2 = Option Selection Event\n");
    printf("3 = Amount Enter Event\n");
    printf("4 = Amount Dispatch Event\n");
    printf("q = Quit\n\n");
    
    while(1) {
        printf("Current State: %s\n", getStateName(currentState));
        printf("Enter event (0-4, q to quit): ");
        
        input = getchar();
        getchar(); // 吸收换行
        
        if(input == 'q' || input == 'Q') {
            printf("Exiting...\n");
            break;
        }
        
        newEvent = (eSystemEvent)atoi(&input);
        
        if(newEvent >= 0 && newEvent < MAX_EVENTS) {
            eSystemState nextState = processEvent(currentState, newEvent);
            if(nextState != currentState) {
                printf("State transition: %s -> %s\n", 
                       getStateName(currentState), getStateName(nextState));
                currentState = nextState;
            }
        } else {
            printf("Invalid event. Please enter 0-4 or q to quit.\n");
        }
        
        printf("\n");
    }
    
    return 0;
}
```

#### Makefile
```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c99
TARGET = stateMachine
SOURCES = main.c state_machine.c
HEADERS = state_machine.h

$(TARGET): $(SOURCES) $(HEADERS)
	$(CC) $(CFLAGS) -o $(TARGET) $(SOURCES)

clean:
	rm -f $(TARGET)

.PHONY: clean
```

#### 简化的 FSM 实现（`stateMachine.c`，源码目录中的实际版本）

```c
#include <stdio.h>
#include <stdlib.h>
// ATM 机的不同状态
typedef enum
{
    Idle_State,
    Card_Inserted_State,
    Pin_Eentered_State,
    Option_Selected_State,
    Amount_Entered_State,
} eSystemState;
// 不同类型的事件
typedef enum
{
    Card_Insert_Event,
    Pin_Enter_Event,
    Option_Selection_Event,
    Amount_Enter_Event,
    Amount_Dispatch_Event
} eSystemEvent;
// 事件处理函数原型
eSystemState AmountDispatchHandler(void)
{
    return Idle_State;
}
eSystemState EnterAmountHandler(void)
{
    return Amount_Entered_State;
}
eSystemState OptionSelectionHandler(void)
{
    return Option_Selected_State;
}
eSystemState EnterPinHandler(void)
{
    return Pin_Eentered_State;
}
eSystemState InsertCardHandler(void)
{
    return Card_Inserted_State;
}
int main(int argc, char *argv[])
{
    eSystemState eNextState = Idle_State;
    eSystemEvent eNewEvent;
    char input;
    while(1)
    {
        printf("curState: %d\n", eNextState);
        // 读取系统事件
        printf("please enter event\n0 = Card_Insert_Event\n1 = Pin_Enter_Event\n2 = Option_Selection_Event\n3 = Amount_Enter_Event\n4 = Amount_Dispatch_Event\n");
        input = getchar( );
        eSystemEvent eNewEvent = atoi(&input);
        switch(eNextState)
        {
        case Idle_State:
        {
            if(Card_Insert_Event == eNewEvent)
            {
                eNextState = InsertCardHandler();
            }
        }
        break;
        case Card_Inserted_State:
        {
            if(Pin_Enter_Event == eNewEvent)
            {
                eNextState = EnterPinHandler();
            }
        }
        break;
        case Pin_Eentered_State:
        {
            if(Option_Selection_Event == eNewEvent)
            {
                eNextState = OptionSelectionHandler();
            }
        }
        break;
        case Option_Selected_State:
        {
            if(Amount_Enter_Event == eNewEvent)
            {
                eNextState = EnterAmountHandler();
            }
        }
        break;
        case Amount_Entered_State:
        {
            if(Amount_Dispatch_Event == eNewEvent)
            {
                eNextState = AmountDispatchHandler();
            }
        }
        break;
        default:
            printf("invalid input\n");
            break;
        }
    }
    return 0;
}
```

## 常见面试问题

1. **什么是有限状态机？**
   - 一种包含有限数量和转移的计算模型
   - 用于建模嵌入式系统的行为
   - 由状态、事件和转移组成

2. **使用 FSM 有什么优点？**
   - 行为清晰、可预测
   - 易于调试和维护
   - 模块化设计
   - 适合实时系统

3. **如何在 C 中实现 FSM？**
   - 用枚举表示状态和事件
   - 实现状态转移表或 switch 语句
   - 创建事件处理函数
   - 用主循环处理事件

4. **FSM 有哪些不同类型？**
   - **Moore 机**：输出只取决于当前状态
   - **Mealy 机**：输出取决于当前状态和输入

5. **如何处理非法状态转移？**
   - 在转移逻辑中加入错误处理
   - 记录非法转移
   - 返回安全状态或错误状态

## 高级话题

### 状态机模式
1. **层次状态机**：状态可以包含子状态
2. **嵌套状态机**：状态本身可以是状态机
3. **并行状态机**：多个状态机并发运行

### 实现注意事项
- **内存效率**：大型状态机使用查表法
- **性能**：为实时系统优化转移逻辑
- **可维护性**：使用清晰的命名规范和文档
- **测试**：为所有转移创建全面测试用例

## 资源
- [State Machine Using C - AticleWorld](https://aticleworld.com/state-machine-using-c/)
- [Finite State Machines in Embedded Systems](https://embeddedartistry.com/blog/2018/07/12/an-introduction-to-finite-state-machines/)
- [State Machine Design Patterns](https://www.state-machine.com/doc/concepts.html)

## 相关文档
- [[taskScheduler]] —— 状态机思想也常用于 RTOS/任务调度
- [[timerWheel]] —— 状态机与定时器常配合
