# STM32面试考点

# 硬件基础

## 1.最小系统有什么？

单片机最小系统是指能够将单片机芯片运行所必需的最少的硬件电路集成在一起的系统。

它是一种基本的单片机应用系统，通常由**主芯片，时钟电路，复位电路，电源电路，BOOT启动电路，程序下载电路，扩展接口**组成，为单片机提供时钟信号、复位信号以及外设接口等必要功能。

## 2.晶振作用

## 3.单片机构造

<img src="./STM32面试考点.assets/image-20250301163455802.png" alt="image-20250301163455802" style="zoom:30%;" />

## 4.stm32引脚分布



## 定时器面试常见问题

### 1.STM32 定时器的种类有哪些？

### 2.如何配置STM32 定时器的时钟源和计数模式？

#### 一、时钟源配置方法

##### 1. 内部时钟模式（常用）

- 配置步骤： ① 使能APB总线时钟（如TIM2属于APB1）② 调用`TIM_InternalClockConfig()`选择内部时钟源

    ③ 时基单元的分频系数通过PSC寄存器设置

    示例代码：

    ```c
    CRCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);
    TIM_InternalClockConfig(TIM2); // 核心配置语句
    ```

##### 2. 外部时钟模式1

- **应用场景**：通过GPIO引脚接收外部脉冲计数

- 配置步骤：① 配置GPIO为浮空输入模式 ② 选择触发通道（TI1/TI2）和边沿检测方式 ③ 调用`TIM_TIxExternalClockConfig()`函数

     示例代码：

    ```c
    CGPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 
    TIM_TIxExternalClockConfig(TIM2, TIM_TS_TI1FP1, TIM_ICPolarity_Rising, 15);
    ```

##### 3. 外部时钟模式2

- **特点**：使用专用ETR引脚接收时钟

- **关键函数**：`TIM_ETRClockMode2Config()`

- 参数配置：

    - 分频因子（如TIM_ExtTRGPSC_OFF表示不分频）
    - 极性选择（上升沿/下降沿触发）

    ```c
    TIM_ETRClockMode2Config(TIM2, TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0x0F);
    ```

#### 二、计数模式配置

##### 1. 模式选择依据

| 模式类型 | 应用场景           | 寄存器配置值                             |
| ---- | -------------- | ---------------------------------- |
| 向上计数 | PWM生成、定时中断     | TIM_CounterMode_Up                 |
| 向下计数 | 递减计时应用         | TIM_CounterMode_Down               |
| 中央对齐 | 电机控制等需要对称波形的场景 | TIM_CounterMode_CenterAligned 1 61 |

##### 2. 关键配置项

在时基初始化结构体中设置：

```
CTIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up; // 示例设置为向上计数
TIM_TimeBaseInitStructure.TIM_Period = 10000-1;  // ARR值
TIM_TimeBaseInitStructure.TIM_Prescaler = 7200-1; // PSC分频值
```

##### 3. 特殊功能寄存器

- ARPE位：控制ARR寄存器的预装载功能（0=立即生效，1=下个周期生效）
- URS位：选择更新事件源，避免误触发中断

### 3.STM32 定时器如何产生中断？

1. 时钟使能、2. 配置时基单元、3. 中断使能、4. NVIC配置、5. 启动定时器

### 4.如何利用STM32 定时器测量信号的频率和脉冲宽度？

#### 一、脉冲宽度测量方法

##### 1. **边沿触发法**

**实现原理**：通过交替捕获上升沿和下降沿的计数器值（CNT），计算时间差值。 **配置步骤**：

1. 配置GPIO为浮空输入模式，启用定时器时钟（如TIM2）
2. 初始化输入捕获通道，设置首次触发为上升沿

```c
CTIM_ICInitTypeDef ic;
ic.TIM_Channel = TIM_Channel_1;
ic.TIM_ICPolarity = TIM_ICPolarity_Rising; //初始上升沿
ic.TIM_ICSelection = TIM_ICSelection_DirectTI;
TIM_ICInit(TIM2, &ic);
```

3. 在捕获中断中切换触发沿并记录CNT值

```c
Cvoid TIMx_IRQHandler() {
    if(TIM_GetITStatus(TIM2, TIM_IT_CC1)) {
        static uint16_t rise_val, fall_val;
        if(当前为上升沿) {
            rise_val = TIM_GetCapture1(TIM2);
            TIM_OC1PolarityConfig(TIM2, TIM_ICPolarity_Falling); //切换为下降沿
        } else {
            fall_val = TIM_GetCapture1(TIM2);
            pulse_width = (fall_val > rise_val) ? (fall_val - rise_val) : 
                        (0xFFFF - rise_val + fall_val); //处理计数器溢出
        }
        TIM_ClearITPendingBit(TIM2, TIM_IT_CC1);
    }
}
```

2. ##### **PWM输入模式**

**硬件特性**：利用定时器的双捕获通道自动映射功能（如TI1同时映射到IC1和IC2） 

**配置要点**：

- 通道1捕获上升沿，通道2捕获下降沿
- 通过`TIM_PWMIConfig()`函数自动配置双通道

```c
CTIM_ICInitTypeDef pwmIC;
pwmIC.TIM_ICSelection = TIM_ICSelection_IndirectTI; //通道1映射到TI2
TIM_PWMIConfig(TIM2, &pwmIC);
TIM_SelectInputTrigger(TIM2, TIM_TS_TI1FP1); //触发源选择
```

#### 二、频率测量方法

##### 1. **周期测量法**

**实现原理**：测量两个连续上升沿之间的时间间隔

**关键代码**：

```c
Cuint32_t prev_cnt = 0, curr_cnt = 0;
void TIMx_CC_IRQHandler() {
    curr_cnt = TIM_GetCapture1(TIM2);
    frequency = SystemCoreClock / (curr_cnt - prev_cnt); //需考虑预分频系数
    prev_cnt = curr_cnt;
}
```

##### 2. **外部时钟模式**

**配置要点**：

- 将外部信号作为定时器时钟源
- 设置从模式为复位模式，实现自动计数器清零

```c
CTIM_ETRClockMode2Config(TIM2, TIM_ExtTRGPSC_OFF, TIM_ExtTRGPolarity_NonInverted, 0);
TIM_SelectSlaveMode(TIM2, TIM_SlaveMode_Reset); //外部信号触发复位
```

### 5.STM32 定时器的PWM 输出功能如何应用？



# ——————————————

# STM32铁头羊速成



# 1.1_基本信息

## STM32的型号及含义

1. 通过雷雕文字获取芯片型号，如STM32F103C8T6。 

2. 型号分解：ST（品牌）、M32（产品类型）、F103（产品系列）、C8T6（规格型号）。 

3. 规格型号包括引脚数量、Flash容量、封装类型和工作温度范围。

<img src="./STM32面试考点.assets/image-20250306103255167.png" alt="image-20250306103255167" style="zoom:40%;" />

<img src="./STM32面试考点.assets/image-20250306103324412.png" alt="image-20250306103324412" style="zoom: 23%;" />

封装就表示，焊盘的一个形状

<img src="./STM32面试考点.assets/image-20250306104441709.png" alt="image-20250306104441709" style="zoom:33%;" />

<img src="./STM32面试考点.assets/image-20250306104939834.png" alt="image-20250306104939834" style="zoom:25%;" />

# 1.2.STM32的引脚分布

<img src="./STM32面试考点.assets/image-20250307164728399.png" alt="image-20250307164728399" style="zoom:65%;" />

## 引脚分布图的查找

Pins and Pin Description。每一张图就代表单片机的一种封装类型

<img src="./STM32面试考点.assets/image-20250307165749926.png" alt="image-20250307165749926" style="zoom: 33%;" />

<img src="./STM32面试考点.assets/image-20250307165856230.png" alt="image-20250307165856230" style="zoom: 25%;" />

<img src="./STM32面试考点.assets/image-20250307165929204.png" alt="image-20250307165929204" style="zoom:25%;" />

## 引脚编号规则

1. 芯片引脚编号规则：找到圆点，逆时针递增编号。 

<img src="./STM32面试考点.assets/image-20250307171156551.png" alt="image-20250307171156551" style="zoom:25%;" />

2. 单片机引脚分布图上的圆点及其编号方式与通用规则一致。

3. 具体到STM32F103C8T6单片机，引脚编号从1到48。

## 特殊功能引脚

1. 特殊功能引脚包括VDD、VSS、NRST、WDBET和BOOT0。

2. VDD和VSS用于供电，VDD接3.3V，VSS接地。对于MOS管来说，它的电流一般是从d流向s，所以我们一般情况下给这个漏极接正电压，给源极接负电压

<img src="./STM32面试考点.assets/image-20250307171413229.png" alt="image-20250307171413229" style="zoom:33%;" />

3. NRST为复位引脚，连接复位按钮。

4. WDBET接备用电池，确保断电后部分功能仍能运行。
5. BOOT0引脚用于选择芯片的启动模式。

这个跳帽移到左边的话，就是boot 0接低电压；移到右边的话，就是boot 0接高电压

<img src="./STM32面试考点.assets/image-20250307171739932.png" alt="image-20250307171739932" style="zoom:50%;" />

## 普通引脚的命名规则

1.普通引脚分组：GP_A、GP_B、GP_C、GP_D等。 

2.每组最多16个引脚，以字母和数字命名。 

3.同一组IO引脚的编号不连续，会间断。

<img src="./STM32面试考点.assets/image-20250307173516389.png" alt="image-20250307173516389" style="zoom: 33%;" />

# ——————————————

# 2.1_[GPIO]4种输出模式

<img src="./STM32面试考点.assets/image-20250307204025786.png" alt="image-20250307204025786" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250307204048818.png" alt="image-20250307204048818" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250307204108223.png" alt="image-20250307204108223" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250307204135701.png" alt="image-20250307204135701" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250307204153305.png" alt="image-20250307204153305" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250307204208196.png" alt="image-20250307204208196" style="zoom:60%;" />

## GPIO模块

片上外设

<img src="./STM32面试考点.assets/image-20250307204429763.png" alt="image-20250307204429763" style="zoom:43%;" />

四个GPIO模块：GPIOA、GPIOB、GPIOC和GPIOD

每个GPIO模块控制相应的引脚组，类似于人体的手控制手指。

## GPIO工作模式

GPIO的八种工作模式，分为输出和输入模式

- 输出模式包括：通用输出推挽、通用输出开漏、复用输出推挽、复用输出开漏。 
- 输入模式包括：输入上拉、输入下拉、输入浮空、模拟模式

<img src="./STM32面试考点.assets/image-20250307212637982.png" alt="image-20250307212637982" style="zoom: 33%;" />

输出1/0

<img src="./STM32面试考点.assets/image-20250307213204257.png" alt="image-20250307213204257" style="zoom: 43%;" />

输入1/0，读寄存器的值

<img src="./STM32面试考点.assets/image-20250307213426066.png" alt="image-20250307213426066" style="zoom:50%;" />

## 输出模式

<img src="./STM32面试考点.assets/image-20250307213704024.png" alt="image-20250307213704024" style="zoom: 33%;" />

简化模式

<img src="./STM32面试考点.assets/image-20250307214312119.png" alt="image-20250307214312119" style="zoom:43%;" />

## 推挽模式

通过==PMOS和NMOS管交替工作==，输出高电压或低电压，不能同时导通，不能同时推和挽

<img src="./STM32面试考点.assets/image-20250307214551704.png" alt="image-20250307214551704" style="zoom:40%;" />

## 开漏模式

PMOS管始终断开，==NMOS管控制输出低电压或高阻抗==

开漏模式的特性：写0输出低电压，写1输出高阻抗

<img src="./STM32面试考点.assets/image-20250307220504249.png" alt="image-20250307220504249" style="zoom:40%;" />

## 通用和复用模式

- 通用模式：通过CPU直接控制IO引脚的输出
- 复用模式：通过其他外设控制IO引脚的输出

<img src="./STM32面试考点.assets/image-20250307221856234.png" alt="image-20250307221856234" style="zoom: 33%;" />

<img src="./STM32面试考点.assets/image-20250307221950857.png" alt="image-20250307221950857" style="zoom:33%;" />

<img src="./STM32面试考点.assets/image-20250307222040106.png" alt="image-20250307222040106" style="zoom:33%;" />

# 2.2_[GPIO]IO的最大输出速度

<img src="./STM32面试考点.assets/image-20250307222213849.png" alt="image-20250307222213849" style="zoom:50%;" />

## IO最大输出速度的定义

1.IO最大输出速度定义：IO引脚每秒钟能够切换状态的次数。 

2.测量方法：通过测量引脚从输出低电压到高电压，再从高电压到低电压的切换速度来确定。 

3.影响因素：引脚的输出模式、电路设计、信号完整性等。

<img src="./STM32面试考点.assets/image-20250308103217280.png" alt="image-20250308103217280" style="zoom:33%;" />

## 上升时间、下降时间和保持时间

1.上升时间：电压从低到高所需的时间。 

2.下降时间：电压从高到低所需的时间。 

.保持时间：电压保持在有效电平的时间。

<img src="./STM32面试考点.assets/image-20250308103301417.png" alt="image-20250308103301417" style="zoom:33%;" />

## IO最大输出速度的限制因素

1.限制因素：主要是上升时间和下降时间的长短。 

2.原理：上升时间和下降时间越长，引脚切换状态的速度越慢。 

3.优化方法：通过减小上升时间和下降时间，可以提高引脚的切换速度。

<img src="./STM32面试考点.assets/image-20250308105129129.png" alt="image-20250308105129129" style="zoom: 33%;" />

## IO最大输出速度的选择原则

<img src="./STM32面试考点.assets/image-20250308105247589.png" alt="image-20250308105247589" style="zoom:33%;" />

选择原则：选择满足要求的最小值。过高的最大输出速度会引入emi问题，也就是会对其他的电子元器件产生电磁干扰

- 低速：适用于对速度要求不高的场景，如驱动LED。 

- 中速：适用于一般速度要求的场景，如SPI通信。 

- 高速：适用于高速通信场景，如USB接口。

<img src="./STM32面试考点.assets/image-20250308105646447.png" alt="image-20250308105646447" style="zoom: 33%;" />

# 2.3_[GPIO]LED闪灯实验

<img src="./STM32面试考点.assets/image-20250308110726179.png" alt="image-20250308110726179" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250308110737562.png" alt="image-20250308110737562" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250308110903339.png" alt="image-20250308110903339" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250308110656991.png" alt="image-20250308110656991" style="zoom: 50%;" />

## LED工作原理

<img src="./STM32面试考点.assets/image-20250308111636791.png" alt="image-20250308111636791" style="zoom:43%;" />



1.LED符号：发光二极管，具有阳极和阴极。 

2.工作电压：阳极接正电压，阴极接负电压。 

3.电流范围：2到10毫安。 

4.限流电阻：限制电流在2到10毫安之间。

## LED接线方式

1.推挽接法：开关放在LED阳极，类似推挽模式。 

<img src="./STM32面试考点.assets/image-20250308111720903.png" alt="image-20250308111720903" style="zoom:43%;" />

<img src="./STM32面试考点.assets/image-20250308111847333.png" alt="image-20250308111847333" style="zoom:43%;" />

2.开漏接法：开关放在LED阴极，类似开漏模式。 

<img src="./STM32面试考点.assets/image-20250308112132040.png" alt="image-20250308112132040" style="zoom:43%;" />

<img src="./STM32面试考点.assets/image-20250308112249289.png" alt="image-20250308112249289" style="zoom:43%;" />

3.电路图对比：推挽接法和开漏接法的电路图对比。

<img src="./STM32面试考点.assets/image-20250308112344296.png" alt="image-20250308112344296" style="zoom: 33%;" />

## 最小系统板电路图分析

该单片机是开漏接法，设置这个l引脚工作模式为开漏

<img src="./STM32面试考点.assets/image-20250308112706495.png" alt="image-20250308112706495" style="zoom: 33%;" />

## 编程步骤与初始化

我们给这只手供血之后，这只手才能开始工作，也就相当于我们这个GPIOC开启时钟，才能开始工作。

```c
//开启GPIOx的时钟
RCC_APB2PriphClockCmd (RCC_APB2Periph_GPIOx, ENABLE);
```

### GPIO写操作编程接口使用方法

<img src="./STM32面试考点.assets/image-20250308163121782.png" alt="image-20250308163121782" style="zoom:33%;" />

函数参数：端口号、引脚编号、比特值

```c
void GPIO_Init(GPIOTypeDef *GPIOx,	           //端口号，取ABCD...
	      GPIO_InitTypeDef *GPIO_InistStruct)；//初始化的参数
```

初始化一枚IO引脚

```c
struct GPIO_InitTypeDef{
	GPIO_Pin;	//引脚编-GPI0_Pin_0..GPI0_Pin_15
	GPI0_Speed; /*最大输出速度-GPI0_Speed_2MHz，
	                        -GPI0_Speed_10MHz,
							-GPI0_Speed_50MHz,
			    */
	GPIO_Mode;  /*模式-GPIO_Mode_Out_PP通用输出推挽push-pull	   	  -GPIO_Mode_IPU输入上拉
					 -GPIO_Mode_Out_OD通用输出开漏out-open	 		-GPIO_Mode_IPD输入下拉
					 -GPIO_Mode_AF_PP复用alternate function输出推挽   -GPIO_Mode_IN_FLOATING输入浮空
				     -GPIO_Mode_AF_OD复用输出开漏		                -GPIO_Mode_AIN模拟模式
				*/
}
```

向输出数据寄存器写0/1

```c
void GPIO_WriteBit(GPIOTypeDef *GPIOx,	//端口号,取A B C D..
					uint16_t GPIO_Pin,  //引脚编号
					BitAction BitValue)；//要写入的值	- Bit_RESET  0	 -Bit_SET	1
```

<img src="./STM32面试考点.assets/image-20250308172750560.png" alt="image-20250308172750560" style="zoom:40%;" />

### 编程实现

```c
int main(void)
{
	// #1.开启GPIOC的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	// #2.初始化IO引脚，PC13通用输出开漏模式2MHz
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	GPIO_InitStruCt.GPIO_Pin = GPIO_Pin_13;
	GPIO InitStruct.GPIO_Mode = GPIO Mode_OuT OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
    
	GPIO_Init (GPIOC, &GPIO_INITSTRUCT)
    
    while(1){
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); //亮
		Delay(100);	//延迟100ms
		GPIO_WriteBit(GPIOC, GPIO_PIN_13, BIt_SET) ; // 熄灭
		Delay(100); //延迟100ms
}
```

编译，调试按钮

<img src="./STM32面试考点.assets/image-20250308173550179.png" alt="image-20250308173550179" style="zoom: 33%;" />

编译，下载到单片机

<img src="./STM32面试考点.assets/image-20250308174630299.png" alt="image-20250308174630299" style="zoom:43%;" />

# 2.4_[GPIO]4种输入模式

<img src="./STM32面试考点.assets/image-20250308195755249.png" alt="image-20250308195755249" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250308195808457.png" alt="image-20250308195808457" style="zoom:60%;" />

## 概述

<img src="./STM32面试考点.assets/image-20250308202615119.png" alt="image-20250308202615119" style="zoom:40%;" />

简化图。保护二极管防止静电损坏单片机芯片

<img src="./STM32面试考点.assets/image-20250308202933176.png" alt="image-20250308202933176" style="zoom:33%;" />

## 上拉电阻和下拉电阻的作用

上拉电阻和下拉电阻起稳定作用。上拉电阻提供默认高电压，下拉电阻提供默认低电压。

<img src="./STM32面试考点.assets/image-20250308203214345.png" alt="image-20250308203214345" style="zoom:40%;" />

当IO引脚悬空时（不接任何负载），上拉电阻或下拉电阻提供默认电压，使电路稳定。

## 上拉电阻的工作原理

1.上拉电阻在IO引脚悬空时提供默认高电压。 

2.施密特触发器实现电压到数字的转换，等效电阻为无穷大。 

未悬空状态下

<img src="./STM32面试考点.assets/image-20250308203717884.png" alt="image-20250308203717884" style="zoom:43%;" />

悬空状态下，上拉电阻与施密特触发器串联，分压后施密特触发器该点为高电压3.3V，经转换，寄存器读到1

<img src="./STM32面试考点.assets/image-20250308203944901.png" alt="image-20250308203944901" style="zoom:43%;" />

# 2.5_[GPIO]按钮实验

<img src="./STM32面试考点.assets/image-20250308212445772.png" alt="image-20250308212445772" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250308212457367.png" alt="image-20250308212457367" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250308212509675.png" alt="image-20250308212509675" style="zoom:60%;" />

## LED接线方法

<img src="./STM32面试考点.assets/image-20250308212732933.png" alt="image-20250308212732933" style="zoom:33%;" />

### 外接LED

<img src="./STM32面试考点.assets/image-20250308212838469.png" alt="image-20250308212838469" style="zoom:33%;" />

<img src="./STM32面试考点.assets/image-20250308212915189.png" alt="image-20250308212915189" style="zoom:33%;" />

### 代码设计

<img src="./STM32面试考点.assets/image-20250308213137650.png" alt="image-20250308213137650" style="zoom:43%;" />

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);//开启GPIOA的时钟

GPIO_InitTypeDef GPIO_InitStruct;//定义类型，起名字
GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_0;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;//输出推挽模式
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;

GPIO_Init(GPIOA, &GPIO_InitStruct);//初始化PA0

GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);//向PA0写1,点亮LED
```

## 按钮接线方法

使能上拉电阻

<img src="./STM32面试考点.assets/image-20250308213622934.png" alt="image-20250308213622934" style="zoom:33%;" />

<img src="./STM32面试考点.assets/image-20250308213725967.png" alt="image-20250308213725967" style="zoom:33%;" />

### 代码设计

```c
//RCC_APB2PeriphClockCmd(GPIOA, ENABLE);	//已经开过一遍了，没必要再开了

//GPIO_InitTypeDef GPIO_InitStruct;	//声明过了，没必要再次声明
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_1;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
//GPIO_InitStruct.GPIo_Speed = …;	//最大输出速度只对输出模式有效
   
GPIO_Init(GPIOA, &GPIO_InitStruct);
```

## 8种变量模式

<img src="./STM32面试考点.assets/image-20250308214203863.png" alt="image-20250308214203863" style="zoom:43%;" />

## 读取IO引脚值

```c
uint8_t GPIO_ReadInputDataBit(GPIOTypeDef *GPIOx,	//端口号
							  uint16_t GPIO_Pin);   //引脚编号
作用：读取IO引脚的值，Bit_SET - 1 	Bit_RESET - 0
```

<img src="./STM32面试考点.assets/image-20250308214736545.png" alt="image-20250308214736545" style="zoom:40%;" />

## 总代码

<img src="./STM32面试考点.assets/image-20250308215018424.png" alt="image-20250308215018424" style="zoom:50%;" />

# ——————————————

# 3.1_[串口]通信协议

<img src="./STM32面试考点.assets/image-20250308220512852.png" alt="image-20250308220512852" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250308220521247.png" alt="image-20250308220521247" style="zoom:60%;" />

## 串口通信接口介绍

1.串口是一种通信接口，用于在两颗芯片之间传输数据。 

2.串口由两个引脚组成：TX（发送）和RX（接收）。 

3.接线时需交叉连接TX和RX引脚。

<img src="./STM32面试考点.assets/image-20250308220833045.png" alt="image-20250308220833045" style="zoom:33%;" />

## 串口数据格式

数据格式包括起始位、数据位、停止位。==空闲状态下为高电压==，起始位为低电压，停止位恢复到高电压

从低位往高位传输，每次传1个字节

<img src="./STM32面试考点.assets/image-20250308221150215.png" alt="image-20250308221150215" style="zoom:40%;" />

  

<img src="./STM32面试考点.assets/image-20250309102938255.png" alt="image-20250309102938255" style="zoom:33%;" />

<img src="./STM32面试考点.assets/image-20250309102954889.png" alt="image-20250309102954889" style="zoom:30%;" />

### 串口数据帧格式

若选择8位数据位置，则7位传输有效数据，最后1位用来校验

<img src="./STM32面试考点.assets/image-20250309103640077.png" alt="image-20250309103640077" style="zoom:40%;" />

### 校验位的使用方法

1.校验位有两种校验方式：奇校验和偶校验。 

2.奇校验要求数据位中有奇数个1。 

3.偶校验要求数据位中有偶数个1。

4.发送方在数据帧中添加校验位，接收方检验数据位中的1的个数是否符合校验要求。

<img src="./STM32面试考点.assets/image-20250309103955968.png" alt="image-20250309103955968" style="zoom:40%;" />

# 3.2_[串口]USART模块的使用方法

<img src="./STM32面试考点.assets/image-20250309104901698.png" alt="image-20250309104901698" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309104912339.png" alt="image-20250309104912339" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309104924502.png" alt="image-20250309104924502" style="zoom:60%;" />

## usart模块基本使用方法

1.介绍user模块的内部结构框图，包括ts引脚和rx引脚。 

2.解释发送数据和接收数据的过程，通过发送数据寄存器和接收数据寄存器进行数据传输。

<img src="./STM32面试考点.assets/image-20250309113121671.png" alt="image-20250309113121671" style="zoom:40%;" />

### 移位寄存器和串并转换

发送数据：把这个发送的数据写入到发送数据寄存器，这八个比特位是同时存储在这个发送数据寄存器。同时存储，我们就把它叫做并行。一个比特位，一个比特位的发，这就是一个串行的数据。需要使用这个移位寄存器进行串并转换。

接收数据：我们接收数据的时候是通过这个rs引脚一个比特位，一个比特位的去接收这个高低电压，然后把它转换成数据存储在这个接收数据寄存器里，这里是一块存储的，所以叫做并行，这里是一个比特位，一个比特位接收的，所以叫做串行。

### 原理

- 发送数据

100写入到发送数据寄存器当中，usart模块会自动的把这个数据移动到这个移位寄存器里。

最后发送，串口数据帧的一个格式，是低位在前，高位在后

<img src="./STM32面试考点.assets/image-20250309113749996.png" alt="image-20250309113749996" style="zoom:43%;" />

- 接收数据

<img src="./STM32面试考点.assets/image-20250309113916821.png" alt="image-20250309113916821" style="zoom:43%;" />

## 数据帧格式设置方法

1.介绍串口数据帧的格式，包括起始位、数据位和停止位。

 2.说明数据位长度、停止位长度和校验方式的可设置性。

<img src="./STM32面试考点.assets/image-20250309114010292.png" alt="image-20250309114010292" style="zoom:43%;" />

此处设置数据帧格式

<img src="./STM32面试考点.assets/image-20250309114059284.png" alt="image-20250309114059284" style="zoom:40%;" />

## 波特率设置方法

<img src="./STM32面试考点.assets/image-20250309114140383.png" alt="image-20250309114140383" style="zoom:40%;" />

### 实际例子

波特率寄存器是用来设置上面这个分屏器的分屏系数。

分屏器呢就是对频率做除法的一种结构

<img src="./STM32面试考点.assets/image-20250309114840225.png" alt="image-20250309114840225" style="zoom:50%;" />

## 串口初始化编程接口

```c
初始化串口
void USART_Init(USARTTypeDef *USARTx,					//串口名称
				USART_InitTypeDef *USART_InistStruct);  //初始化的参数
```

```c
初始化的参数
struct USART_InitTypeDef{
    
	uint32_t USART_BaudRate;	//波特率
	uint16_t USART_WordLength;  /*数据位长度 - USART_WordLength_8b
										   - USART_WordLength_9b */
	uint16_t USART_StopBits;    /*停止位长度 - USART_StopBits__5
										   - USART_StopBits_1
										   - USART_StopBits_1_5
										   - USART_StopBits_2 */
	uint16_t USART_Parity;      /*校验方式  - USART_Parity_No
                                          - USART_Parity_Even 偶校验
                                          - USART_Parity_Odd 奇校验 */
	uint16_t USART_Mode;	/*数据收发方向  - USART_Mode_Tx
										  - USART_Mode_Rx
                                          - USART_Mode_Tx | USART_Mode_Rx 收发双向 */
}
```

<img src="./STM32面试考点.assets/image-20250309123001114.png" alt="image-20250309123001114" style="zoom:40%;" />

# 3.3_[串口]为串口初始化IO引脚

<img src="./STM32面试考点.assets/image-20250309123215731.png" alt="image-20250309123215731" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309123229275.png" alt="image-20250309123229275" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309123304343.png" alt="image-20250309123304343" style="zoom:60%;" />

## GPIO库函数初始化方法

<img src="./STM32面试考点.assets/image-20250309123510893.png" alt="image-20250309123510893" style="zoom:40%;" />

## USART模块引脚分布

CTS和RTS用于硬件流控，CK用于同步模式

<img src="./STM32面试考点.assets/image-20250309144614861.png" alt="image-20250309144614861" style="zoom: 33%;" />

### 引脚分布表

引脚分布表包含多种封装信息，不同封装引脚分布不同

C8T6引脚分布

<img src="./STM32面试考点.assets/image-20250309144916150.png" alt="image-20250309144916150" style="zoom:50%;" />

### 通用，复用

<img src="./STM32面试考点.assets/image-20250309145131410.png" alt="image-20250309145131410" style="zoom:40%;" />

### 重映射功能及设置

重映射功能允许将外设引脚映射到其他位置，如果PA9和PA10用不了，可以USART的TX和RX引脚可以重映射到PB6和PB7

<img src="./STM32面试考点.assets/image-20250309145400253.png" alt="image-20250309145400253" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250309145511494.png" alt="image-20250309145511494" style="zoom:33%;" />

参考手册，了解重映射表，以下是整理分类表

![image-20250309145602869](./STM32面试考点.assets/image-20250309145602869.png)

### IO配置表及其使用

<img src="./STM32面试考点.assets/image-20250309150352571.png" alt="image-20250309150352571" style="zoom:43%;" />

位置

<img src="./STM32面试考点.assets/image-20250309150445006.png" alt="image-20250309150445006" style="zoom:33%;" />

- 全双工模式：收发同时进行，两根线互不影响
- 半双工模式：共用1根线发送，不能同时收发
- 同步模式：增加CK线，用于传输时钟信号
- 硬件流控：增加CTS和RTS线，用于流控

<img src="./STM32面试考点.assets/image-20250309150845980.png" alt="image-20250309150845980" style="zoom: 50%;" />

## 编写代码(默认PA9 PA10)

```c
GPIO_InitTypeDef GPIO_InitStruct;
//TxPA9复用输出推挽
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
GPIO_InitStruct.GPIO_Pin   = GPIO_Pin_9;
GPIO_InitStruct.GPIO_Mode  = GPIO_Mode_AF_PP; 	//模式
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_10MHz;	//最大速度
GPIO_Init(GPIOA, &GPIO_InitStruct);

//RxPA10输入浮空输入上拉
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_10;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;// 模式
GPIO_Init(GPIOA, &GPIO_InitStruct);
```

<img src="./STM32面试考点.assets/image-20250309151520303.png" alt="image-20250309151520303" style="zoom:50%;" />

## 编写代码(重映射 PB6 PB7)

使用AFIO模块进行重映射设置，并初始化PB6为复用输出推挽模式，PB7为输入上拉模式

<img src="./STM32面试考点.assets/image-20250309151606751.png" alt="image-20250309151606751" style="zoom:43%;" />

```c
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE); //使能AFIO模块的时钟
GPIO_PinRemapConfig(GPIO_Remap_USART1, ENABLE); //USART1_REMAP=1 重映射打开

GPIO_InitTypeDef GPIO_InitStruct;
//Tx PB6输出推挽
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_6;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP; //模式
GPIO_InitStruct.GPIO_Speed= GPIO_Speed_10MHz;//最大速度
GPIO_Init(GPIOB, &GPIO_InitStruct);

//Rx PB7输入浮空输入上拉
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_7;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;//模式
GPIO_Init(GPIOB, &GPIO_InitStruct);
```

<img src="./STM32面试考点.assets/image-20250309213706843.png" alt="image-20250309213706843" style="zoom:50%;" />

# 3.4_[串口]发送数据

<img src="./STM32面试考点.assets/image-20250309213816940.png" alt="image-20250309213816940" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309213836470.png" alt="image-20250309213836470" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309213850650.png" alt="image-20250309213850650" style="zoom:60%;" />

## 串口数据发送过程

1.串口数据发送过程包括CPU将数据写入发送数据寄存器，数据通过移位寄存器逐比特位发送。 

2.发送过程中，发送数据寄存器为空时才允许写入新数据，防止数据覆盖。 

3.数据发送完成后，发送数据寄存器和移位寄存器都变为空。

## TxE和TC标志位

通过查询标志位的值，可以是0/1，反映了它模块当前的一个工作状态

<img src="./STM32面试考点.assets/image-20250309215210052.png" alt="image-20250309215210052" style="zoom:40%;" />

- TxE标注位表示发送数据寄存器是否为空，TxE=1表示发送数据寄存器为空，反之有数据 TxE=0

我们在发送数据之前，也就是把这个数据写入到这个发送数据寄存器之前，我们最好先判断一下这个发送数据寄存器是不是空的

- TC标注位表示数据发送是否完成，TC=1表示发送数据寄存器和移位寄存器都为空，发送完成

<img src="./STM32面试考点.assets/image-20250309221828305.png" alt="image-20250309221828305" style="zoom:40%;" />

## 编程接口介绍(3组)

```c
作用：控制USART模块的使能和禁止（总开关）
void USART_Cmd(USARTTypeDef *USARTx,	  //串口名称
			   FunctionalState NewState); //ENABLE-使能; DISABLE-禁止
```

<img src="./STM32面试考点.assets/image-20250309223212535.png" alt="image-20250309223212535" style="zoom:50%;" />

```c
作用：查询USART标志位的值。返回值：RESET - 0; SET - 1
FlagStatus USART_GetFlagStatus(USARTTypeDef *USARTx, //串口名称
							   uint16_t USART_FLAG); //要查询的标志位
```

<img src="./STM32面试考点.assets/image-20250309224143154.png" alt="image-20250309224143154" style="zoom:40%;" />

```c
作用：把要发送的数据写入到发送数据寄存器里
void USART_SendData(USARTTypeDef *USARTx,	//串口名称
					    uint16_t Data);		//要发送的数据，如果用无符号的8位int，当数据为9位时候就发送不出
```

<img src="./STM32面试考点.assets/image-20250309224442440.png" alt="image-20250309224442440" style="zoom:40%;" />

## 发送数据代码实现

```c
// @作用:使用串口一次性发送多个字节
// @参数:pData - 要发送的数据	Size - 字节的数量
void My_USART_sendBytes(USART_TypeDef *USARTx, uint8_t *pData, uint16_t Size)
{
	for(uint32_t i = 0; i < Size; i++)
    {
		// #1.等待发送数据寄存器空
		while(USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
		// #2，将要发送的数据写入到发送数据寄存器
		USART_SendData(USARTx, pData[i]);
	}
	// #3.等待数据发送完成
	while(USART_GetFlagStatus(USART1, USART_FLAG_TC) == RESET);
}
```

## ==串口发送完整代码==

```c
#include "stm32f10x.h"
#include <stdio.h>
#include "delay.h"

void My_USART_sendBytes(USART_TypeDef *USARTx, uint8_t *pData, uint16_t Size);
void My_USART_Init(void);

int main(void)
{
    Delay_Init();
    My_USART_Init();
    
//  uint8_t bytesToSend[] = {1,2,3,4,5};
    
//  My_USART_sendBytes(USART1, bytesToSend, 5);
    
//	printf("Hello world. \r\n");
    
	while(1)
    {
		uint32_t currentTick = GetTick(); // 获取当前的时间

		uint32_t miliseconds = currentTick % 1000; currentTick/=1000;
		uint32_t seconds = currentTick % 60;	   currentTick/=60;
		uint32_t minute  = currentTick % 60;       currentTick/=60;
		uint32_t hour    = currentTick;

		printf("%02u:%02u:%02u.%03u\r\n", hour, minute, seconds, miliseconds);
        
        Delay(100);
    }
}

//
// @简介：通过串口发送多个字节
// @参数 USARTx：填写串口的名称
// @参数 pData：要发送的数据
// @参数Size：要发送数据的数量，单位是字节
//
void My_USART_sendBytes(USART_TypeDef *USARTx, uint8_t *pData, uint16_t Size)
{
    for(uint32_t i = 0; i < Size; i++)
    {
		// #1.等待发送数据寄存器空
		while(USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
		// #2，将要发送的数据写入到发送数据寄存器
		USART_SendData(USARTx, pData[i]);
	}
	// #3.等待数据发送完成
	while(USART_GetFlagStatus(USART1, USART_FLAG_TC) == RESET);
}

//
// @简介：对USART1进行初始化
// 	     PB6 - Tx, PB7 - Rx
//       115200, 8, 1, None, 双向
//
void My_USART_Init(void)
{
    // #1.初始PB6和PB7
	GPIO_InitTypeDef GPIO_InitStruct;
    
//	// PA9 tX
//	RCC_APB2PeriphCloCkCmd(RCC _APB2Periph_GPIOA, ENABLE);
//	GPIO_InitStruCt.GPIO_Pin = GPIO_Pin_9;
//	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
//	GPIO_InitStruCt.GPIO_Speed = GPIO_Speed_10MHz;
//	GPIO_Init(GPIOA, &GPIO_InitStruct);
//
//  // PA10 RX
//	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
//	GPIO InitStruct.GPIO_Pin = GPIO_Pin_10;
//	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
//	GPIO_Init(GPIOA, &GPIO_InitStruct);

	RCC_APB2PriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
    GPIO_PinRemapConfig(GPIO_Remap_USART1, ENABLE);
    
    // PB6 重映射
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
    
	GPIO_InitStruct.GPIO_Pin   = GPIO_Pin_6;
	GPIO_InitStruct.GPIO_Mode  = GPIO_Mode_AF_PP; 	
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_10MHz;	
	GPIO_Init(GPIOB, &GPIO_InitStruct);
    
    // PB7 重映射
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
    
	GPIO_InitStruct.GPIO_Pin   = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode  = GPIO_Mode_IPU; 	
	GPIO_Init(GPIOB, &GPIO_InitStruct);
    
    // #2.初始化USART1
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
    
	USARI_InitTypeDef USART_InitStruct;
    
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_Parity = USART_Parity_No;
    
	USART_Init(USART1, &USART_InitStruct);
    
	USART_Cmd(USART1, ENABLE); // 闭合总开关
}

int fputc(int ch, FILE *f)
{
	// #1.等待发送数据寄存器为空，等待TXE = 1
	while(USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
	// #2.写入发数据寄存器当中
	USART_SendData(USART1, (uint8_t)ch);
    
	return ch;
}

```

<img src="./STM32面试考点.assets/image-20250309235309615.png" alt="image-20250309235309615" style="zoom: 33%;" />

使用串口调试助手软件配置串口参数，与单片机保持一致

<img src="./STM32面试考点.assets/image-20250310094320079.png" alt="image-20250310094320079" style="zoom:50%;" />

选择hex，设置完这个之后，我们按一下这个单片机的复位按钮

<img src="./STM32面试考点.assets/image-20250310094812447.png" alt="image-20250310094812447" style="zoom:40%;" />

# 3.5_[串口]格式化打印字符串

<img src="./STM32面试考点.assets/image-20250309214218792.png" alt="image-20250309214218792" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309214229619.png" alt="image-20250309214229619" style="zoom:60%;" />

## 格式化打印字符串解释

这里我们使用printf去打印这行问候语的时候，就是在使用格式化打印字符串。

<img src="./STM32面试考点.assets/image-20250310095518981.png" alt="image-20250310095518981" style="zoom:40%;" />

## 工程重命名和文件夹管理

例如，修改文件夹名称为printf_test，工程名称为printf_test

<img src="./STM32面试考点.assets/image-20250310095733366.png" alt="image-20250310095733366" style="zoom:50%;" />

### 代码整理和初始化封装

详情请看串口发送完整代码

[串口发送完整代码](##串口发送完整代码)

## 格式化字符串编程原理

printf这个函数，做了两件事：

1. 生成格式化字符串
2. fputc是一个函数，它的作用是每次发送一个字符，所以对于这个字符串有多少个字符，就需要调用多少次。只看第一个参数，这个参数呢叫ch，表示我们要发送的那个字符
3. 因为每次发送一个字符，所以每次通过这个参数传进来一个字符，然后这个字符会被发送到控制台里。但是对于单片机来说，它是没有控制台的。注意观察，这个fputc这个函数前面是有一个_weak关键字代表了这个函数是能够被重写的，所以重写这个函数，把它改成这种形式——不把它发送到这个控制台上了，而是通过串口把这个字符给它发送出去


<img src="./STM32面试考点.assets/image-20250310180723321.png" alt="image-20250310180723321" style="zoom:40%;" />

## 重写fputc函数

```c
int fputc(int ch, FILE *f){
	// #1.等待发送数据寄存器为空，等待TXE = 1
	while(USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
	// #2.写入发数据寄存器当中
	USART_SendData(USART1, (uint8_t)ch);
	return ch;
}
```

## 打印单片机当前时间

获取单片机当前时间的方法：get_take函数

```c
uint32_t currentTick = GetTick(); // 获取当前的时间

// 计算毫秒、秒、分钟和小时的值
uint32_t miliseconds = currentTick % 1000; currentTick/=1000;
uint32_t seconds = currentTick % 60;	   currentTick/=60;
uint32_t minute  = currentTick % 60;       currentTick/=60;
uint32_t hour    = currentTick;

printf("%02u:%02u:%02u.%03u\r\n", hour, minute, seconds, miliseconds); //占位符号02u，两位整数的形式，不足两位会自动补0
```

<img src="./STM32面试考点.assets/image-20250310195831568.png" alt="image-20250310195831568" style="zoom:40%;" />

# 3.6_[串口]接收数据

<img src="./STM32面试考点.assets/image-20250309214339129.png" alt="image-20250309214339129" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250309214347949.png" alt="image-20250309214347949" style="zoom:60%;" />

## RxNE标志位

<img src="./STM32面试考点.assets/image-20250310222159579.png" alt="image-20250310222159579" style="zoom:40%;" />

## 数据接收的代码

### 编程接口

```c
作用：从接收数据寄存器读取数据
uint16_t USART_ReceiveData(USARTTypeDef *USARTx); //串口名称
```

### 框架设计

```c
// #1.等待接收数据寄存器非室
while(USART_GetFlagStatus(USARTx, USART_FLAG_RXNE) == RESET);
// #2.接收数据
uint8_t btyeRcvd = USART_ReceiveData(USARTx);
// #3.处理数据
...
```

## 串口去控制LED代码

```c
#include "stm32f10x.h"

void My_USART_Init(void);
void My_OnBoardLED_Init(Void);

int main(void)
{
    My_USART_Init();
    My_OnBoardLED_Init();
    
	while(1)
    {
		// #1.等待接收数据寄存器非空
		while(USART_GetFlagStatus(USARTx, USART_FLAG_RXNE) == RESET);
		// #2.接收数据
		uint8_t btyeRcvd = USART_ReceiveData(USART1);
		// #3.处理数据
        if(byteRcvd == '0')
			GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET); // 亮灯
		else if(byteRcvd == '1')
			GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); // 灭灯
    }
}

//
// @简介：对USART1进行初始化
// 	     PB6 - Tx, PB7 - Rx
//       115200, 8, 1, None, 双向
//
void My_USART_Init(void)
{
    // #1.初始PB6和PB7
	GPIO_InitTypeDef GPIO_InitStruct;
    
//	// PA9 tX
//	RCC_APB2PeriphCloCkCmd(RCC _APB2Periph_GPIOA, ENABLE);
//	GPIO_InitStruCt.GPIO_Pin = GPIO_Pin_9;
//	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
//	GPIO_InitStruCt.GPIO_Speed = GPIO_Speed_10MHz;
//	GPIO_Init(GPIOA, &GPIO_InitStruct);
//
//  // PA10 RX
//	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
//	GPIO InitStruct.GPIO_Pin = GPIO_Pin_10;
//	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
//	GPIO_Init(GPIOA, &GPIO_InitStruct);

	RCC_APB2PriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
    GPIO_PinRemapConfig(GPIO_Remap_USART1, ENABLE);
    
    // PB6
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
    
	GPIO_InitStruct.GPIO_Pin   = GPIO_Pin_6;
	GPIO_InitStruct.GPIO_Mode  = GPIO_Mode_AF_PP; 	
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_10MHz;	
	GPIO_Init(GPIOB, &GPIO_InitStruct);
    
    // PB7
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
    
	GPIO_InitStruct.GPIO_Pin   = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode  = GPIO_Mode_IPU; 	
	GPIO_Init(GPIOB, &GPIO_InitStruct);
    
    // #2.初始化USART1
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
    
	USARI_InitTypeDef USART_InitStruct;
    
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_Parity = USART_Parity_No;
    
	USART_Init(USART1, &USART_InitStruct);
    
	USART_Cmd(USART1, ENABLE); // 闭合总开关
}

void My_OnBoardLED_Init(Void)
{
	GPIO_InitTypeDef GPIO_InistStruct;
    
    RCC_APB2PeriphCloCkCmd(RCC _APB2Periph_GPIOC, ENABLE);
    
	GPIO_InitStruCt.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruCt.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOC, &GPIO_InitStruct);
    
    GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
}

```

## 错误标志位

<img src="./STM32面试考点.assets/image-20250310231242244.png" alt="image-20250310231242244" style="zoom:40%;" />

### PE：奇偶校验错

<img src="./STM32面试考点.assets/image-20250311101222063.png" alt="image-20250311101222063" style="zoom:40%;" />

### FE：帧格式错误

<img src="./STM32面试考点.assets/image-20250311101703500.png" alt="image-20250311101703500" style="zoom:40%;" />

### NE：噪声错

串口的一个数据帧，接收方在收数据的时候其实它是对每一个位进行多次采样的，比如说它进行三次采样，那采样的都是高电压的时候，它才认为是一个1。如果三次都是低电压的话，它才认为是一个0

但是如果出现了这种情况，两次采到了低电压，一次采到了高电压，那么这种情况既不是0也不是1，这是因为这里有噪声的干扰，造成了接收的一个错误，认为这是一个无效的数据，是噪声错

<img src="./STM32面试考点.assets/image-20250311102120682.png" alt="image-20250311102120682" style="zoom:40%;" />

### ORE：过载错

出现第三个字节进来的时候，把第二个字节给覆盖掉了，导致第二个字节丢失。因为对方发数据的速度比收数据的速度要快，读的太慢了，所以就导致中间的这个字节丢失了这种错误，叫做过载错

<img src="./STM32面试考点.assets/image-20250311103039944.png" alt="image-20250311103039944" style="zoom:55%;" />

# 3.7_[串口]封装常用功能

<img src="./STM32面试考点.assets/image-20250310203605927.png" alt="image-20250310203605927" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310203619743.png" alt="image-20250310203619743" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310203630452.png" alt="image-20250310203630452" style="zoom:60%;" />

## my_lib文件夹介绍

<img src="./STM32面试考点.assets/image-20250311104538718.png" alt="image-20250311104538718" style="zoom:40%;" />

<img src="./STM32面试考点.assets/image-20250311104600516.png" alt="image-20250311104600516" style="zoom:40%;" />

## 串口数据发送函数

```c
void My_USART_SendByte(...);	//发送一个字节
void My_USART_SendBytes(...);	//发送多个字节
void My_USART_SendChar(...);	//发送一个字符
void My_USART_SendString(...);	//发送字符串
void My_USART_Printf(...);		//发送格式化字符串
```

<img src="./STM32面试考点.assets/image-20250311110119579.png" alt="image-20250311110119579" style="zoom:43%;" />

## 串口数据接收函数

```c
My_USART_ReceiveByte(...);	//接收一个字节
My_USART_ReceiveBytes(...);	//接收多个字节
My_USART_ReceiveLine(...);	//接收一行字符串
```

<img src="./STM32面试考点.assets/image-20250311112025180.png" alt="image-20250311112025180" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250311112143287.png" alt="image-20250311112143287" style="zoom:50%;" />

把鼠标放在需要的函数上。点击F12按钮进入到这个源码

<img src="./STM32面试考点.assets/image-20250311112727830.png" alt="image-20250311112727830" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250311112853243.png" alt="image-20250311112853243" style="zoom:50%;" />

# ——————————————

# 4.1_[I2C]基本电路结构

<img src="./STM32面试考点.assets/image-20250310203922149.png" alt="image-20250310203922149" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310203937827.png" alt="image-20250310203937827" style="zoom:60%;" />

## I2C总线与串口通信的区别和优势

串口通信只能实现一对一的通信，限制了设备连接数量。串口数量有限，无法连接大量设备

## I2C总线的基本电路结构

1.I2C总线由两根线组成：SCL（时钟线）和SDA（数据线）。 

2.总线只有一个主机，多个从机，主机通常由单片机担任。 

3.所有从机的SCL和SDA引脚都连接到总线上。 每个从机都有一个七位地址，地址范围从0000000到1111111，共128个地址，注意存在特殊地址不能直接用

4.总线两端分别接有上拉电阻。

<img src="./STM32面试考点.assets/image-20250311130646353.png" alt="image-20250311130646353" style="zoom:40%;" />

## 时钟线和数据线的作用

1.时钟线SCL：由主机发送，控制数据传输的速率。信号的频率决定了数据传输的快慢。 

2.数据线SDA：双向通信，可以由主机或从机发送数据。 

<img src="./STM32面试考点.assets/image-20250311151325469.png" alt="image-20250311151325469" style="zoom:40%;" />

## 逻辑线与

1.通过硬件电路实现逻辑与运算，称为逻辑线与。 

2.所有从机的SCL和SDA引脚都设置为开漏输出。 

3.上拉电阻确保当所有引脚输出高阻抗时，总线保持高电平。 

4.当任意引脚输出低电平时，总线被拉低。

<img src="./STM32面试考点.assets/image-20250311151731417.png" alt="image-20250311151731417" style="zoom:40%;" />

## 主机发送时钟信号

时钟信号只能由主机，也就是单片机产生，然后发送给从机，方向是单向的。通过逻辑线与实现高低电压。SCL引脚交替输出低电平和高电平，控制时钟信号

<img src="./STM32面试考点.assets/image-20250311152056874.png" alt="image-20250311152056874" style="zoom:40%;" />

## 主机向从机发送数据

SDA引脚输出低电平或高电平，表示数据0或1。同样是逻辑线与实现高低电压

## 从机向主机发送数据

 主机向SDA引脚写1，释放总线，其他的从机呢也都写一个1把引脚呢从SDA线上断开，发生从机可以向总线写入数据0或1

<img src="./STM32面试考点.assets/image-20250311152549512.png" alt="image-20250311152549512" style="zoom:40%;" />

# 4.2_[I2C]通信协议

<img src="./STM32面试考点.assets/image-20250310204109329.png" alt="image-20250310204109329" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204123356.png" alt="image-20250310204123356" style="zoom:60%;" />

## I2C通信基本流程

寻址：主机往总线上去发送从机的7位地址，地址后边还跟着一个比特位，表示数据通信的方向（读还是写）

<img src="./STM32面试考点.assets/image-20250311154151921.png" alt="image-20250311154151921" style="zoom:40%;" />

## I2C数据帧格式

起始位，停止位，这是7位地址+数据方向，然后中间是数据传输。一次性能传输多个字节

串口是每次只能传输八到九个比特位，一般我们是传输一个字节

### 起始位和停止位

起始位：在SCL是高电压的时候，SDA上出现一个下降沿

停止位：在SCL是高电压的时候，SDA上出现一个上升沿

<img src="./STM32面试考点.assets/image-20250311154916906.png" alt="image-20250311154916906" style="zoom:40%;" />

### 寻址阶段

rw位就是方向位，读1写0

从机会发送一个ACk来应答，这个应答信号就相当于主机叫从机的名字，从机回到，当主机收到这个ack之后，就意味着寻址成功

<img src="./STM32面试考点.assets/image-20250311155423052.png" alt="image-20250311155423052" style="zoom:40%;" />

### ACK应答原理

主机先释放掉SDA，写一个1，此时会从这个总线上的SDA这条线上断开了，这条线就悬空了，但是由于这个上拉电阻的作用，SDA这条线就呈现一个高电压，如果没有设备应答它的话，那么它就始终维持一个高电压。如果有设备要应答，就会把这个SDA重新拉低，主机就会检测到一个低电压从机

### 数据位

主机发送数据，就是主机去发送一个字节，然后从机回一个ack；主机发一个字节，从机回一个ack

如果是主机读取从机发送，就是从机发送一个字节，主机发送一个ack；从机发送一个字节。主机发送一个ack

<img src="./STM32面试考点.assets/image-20250311160045741.png" alt="image-20250311160045741" style="zoom:40%;" />

## 发数据和收数据的流程例子

<img src="./STM32面试考点.assets/image-20250311161023838.png" alt="image-20250311161023838" style="zoom:43%;" />

注意：当我们读取完这个数据之后，主机不想读新的数据了，所以这里发的是一个NAK，也就是不应答

<img src="./STM32面试考点.assets/image-20250311161448596.png" alt="image-20250311161448596" style="zoom:43%;" />

# 4.3_[I2C]I2C模块的使用方法

<img src="./STM32面试考点.assets/image-20250310204237175.png" alt="image-20250310204237175" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204248526.png" alt="image-20250310204248526" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204257833.png" alt="image-20250310204257833" style="zoom:60%;" />

## I2C模块介绍

I2C模块是单片机上的一个片上外设，提供I2C接口。I2C模块使用SCL和SDA两个引脚，分别是时钟线和数据线。

## IO引脚初始化

1.IO引脚初始化的步骤：查找引脚分布表、重映射表和IO参数表。 

2.引脚分布表介绍单片机的48个引脚及其主功能和复用功能。

3.IO参数表指定引脚应被初始化为复用输出开漏模式。

<img src="./STM32面试考点.assets/image-20250311162736504.png" alt="image-20250311162736504" style="zoom:50%;" />

## ==I2C完整代码==

main函数

```c
#include "stm32f10x.h"

void My_I2C_Init(void);
void My_OnBoardLED_Init(void);
int My_I2C_SendBytes(I2C_TypeDef *I2Cx, uint8_t Addr, uint8_t *pData, uint16_t Size);
int My_I2C_ReceiveBytes(I2C_TypeDef *I2Cx, uint8_t Addr, uint8_t *pBuffer, uint16_t Size);

int main(void)
{
    My_I2C_Init();
    My_OnBoardLED_Init();
    
    uint8_t commands[] = {0x00, 0x8d, 0x14, 0xaf, 0xa5};
    
    My_I2C_SendBytes(I2C1, 0x78, commands, 5);
    
    uint8_t rcvd;
    
    My_I2C_ReceiveBytes(I2C1, 0x78, &rcvd, 1)
    
    if((rcvd & (0x01 << 6)) == 0)
    {
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET);
	}
    else
    {
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
    }
    
	while(1)
    {
    }
}

void My_I2C_Init (void)
{
	// #1. IO引脚初始化
	// 对I2C1进行重映射
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	GPIO_PinRemapConfig(GPIO_Remap_I2C1, ENABLE);
	
    // 对PB8和PB9进行初始化
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
    
    GPIO_Init(GPIOB, &GPIO_InitStruct);
    
    GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin  = GPIO_Pin_8 | GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_OD; 
	GPIO_InitStruct.GPIO_Speed= GPIO_Speed_2MHz;
    
    // #2. 初始化I2C1模块
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);  // 开启I2C1的时钟
	RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, ENABLE);  // 施加复位信号
	RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, DISABLE); // 释放复位信号

	I2C_InitTypeDef I2C_InitStruct;

	I2C_InitStruct.I2C_ClockSpeed = 400000;	// 波特率400k
	I2C_InitStruct.I2C_Mode = I2C_Mode_I2C;	// 标准的I2C
	I2C_InitStruct.I2C_DutyCycle = I2C_DutyCycle_2; // 占空比 2:1
	I2C_Init(I2C1, &I2C_InitStruct);

	I2C_Cmd(I2C1, ENABLE); // 闭合I2C1的总开关
}

int My_I2C_SendBytes(I2C_TypeDef *I2Cx, uint8_t Addr, uint8_t *pData, uint16_t Size)
{
    // #1.等待总线空闲
    while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_BUSY) == SET);
    
    // #2.发送起始位
	I2C_GenerateStart(I2Cx, ENABLE);
    
	while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_SB) == RESET);
    
    // #3.寻址阶段
	// 清除AF
	I2C_ClearFlag(I2Cx, I2C_FLAG_AF);
	// 发送地址 + RW#
	I2C_SendData(I2Cx, Addr & 0xfe);

	while(1)
    {
		if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_ADDR) == SET) 
        {
            break;
        }
		if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_AF) == SET)
        {
			I2C_GenerateStop(I2Cx, ENABLE); // 发送停止位
			return -1;  // 寻址失败
	    }
     }
    
    // 清除ADDR（先读SR1，再读SR2）
	I2C_ReadRegister(I2Cx, I2C_Register_SR1);
	I2C_ReadRegister(I2Cx, I2C_Register_SR2);
    
    // #4.发送数据
    for(uint16_t i=0; i<Size; i++)
    {
		While(1)
        {
			if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_AF) == SET) //上一个数据是否被拒收
            {
				I2C_Generatestop(I2Cx, ENABLE);
				return -2; // 数据拒收
    		}
			if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_TXE) == SET) 
            {
                break;
            }
		}
        
        I2C_SendData(I2Cx, pData[i]);
	}
    
    while(1)
    {
		if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_AF) == SET) //上一个数据是否被拒收
        {
			I2C_GenerateStop(I2Cx, ENABLE);
			return -2; // 数据拒收
    	}
		if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_BTF) == SET) break;
	}
    
    // #5.发送停止位
    I2C_GenerateStop(T2Cx,ENABLE); // 发送停止位
	return 0; // 成功
}

int My_I2C_ReceiveBytes(I2C_TypeDef *I2Cx, uint8_t Addr, uint8_t *pBuffer, uint16_t Size)
{
    // #1.发送起始位
	I2C_GenerateStart(I2Cx, ENABLE);
    
    while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_SB) == RESET);
    
    // #2.寻址阶段
    I2C_ClearFlag(I2Cx, I2C_FLAG_AF);
	// 发送地址 + RW#
	I2C_SendData(I2Cx, Addr | 0x01);
    
    while(1)
    {
		if(I2C_GETFLAGSTATUS (I2CX, I2C_FLAG_AF) == SET)
        {
			I2C_GenerateSTOP (I2Cx, ENABLE);
			return -1: // 寻址失败
		}
        
		if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_ADDR) == SET)
        {
            break;
        }
    }
    
    // #3.接收数据
    if(Size == 1)
    {
		// 清除ADDR
		I2C_ReadRegister(I2Cx,I2C_Register_SR1);
		I2C_ReadRegister(I2Cx,I2C_Register_SR2);
        
		// ACK=0 STOP=1
		I2C_AcknowledgeConfig(I2Cx, DISABLE);
		I2C_GenerateSTOP(I2Cx, ENABLE);
        
		// 等待RXNE -> 1
		while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
        
		//读取数据
		pBuffer[0] = I2C_ReceiveData(I2Cx);
    }
    
    else if(Size == 2)
    {
        // 清除ADDR
		I2C_ReadRegister(I2Cx,I2C_Register_SR1);
		I2C_ReadRegister(I2Cx,I2C_Register_SR2);
        
        // ACK=1
        I2C_AcknowledgeConfig(I2Cx, ENABLE);
        
        // 等待接收完成
        while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
        
        // 读取第一个字节
        pBuffer[0] = I2C_ReceiveData(I2Cx);    
        
        // ACK=0 STOP=1
		I2C_AcknowledgeConfig(I2Cx, DISABLE);
		I2C_GenerateSTOP(I2Cx, ENABLE);
        
        // 等待接收完成
        while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
        
        // 读取第二个字节
        pBuffer[1] = I2C_ReceiveData(I2Cx); 
    }
    
    else
    {
        // 清除ADDR
		I2C_ReadRegister(I2Cx,I2C_Register_SR1);
		I2C_ReadRegister(I2Cx,I2C_Register_SR2);
        
        // ACK=1
        I2C_AcknowledgeConfig(I2Cx, ENABLE);
        
        for(uint16_t i=0; i<Size-1; i++)
        {
        	// 等待接收完成
        	while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
        
        	// 读取数据
        	pBuffer[i] = I2C_ReceiveData(I2Cx);  
        }
        
        // ACK=0 STOP=1
		I2C_AcknowledgeConfig(I2Cx, DISABLE);
		I2C_GenerateSTOP(I2Cx, ENABLE);
        
        // 等待接收完成
        while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_RXNE) == RESET);
        
        // 读取最后一个数据
        pBuffer[Size-1] = I2C_ReceiveData(I2Cx); 
    }
    
    return 0; // 表示接收成功
    
}

void My_OnBoardLED_Init (Void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2PERIPH_GPIOC, ENABLE);
    
	GPIO_InitTypeDef GPIO_InitStruct;
    
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
    
	GPIO_Init(GPIOC, &GPIO_InitStruct);
}
```

## OLED显示器连接

<img src="./STM32面试考点.assets/image-20250311164341401.png" alt="image-20250311164341401" style="zoom:40%;" />

## I2C速度模式

1个时钟周期传递1个比特位

<img src="./STM32面试考点.assets/image-20250311164819392.png" alt="image-20250311164819392" style="zoom: 40%;" />

1.I2C支持的标准模式(SM)、快速模式(FM)、快速增强模式(FM+)、高速模式(HSM)和超快模式(UFM)。 

2.单片机支持标准模式和快速模式。

<img src="./STM32面试考点.assets/image-20250311165541414.png" alt="image-20250311165541414" style="zoom:33%;" />

## 快速模式下时钟信号的占空比

只有快速模式下可以设置时钟信号的占空比

占空比就是在一个周期里高电压占整个周期的一个比例。默认情况下选择 2比1占空比

<img src="./STM32面试考点.assets/image-20250311171039357.png" alt="image-20250311171039357" style="zoom:40%;" />

## 初始化编程接口

```c
作用：对I2C进行初始化
void I2C_Init(I2CTypeDef *2CX, // I2C的名称，I2C1，I2C2
			  I2C_InitTypeDef *I2C_InitStruct); // 用于传递初始化参数

struct I2C_InitTypeDef{
	uint32_tI2C_ClockSpeed; // 波特率, <=100 Sm, <=400 Fm
	uint16_tI2C_Mode; /* 模式 I2C_Mode_I2C        - 标准I2C模式
							 I2C_Mode_SMBusDevice - 系统管理总线设备模式
							 I2C_Mode_SMBusHost   -系统管理总线主机模式 */
	uint16_tI2C_DutyCycle; /* 快速模式下时钟信号的占空比，I2C_DutyCycle_16_9
													 I2C_DutyCycle_2 */
	uint16_t I2C Ack; 			// 与从机模式有关
	uint16_t I2C_OwnAddress1;   // 与从机模式有关
	uint16_t I2C AcknowledgedAddress; // 用于选择10位从机地址模式
}
```

在使用这个I2C模块之前，需要对这个模块做一个重启，这两行代码就是做重启

```c
RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);		// 开启I2C1的时钟
RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, ENABLE);		// 施加复位信号
RCC_APB1PeriphResetCmd(RCC_APB1Periph_I2C1, DISABLE);	    // 释放复位信号

I2C_InitTypeDef I2C_InitStruct;

I2C_InitStruct.I2C_ClockSpeed = 400000;	// 波特率400k
I2C_InitStruct.I2C_Mode = I2C_Mode_I2C;	// 标准的I2C
I2C_InitStruct.I2C_DutyCycle = I2C_DutyCycle_2; // 占空比 2:1
I2C_Init(I2C1, &I2C_InitStruct);

I2C_Cmd(I2C1, ENABLE); // 闭合I2C1的总开关，断开开关是disable
```

只有闭合这个总开关之后，I2C模块才能正常的传输数据

<img src="./STM32面试考点.assets/image-20250311193251349.png" alt="image-20250311193251349" style="zoom: 43%;" />

# 4.4_[I2C]写数据

<img src="./STM32面试考点.assets/image-20250310204635915.png" alt="image-20250310204635915" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204647528.png" alt="image-20250310204647528" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204702214.png" alt="image-20250310204702214" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310204714738.png" alt="image-20250310204714738" style="zoom:60%;" />

## I2C模块内部结构

状态寄存器SR1和SR2，包含反映模块运行状态的标志位

SDA控制电路，用于控制引脚发送波形或解析波形，下面有3个控制寄存器

SCL控制电路，用于控制引脚时钟信号

<img src="./STM32面试考点.assets/image-20250311194949574.png" alt="image-20250311194949574" style="zoom:50%;" />

## I2C数据发送过程

I2C数据帧格式，包括起始位、寻址阶段、数据传输阶段、停止位

- 起始位的发送，通过start位寄存器写入1，I2C模块会主动把SDA拉低
- 寻址阶段，主机发送7位从机地址+RW位，等待从机响应，通过写入发生数据寄存器实现，ACK位是查询SR1寄存器的标志位
- 数据传输阶段，主机发送数据到从机，等待从机回应ACK
- 停止位的发送，通过在stop写入1，I2C模块会主动把SDA拉高

### 编程接口

```c
作用：通过I2C向从机发送若干个字节
int My_I2C_SendBytes(I2C_TypeDef *I2Cx,   // I2C接口的名称
						  uint8_t Addr,   // 从机地址，靠左对齐
                          uint8_t *pData, // 要发送的数据
						  uint16_t Size); // 要发送的数据的数量（字节）
```

<img src="./STM32面试考点.assets/image-20250311200731204.png" alt="image-20250311200731204" style="zoom: 67%;" />

[I2C完整代码](##I2C完整代码)

### 总线空闲

```c
// #1.等待总线空闲，SET = 1 忙
while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_BUSY) == SET);
```

<img src="./STM32面试考点.assets/image-20250311202218130.png" alt="image-20250311202218130" style="zoom:60%;" />

### 发送起始位

```c
// #2.发送起始位
I2C_GenerateStart(I2Cx, ENABLE);
while(I2C_GetFlagStatus(I2Cx, I2C_FLAG_SB) == RESET);
```

<img src="./STM32面试考点.assets/image-20250311202704227.png" alt="image-20250311202704227" style="zoom:60%;" />

### 寻址阶段

发送7位从机地址+RW位，主机释放SDA线等待从机响应拉低

通过查询ADDR和AF标志位来判断寻址是否成功。如果AF标志位变为1，表示寻址失败；如果ADDR标志位变为1，表示寻址成功

```c
// #3.发送地址
// 清除AF
I2C_ClearFlag(I2Cx, I2C_FLAG_AF);
// 发送地址 + RW#
I2C_SendData(I2Cx, Addr & Oxfe);

while(1){
	if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_ADDR) == SET) break;
	if(I2C_GetFlagStatus(I2Cx, I2C_FLAG_AF) == SET){
		I2C_GenerateStop(I2Cx,ENABLE); // 发送停止位
		return -1; // 寻址失败
    }
}
```

写数据，最后1位RW位清0

<img src="./STM32面试考点.assets/image-20250311203606128.png" alt="image-20250311203606128" style="zoom:50%;" />

寻址成功后需要对ADDR标志位进行清零操作

```c
//清除ADDR（先读SR1，再读SR2）
I2C_ReadRegister(I2Cx, I2C_Register_SR1);
I2C_ReadRegister(I2Cx, I2C_Register_SR2);
```

<img src="./STM32面试考点.assets/image-20250311203824025.png" alt="image-20250311203824025" style="zoom:67%;" />

### 发送数据阶段

```c
for(i=0; i<Size; i++){
	//等待发送数据寄存器空（AF+TXE）
	//发送数据
}
//等待数据发送完成（AF+BTF）
```

在写入这个发送数据寄存器之前，得判断一下这个发送数据寄存器里到底有没有值，如果它本来就有值的话，会造成新的数据把旧的数据给覆盖了。所以我们在发送之数据之前查一下这个TXE标注位，除了去查这个TXE标志位之外，还得看一看上个数据有没有被拒收，也就是这里去查一下这个ACK，就是查AF标注位，若为1，就说明上一个被发的数据拒收了

<img src="./STM32面试考点.assets/image-20250312153552556.png" alt="image-20250312153552556" style="zoom: 67%;" />

```c
// 等待发送数据寄存器空
While(1){
    
	if(I2C_GetFlagStatus(..AF..) == SET){
		I2C_Generatestop(...);
		return -2; // 数据拒收
    }
	if(I2C_GetFlagStatus(..TxE..) == SET) break;
}
```

只有当这两个寄存器都是空的时候，才证明发送完成了，我们可以通过查BTF这个标注位来判断这两个寄存器是不是空。当BTF = 1，就说明我们的这个数据发送完成了。同时，还得查一下这个AF标志位，防止这个上一个数据被拒收。

```c
// 等待数据发送完成，即移位寄存器和发送数据寄存器都是空
while(1){
    
	if(I2C_GetFlagStatus(..AF..) == SET){
		I2C_GenerateStop(...);
		return -2; // 数据拒收
    }
	if(I2C_GetFlagStatus(..BTF..)== SET) break;
}
```

### 发送停止位

```c
I2C_GenerateStop(T2Cx,ENABLE); //发送停止位
return 0; // 成功
```

## 实验验证

<img src="./STM32面试考点.assets/image-20250311205828408.png" alt="image-20250311205828408" style="zoom: 50%;" />

# 4.5_[I2C]读数据

<img src="./STM32面试考点.assets/image-20250310210401419.png" alt="image-20250310210401419" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310210429583.png" alt="image-20250310210429583" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310210445942.png" alt="image-20250310210445942" style="zoom:60%;" />

## 编程接口

```c
作用：通过12C从从机读取若干个字节
返回：0-读取成功  -1-寻址失败(即从机无返回ACK位)
    
int My_I2C_ReceiveBytes(I2C_TypeDef *I2Cx, 		//I2C接口的名称
					         uint8_t Addr, 		//从机地址，靠左
							 uint8_t *pBuffer,	//接收缓冲区
							 uint16_t Size);	//要接收的数据的数量（字节）
```

![image-20250311235722854](./STM32面试考点.assets/image-20250311235722854.png)

### 发送起始位

通过调用I2C_GenerateStart函数发送起始位，并查询SB标志位确认发送完成

### 寻址阶段

1.发送7位从机地址和RW位（读操作为1），等待从机返回ACK。 

2.通过查询AF和ADDR标志位确认寻址是否成功。使用前需要清除AF标志位

<img src="./STM32面试考点.assets/image-20250312105608893.png" alt="image-20250312105608893" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250312110427153.png" alt="image-20250312110427153" style="zoom:50%;" />

### 发送ACK和NAK

<img src="./STM32面试考点.assets/image-20250312111057428.png" alt="image-20250312111057428" style="zoom:67%;" />

ACK这个比特位**==只作用于当前正在被接收的字节==**，也就是这个字节正在被接收，但是还没有被接收完。如果这个字节已经接收完了，它的ACK或者NAK已经发送出去了，再去设置这个比特位，就没有作用了

<img src="./STM32面试考点.assets/image-20250312111248089.png" alt="image-20250312111248089" style="zoom:60%;" />

```c
I2C_AcknowledgeConfig(I2Cx, ENABLE);
```

表示 I2C 硬件使能后(ACK=1)在接收每个字节后自动发送 ACK 信号

### 发送停止位

通过设置stop比特位为1来发送停止位

## 数据接收代码编写(3种情况)

[I2C完整代码](##I2C完整代码)

### 1个字节size = 1

<img src="./STM32面试考点.assets/image-20250312113053589.png" alt="image-20250312113053589" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250312113248004.png" alt="image-20250312113248004" style="zoom:60%;" />

### 2个字节size = 2

<img src="./STM32面试考点.assets/image-20250312142816201.png" alt="image-20250312142816201" style="zoom:67%;" />

### 多个字节size > 2

<img src="./STM32面试考点.assets/image-20250312145603556.png" alt="image-20250312145603556" style="zoom:67%;" />

## 测试代码和实验现象

<img src="./STM32面试考点.assets/image-20250312154846993.png" alt="image-20250312154846993" style="zoom:67%;" />

# 4.6_[I2C]软I2C

<img src="./STM32面试考点.assets/image-20250310210606016.png" alt="image-20250310210606016" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310210638677.png" alt="image-20250310210638677" style="zoom:60%;" />

## 软件I2C的使用方法

1.软件iI2C的定义：通过软件模拟硬件iPhone c的功能。 

2.使用原因：单片机硬件I2C操作繁琐且引脚位置受限。 

3.原理：根据I2C数据帧格式，使用普通IO引脚模拟SCL和SDA波形。

参数设置：设置为通用输出开漏模式，初始化为高电压。

<img src="./STM32面试考点.assets/image-20250312163321170.png" alt="image-20250312163321170" style="zoom: 60%;" />

## 编程接口的实现

### 1.IO引脚初始化

### 2.IO读写和延迟函数

```c
void scl_write(uint8_t level); // 向sCL写0或者写1

void sda_write(uint8_t level); //向SDA写0或者写1

uint8_t sda_read(void);    // 读取SDA的值

void delay_us(uint32_t us) // 微秒级延迟
{
	uint32_t n = us * 8;
	for(uint32_t i=0; i<n; i++); //对于当前配置单片机来说，每执行1次for循环消耗1/8 us
}
```

### 3.起始位和停止位

一定要确保这个SCL是低电压的时候，才向SDA去写值

我们假设如果这个SCL是高电压的时候，去改变这个SDA的值，那这个时候SCL高电压一个下降沿，这就等于发了一个起始位了

<img src="./STM32面试考点.assets/image-20250312185252872.png" alt="image-20250312185252872" style="zoom:60%;" />

### 4.发送1个字节

<img src="./STM32面试考点.assets/image-20250312191658947.png" alt="image-20250312191658947" style="zoom:67%;" />

发送字节。读取ACK或NAK

<img src="./STM32面试考点.assets/image-20250312193231111.png" alt="image-20250312193231111" style="zoom: 67%;" />

计算当前比特位是0还是1，x的计算，与0 或1，

<img src="./STM32面试考点.assets/image-20250312193658749.png" alt="image-20250312193658749" style="zoom: 60%;" />

### 5.接收1个字节

<img src="./STM32面试考点.assets/image-20250312201615013.png" alt="image-20250312201615013" style="zoom:67%;" />

ACK=0时，回NAK；ACK=1时，回ACK

<img src="./STM32面试考点.assets/image-20250312204438133.png" alt="image-20250312204438133" style="zoom:67%;" />

## ==I2C软完整代码==

```c
#include "stm32f10x.h"

void My_SI2C_Init(void);
void scl_write(uint8_t level); 
void sda_write(uint8_t level); 
uint8_t sda_read(void); 
void delay_us(uint32_t us);
void SendStart(void);
void SendStop(void);
uint8_t send_byte(uint8_t byte);
uint8_t receive_byte(uint8_t Ack);

int My_SI2c_sendBytes(uint8_t Addr, uint8_t*pData, uint16_t Size);
int My_SI2C_ReceiveBytes(uint8_t Addr, uint8_t *pBuffer, uint16_t Size);

int main(void)
{
    My_SI2C_Init();
    
    uint8_t commands[] = {0x00, 0x8d, 0x14, 0xaf, 0xa5};
    
    My_SI2c_SendBytes(0x78, commands,5);
    
    while(1)
    {
    }
}

void My_SI2C_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
    
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
    
	GPIO_Init (GPIOA, &GPIO_InitStruct);
    
    //初始电压拉高
    GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
	GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);
}

void scl_write(uint8_t level);
{
	//PA0
	if (level == 0)
    {
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
	}
	else
    {
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
    }
}

void sda_write(uint8_t level);
{
	//PA1
	if (level == 0)
    {
		GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_RESET);
	}
	else
    {
		GPIO_WriteBit(GPIOA, GPIO_Pin_1, Bit_SET);
    }    
}

uint8_t sda_read(void);
{
    if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_l) == Bit_SET)
    {
        return 1;
    }
	else
    {
        return 0;
    }
}

void delay_us(uint32_t us)
{
    uint32_t n = us * 8;
	for(uint32_t i=0; i<n; i++);
}

void SendStart(void)
{
    sda_write(0);
    delay_us(1);
}

void SendStop(void)
{
    scl_write(0);
	sda_write(0);
	delay(1);
	scl_wirte(1);
	delay(1);
	sda_write(1);
	delay(1);
}

uint8_t send_byte(uint8_t Byte);
{
    for(int8_t i=7; i>=0; i++)
    {	
    	scl_write(0);
        if((Byte & (0x01 << i)) != 0)
        {
            sda_write(1);
        }
		else
        {
            sda_write(0)
        }		
		delay_us(1);
		scl_wirte(1);
		delay_us(1);
    }
    
    // 读取ACK或NAK
    scl_write(0);
	sda_write(1);
	delay_us(1);
	scl_write(1);
	delay_us(1);
	return sda_read();
}

uint8_t receive_byte(uint8_t Ack)
{
	uint8_t byte = 0;
    
	for(int8_t i=7; i>=0; i--)
    {
		scl_write(0); 
		sda_write(1); 
    	delay_us(1);
		scl_write(1); 
    	delay_us(1);
		if(sda_read() != 0) //初始byte是低电压
        {
           byte |= 0x01 << i; 
        }
			
	//回复ACK或NAK，ACK=0，NAK=1
	scl_write(0); 
    sda_write(!Ack);
    delay_us(1);
	scl_write(1); 
    delay_us(1); 
        
    return byte;
}
    
int MyLSI2c_sendBytes(uint8_t Addr, uint8_t*pData, uint16_t Size)
{
    SendStart();
	if(SendByte(Addr & 0xfe) != 0)
    {
       SendStop();
		return -1; 
    }
	
    for(uint32_t i=0; i<Size; i++)
    {
		if(SendByte(pData[i]) != 0)
    	{
			SendStop();
			return -2;
    	}
    }
    
    SendStop();
    return 0;
}
    
int My_SI2C_ReceiveBytes(uint8_t Addr, uint8_t *pBuffer, uint16_t Size)
{
	SendStart();
	if(SendByte(Addr | 0x01) != 0)
    {
    	SendStop();
		return -1; 
    }
	
    for(uint32_t i=0; i<Size-1; i++)
    {
		pBuffer[i] = ReceiveByte(1);
    }
    
    pBuffer[Size-1] = ReceiveByte(0);
    
    SendStop();
    return 0;
}
```

## 综合编程接口的实现

[I2C完整代码](##I2C软完整代码)

软I2C写

```c
int MyLSI2c_sendBytes(uint8_t Addr,   // 从机地址,靠左
					  uint8_t*pData,  // 要发送的数据
					  uint16_t Size); // 要发送的数据的数量（字节）
```

软I2C读

```c
int My_SI2C_ReceiveBytes(uint8_t Addr,     // 从机地址,靠左
						 uint8_t *pBuffer, // 接收缓冲区
                         uint16_t Size);   // 发送的数据的数量（字节）
```

![image-20250312213436964](./STM32面试考点.assets/image-20250312213436964.png)

# 4.7_[I2C]封装常用功能

<img src="./STM32面试考点.assets/image-20250310210710815.png" alt="image-20250310210710815" style="zoom:60%;" />

## 自带编程接口

<img src="./STM32面试考点.assets/image-20250312222728752.png" alt="image-20250312222728752" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250312224055446.png" alt="image-20250312224055446" style="zoom:67%;" />

# 4.8_[I2C]OLED显示器

<img src="./STM32面试考点.assets/image-20250310210751191.png" alt="image-20250310210751191" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310210855619.png" alt="image-20250310210855619" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310210911282.png" alt="image-20250310210911282" style="zoom:60%;" />

# ——————————————

# 5.1_[SPI]电路结构和通信协议

<img src="./STM32面试考点.assets/image-20250310211835406.png" alt="image-20250310211835406" style="zoom:60%;" />

## SPI总线的电路结构

SPI总线由主机（master）和多个从机（slave）组成。

SPI总线的主要引脚包括

- MOSI，主机发送数据到从机的引脚
- MISO，从机发送数据到主机的引脚
- SCK，串行时钟引脚，用于控制通信速度；主机产生时钟信号，从机接收时钟信号
- NSS，低电压有效的从机选择引脚，发送低电压来选择与之通信的从机

<img src="./STM32面试考点.assets/image-20250317104850620.png" alt="image-20250317104850620" style="zoom:67%;" />

## SPI总线的分类

### 极性

低极性时，空闲状态下时钟信号为低电压。高极性时，空闲状态下时钟信号为高电压。

<img src="./STM32面试考点.assets/image-20250317105725771.png" alt="image-20250317105725771" style="zoom:50%;" />

### 相位

第一边沿采集在上升沿采集数据，第二边沿采集在下降沿采集数据 

据这个采集数据不同的时间点，可以划分出这两这么两种相位

<img src="./STM32面试考点.assets/image-20250317110027827.png" alt="image-20250317110027827" style="zoom:67%;" />

### 四种模式

<img src="./STM32面试考点.assets/image-20250317110207040.png" alt="image-20250317110207040" style="zoom:60%;" />

## 比特位的传输顺序

1.LSB First：list significant bit，先传输最低有效位。 

2.MSB First：most significant bit first，先传输最高有效位。

 <img src="./STM32面试考点.assets/image-20250317110435832.png" alt="image-20250317110435832" style="zoom:67%;" />

## 数据的宽度

1. 8比特：每次传输8个比特位，即一个字节的数据。 

2. 16比特：每次传输16个比特位。
3. <img src="./STM32面试考点.assets/image-20250317110721184.png" alt="image-20250317110721184" style="zoom:67%;" />

# 5.2_[番外]按钮驱动程序编写

<img src="./STM32面试考点.assets/image-20250310211905765.png" alt="image-20250310211905765" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310212022817.png" alt="image-20250310212022817" style="zoom:60%;" />

## 基本原理

### 按钮

IO引脚设置成输入上拉模式，也就是使能这里的上拉电阻，然后当这个按钮松开，IO引脚是悬空的，它会在这个上拉电阻的作用下，这一点的电压呢被拉高到高电压3.3V。因此此时我们读到的这个值1。

如果我们把这个按钮按下的话，这两个点就导通了，通过按钮直接接地。电压就是0V，所以我们在这个输入数据寄存器上读到的值就是0，所以按钮按下的时候读到的是0，按钮松开的时候读到的是1。

<img src="./STM32面试考点.assets/image-20250317144628353.png" alt="image-20250317144628353" style="zoom:40%;" />

### 板载LED

这个板载LED，用的是标准的开漏接法，板载LED的阳极呢，连接的是高电压3.3V。阴极通过一个限流电阻接到单片机的PC13，所以应该把这个PC13设置成输出开漏模式。当我们向这个PC13写0的时候，这个引脚输出的是低电压，相当于这个LED的阴极接的是地就会有电流从上面流过LED就点亮了。

<img src="./STM32面试考点.assets/image-20250317114149925.png" alt="image-20250317114149925" style="zoom: 67%;" />

## 初始化IO引脚

```c
void App_Button_Init(void)	//按钮初始化
{
	// PA0 
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	
	GPIO_Init(GPIOA, &GPIO_InitStruct);
}

void App_OnBoardLED_Init(void)	//LED初始化
{
	// PC13
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	
	GPIO_Init(GPIOC, &GPIO_InitStruct);
}
```



## 按钮驱动程序原理

通过读取IO引脚的状态变化来捕捉按钮状态。使用while循环和变量记录上次和当前的状态。

<img src="./STM32面试考点.assets/image-20250317150439403.png" alt="image-20250317150439403" style="zoom:65%;" />

```c
int main(void)
{
	App_Button_Init(); // 按钮初始化
	App_OnBoardLED_Init(); // LED初始化
	
	uint8_t current = Bit_SET, previous = Bit_SET;
	
	while(1)
	{
		previous = current; // 保存上次按钮的状态		
		current  = GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_0);
		
		if(current != previous)
		{
			if(current == Bit_SET) // 按钮松开
			{
				// 改变LED的亮灭状态
				if(GPIO_ReadOutputDataBit(GPIOC, GPIO_Pin_13) == Bit_SET) // 写1
				{
					GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); // 写0
				}
				else // 按钮按下
				{
					GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
				}
			}
			else
			{
			}
			Delay(10);
		}
	}
}
```

### IO引脚控制这两个moss管的通断来输出不同的电压

```c
void GPIO_WriteBit(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, BitAction BitVal);
作用：向输出数据寄存器写0/1
    
uint8_t GPIO_ReadOutputDataBit(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);
作用：读取输出数据寄存器的一个比特位
```



```c
//读取输出数据寄存器的值
if(GPIO_ReadOutputDataBit(...) == Bit_SET) // 如果是1
	GPIO_WriteBit(..., Bit RESET); // 写0
else // 如果是0
	GPIO_WriteBit(.., Bit_SET); // 写1
```



### 软件消抖方法

过载片形变，两个电极之间导通关系。所以当我们按下或者是松开这个按钮的瞬间，这个过载片并不能立刻稳定下来，它就会出现类似于这样的波形

<img src="./STM32面试考点.assets/image-20250317153858824.png" alt="image-20250317153858824" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250317154108496.png" alt="image-20250317154108496" style="zoom: 67%;" />

# 5.3_[番外]按钮代码的封装

<img src="./STM32面试考点.assets/image-20250310212052827.png" alt="image-20250310212052827" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310212125792.png" alt="image-20250310212125792" style="zoom:60%;" />

实验现象包括单击增加、双击清零、长按持续增加。通过回调函数和进程函数实现按钮功能

## 电路搭建和串口初始化

1. TX——>AF_PP模式，复用推挽模式
2. RX——>IPU上拉模式or输入浮空模式
3. usart初始化，数据帧的格式以及波特率的配置
4. 闭合总开关

### 编程接口

```c
void My_Button_Init(Button_TypeDef *Button, Button_InitTypeDef *Button_InistStruct)
作用：初始化按钮（包括了IO引脚的初始化）
    
struct GPIO_InitTypeDef{
	GPIO_TypeDef *GPIOx; // IO引脚的端口号
	uint16_t GPIO_Pin;   // IO引脚的引脚编号
	uint32_t LongPressTime;		// 长按的时间阈值，单位毫秒，0表示默认(1000)
	uint32_t LongPressInterval; // 长按后持续触发的时间间隔，0表示默认(100)
	uint32_t clickInterval;		// 连击的最大时间间隔，0表示默认（200）
	//回调函数，没有的话填0！
	void(*button_pressed_cb)(void);  // 回调函数 - 按钮按下
	void(*button_released_cb)(void); // 回调函数 - 按钮抬起
	void(*button_clicked_cb)(uint8_tclicks); 	 // 回调函数 - 按钮点击
	void(*button_long_pressed_cb)(uint8_tticks); // 回调函数 - 按钮长按
}
```



```c
void My_Button_Proc(Button_TypeDef *Button); // 进程函数
作用：按钮状态进程检测
```

<img src="./STM32面试考点.assets/image-20250319103735067.png" alt="image-20250319103735067" style="zoom: 67%;" />



## 代码设计

```c
#include "stm32f10x.h"
#include "usart.h"
#include "button.h"

Button_TypeDef button1;
uint32_t cnt = 0;

void App_USART1_Init(void);
void App_Button_Init(void);
void button_clicked_cb(uint8_t clicks);
void button_long_pressed_cb(uint8_t ticks);

int main(void)
{
	App_USART1_Init();
//	My_USART_SendString(USART1, "Hello world.\r\n");
	App_Button_Init();
		
	while(1)
	{
		My_Button_Proc(&button1);
	}
}

void App_USART1_Init(void)
{
	// #1. 初始化IO引脚 PA9 Tx P10 Rx
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 初始化USART1
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	USART_InitTypeDef USART_InitStruct;
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_Init(USART1, &USART_InitStruct);
	
	USART_Cmd(USART1, ENABLE);
}

void App_Button_Init(void)
{
	Button_InitTypeDef Button_InitStruct;
	
	Button_InitStruct.GPIOx = GPIOA;
	Button_InitStruct.GPIO_Pin = GPIO_Pin_0;
	Button_InitStruct.ClickInterval = 0;
	Button_InitStruct.LongPressTime = 0;
	Button_InitStruct.LongPressTickInterval = 0;
	Button_InitStruct.button_clicked_cb = button_clicked_cb;
	Button_InitStruct.button_pressed_cb = 0;
	Button_InitStruct.button_released_cb = 0;
	Button_InitStruct.button_long_pressed_cb = button_long_pressed_cb;
	
	My_Button_Init(&button1, &Button_InitStruct);
}

void button_clicked_cb(uint8_t clicks)
{
	if(clicks == 1)
	{
		cnt++;
		My_USART_Printf(USART1, "%d", cnt);
	}
	else if(clicks == 2)
	{
		cnt = 0;
		My_USART_Printf(USART1, "%d", cnt);
	}
}

void button_long_pressed_cb(uint8_t ticks)
{
	cnt++;
	My_USART_Printf(USART1, "%d", cnt);
}

```



# 5.4_[SPI]IO引脚初始化

<img src="./STM32面试考点.assets/image-20250310212300669.png" alt="image-20250310212300669" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310212340148.png" alt="image-20250310212340148" style="zoom:60%;" />

## SPI模块介绍

SPI模块是单片机上的串行外设接口，用于与外部设备进行通信

<img src="./STM32面试考点.assets/image-20250319155127563.png" alt="image-20250319155127563" style="zoom:67%;" />

## W25Q64模块介绍

1. W25Q64是一种Flash芯片，类似于电脑的硬盘，可以存储数据且掉电后数据不丢失。 

2. 单片机内部的Flash用于存储程序，而W25Q64用于存储单独的数据，类似于移动硬盘。 

3. W25Q64使用SPI接口与单片机进行通信。

<img src="./STM32面试考点.assets/image-20250319155239813.png" alt="image-20250319155239813" style="zoom:67%;" />

### W25Q64引脚定义

VCC和GND用于接电源，其中VCC接3.3伏，GND接地。 

DI是数据输入引脚，连接主机的MOSI。DO是数据输出引脚，连接主机的MISO。 

CLK是时钟引脚，连接主机的SCK

<img src="./STM32面试考点.assets/image-20250319155405375.png" alt="image-20250319155405375" style="zoom:60%;" />

## SPI引脚位置定位

nss引脚，它是从机的片选引脚，所以如果我们使用的是主机模式的话，这个引脚就用不到

<img src="./STM32面试考点.assets/image-20250319155725209.png" alt="image-20250319155725209" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250319155952716.png" alt="image-20250319155952716" style="zoom: 67%;" />

## IO引脚模式选择

![image-20250319161939087](./STM32面试考点.assets/image-20250319161939087.png)

## IO最大输入速度选择

选择满足要求的最小值，单片机提供三档最大输入速度：2兆、10兆和50兆

<img src="./STM32面试考点.assets/image-20250319162619705.png" alt="image-20250319162619705" style="zoom: 60%;" />

## ==SPI初始化完整代码==

<img src="./STM32面试考点.assets/image-20250319162655514.png" alt="image-20250319162655514" style="zoom: 50%;" />

```c
#include "stm32f10x.h"

void App_SPI1_Init(void);

int main(void)
{
	App_SPI1_Init();
	
	while(1)
	{
	}
}

void App_SPI1_Init(void)
{
	// #1. 初始化IO引脚
	
	// PB3 SCK AF_PP 2MHz
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_3;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	// PB4 MISO IPU 
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_4;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	// PB5 MOSI AF_PP 2MHz
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_5;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOB, &GPIO_InitStruct);
	
	// PA15 OUT_PP 2MHz
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_15;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
    
    // #2. 对SPI本身进行初始化
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);
	
	SPI_InitTypeDef SPI_InitStruct;
	
	SPI_InitStruct.SPI_Mode = SPI_Mode_Master;
	SPI_InitStruct.SPI_Direction = SPI_Direction_2Lines_FullDuplex;
	SPI_InitStruct.SPI_DataSize = SPI_DataSize_8b;
	SPI_InitStruct.SPI_CPOL = SPI_CPOL_High;
	SPI_InitStruct.SPI_CPHA = SPI_CPHA_2Edge;
	SPI_InitStruct.SPI_FirstBit = SPI_FirstBit_MSB;
	SPI_InitStruct.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_64;
	SPI_InitStruct.SPI_NSS = SPI_NSS_Soft; 
	
	SPI_Init(SPI1, &SPI_InitStruct);
	
	SPI_NSSInternalSoftwareConfig(SPI1, SPI_NSSInternalSoft_Set);
}
```



# 5.5_[SPI]SPI模块的初始化

<img src="./STM32面试考点.assets/image-20250310212433223.png" alt="image-20250310212433223" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310212523805.png" alt="image-20250310212523805" style="zoom:60%;" />

## SPI模块的内部结构框图

<img src="./STM32面试考点.assets/image-20250319193050439.png" alt="image-20250319193050439" style="zoom: 75%;" />

SPI通信的四种方向：两线全双工、两线只读、单线发送和单线接收

粉色框框是SPI的标志位，通过查询获得SPI的工作状态

TXE是发送数据寄存器空

RxNE是接入数据寄存器非空

## 编程接口

[SPI初始化完整代码](##SPI初始化完整代码)

 ```c
 void SPI_Init(SPI_TypeDef*SPIx,  // SPI的名称，可以是SPI1或SPI2
 SPI_InitTypeDef*SPI_InitStruct); // 用于传递初始化参数
 作用：对SPI模块进行初始化
     
 structSPI_InitTypeDef{
 	uint16_t SPI_Direction; // 用来选择SPI通信的方向
 	uint16_t SPI_Mode; 		// 用来选择SPI的模式，SPI_Master - 主机，SPI_Slave - 从机
 	uint16_t SPI_DataSize; 	// 数据宽度，SPI_DataSize_8b - 8bit，SPI_DataSize_16b - 16bit
 	uint16_t SPI_CPOL; 		// 时钟的极性
 	uint16_t SPI_CPHA; 		// 时钟的相位
 	uint16_t SPI_NSS; 		// 软件NSS/硬件NSS
 	uint16_t SPI_BaudRatePrescaler; // 用来选择波特率分频器的分频系数
 	uint16_t SPI_FirstBit; 	// 比特位的传输顺序，SPI_FirstBit_MSB，SPI_FirstBit_LSB，
 }
 ```

SPI要传输一个字节，由八个比特位组成的，从bit7到bit0。bit0的权重是2^0，它就是权重最小的叫做LSB。然后bit7，它代表2^&，它的权重是最大的就是MSB。决定数据传输方向。

### 通信方向选择

```c
structSPI_InitTypeDef{
...
	uint16_t SPI_Direction; // 通信的方向 -SPI_Direction_2Lines_FullDuplex  2线全双工
							// 			-SPI_Direction_2Lines_ReadOnly    2线只读
							// 			-SPI_Direction_1Line_Rx           单线接收
							// 			-SPI_Direction_1Line_Tx           单线发送
...
}
```

#### 2线全双工

全双工就是通讯接口既能发送数据，也能接收数据，并且这个数据的收发是可以同时进行的

#### 2线只读

对于从机来说，它只通过这个mosi接收主机发来的数据，但是同时不会通过这个miso向主机返回数据，所以它是只是读取数据，所以叫做两线只读

一般用于主机像从机广播，广播的时候这个主机指向从机发送数据，而不要求这个从机返回数据

#### 单线接收/单线发送

主机的mosi连接从机的miso，两个方向不能同时进行的

### 数据宽度、极性、相位和比特位传输顺序

**数据宽度**：就是每次通过这个SPI总线传输的数据位的一个数量。16个比特位其实就是代表我们编程里面的一个半字。

<img src="./STM32面试考点.assets/image-20250319210349187.png" alt="image-20250319210349187" style="zoom:67%;" />

### 极性和相位

极性：空闲状态下，时钟信号上是高电压or低电压

相位：数据采集是在第一边沿-上升沿or第二边沿-上升沿

<img src="./STM32面试考点.assets/image-20250319210906252.png" alt="image-20250319210906252" style="zoom: 67%;" />

### 比特位的传输顺序

<img src="./STM32面试考点.assets/image-20250319212633834.png" alt="image-20250319212633834" style="zoom: 67%;" />

## W25Q64配置

<img src="./STM32面试考点.assets/image-20250319212752566.png" alt="image-20250319212752566" style="zoom: 80%;" />

<img src="./STM32面试考点.assets/image-20250319213037188.png" alt="image-20250319213037188" style="zoom: 67%;" />

[SPI初始化完整代码](##SPI初始化完整代码)

#### 波特率设置

它的最高的这个时钟频率在80M赫兹，这个频率呢是非常高的了，但是我们是用面包板进行连接的。这个面包板的电路呢没有PCB板那么稳定，所以选一个比较低的波特率来看一下实验现象就可以了。比如1M赫兹左右

<img src="./STM32面试考点.assets/image-20250319213840396.png" alt="image-20250319213840396" style="zoom:50%;" />

#### 分屏器

<img src="./STM32面试考点.assets/image-20250319214342074.png" alt="image-20250319214342074" style="zoom: 55%;" />

## NSS引脚配置

### 一主多从

nss引脚外接一个高电压，其实这个原因是对于我们的这个SPI总线来说，它支持一种叫做多主机的模式，我们把这个SPI作为主机的时候，我们也要去适配这个多主机的模式

### 多主机模式

只要这个nss引脚输入了一个低电压。那么它的这个主机身份就丢失了

<img src="./STM32面试考点.assets/image-20250319215058345.png" alt="image-20250319215058345" style="zoom:67%;" />

软件nss是一个内部的nss，这个内部的nss呢，它是一个比特位，我们可以向这个比特位写0，或者是写1。我们写0的时候，等效的输入的就是低电压，而写1的时候呢，等效输入的就是高电压

![image-20250319215833746](./STM32面试考点.assets/image-20250319215833746.png)

```c
SPI_NSSInternalSoftwareConfig(SPI1, SPI_NSSInternalSoft_Set); 
//第一个参数,填SPI的名称 第二个参数set就表示1,reset表示
```



# 5.6_[SPI]数据收发

<img src="./STM32面试考点.assets/image-20250310214242238.png" alt="image-20250310214242238" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310214252543.png" alt="image-20250310214252543" style="zoom:60%;" />

## SPI总线数据收发特点

SPI总线由mosi（主发重收）、miso（主收重发）、sCK（串行时钟）和nss（从机片选信号）组成

SPI数据收发必须是双向的，主机每发送一个比特位给从机，必然会从从机收到一个比特位

<img src="./STM32面试考点.assets/image-20250320113523148.png" alt="image-20250320113523148" style="zoom:67%;" />

## 编程接口

```c
void App_SPI_MasterTransmitReceive(SPITypeDef*SPIx, 	// SPI的名称
								const uint8_t *pDataTx, // 要发送的数据
									  uint8_t *pDataRx, // 接收到的数据
									  uint16_t Size);   // 收发数据的数量
作用：使用SPI总线收发数据
```

## 数据收发过程

1.闭合SPI总开关，开始SPI通信。 

2.提前写入第一个字节数据到发送数据寄存器。 

3.进入循环，每次循环发送一个字节，接收一个字节。 

- 发送数据之前一定要查一下这个TXE标志位，只有这个标志位等于发送数据寄存器才是空，才能把这个新的数据写进去。
- 接收数据必须等待这个RxNE=1，此时接收数据寄存器里边才有值，才能通过这个程序把它读出来。

4.循环执行size-1次，每次循环发送和接收一个字节。 

5.最后接收最后一个字节，断开SPI总开关。

<img src="./STM32面试考点.assets/image-20250320114437094.png" alt="image-20250320114437094" style="zoom: 67%;" />

## SPI收发代码

I2S是一个数字音频的接口，是用来传声音的。这个接口可以用来操作I2S总线，也可以操作SPI总线

大家注意！我们发送字节的时候发送的是第 i+1个字节，然后接收的时候接收的是第 i 个字节

```c
void App_SPI_MasterTransmitReceive(SPI_TypeDef *SPIx,
const uint8_t *pDataTx, uint8_t *pDataRx, uint16_t Size)
{
	SPI_Cmd(SPIx, ENABLE); // #1. 闭合总开关
	SPI_I2S_SendData(SPIx, pDataTx[0]); // #2. 发送第一个字节
    // #3.
	for(uint16_t i=0; i<Size-1; i++)
	{
		// 向TDR写数据，发送一个字节
		while(SPI_I2S_GetFlagStatus(SPIx, SPI_I2S_FLAG_TXE) == RESET);
		SPI_I2S_SendData(SPIx, pDataTx[i+1]);
		// 向RDR读数据，接收一个字节
		while(SPI_I2S_GetFlagStatus(SPIx, SPI_I2S_FLAG_RXNE) == RESET);
		pDataRx[i] = SPI_I2S_ReceiveData(SPIx);
	}
	// #4. 读出最后一个字节
	while(SPI_I2S_GetFlagStatus(SPIx, SPI_I2S_FLAG_RXNE) == RESET);
	pDataRx[Size-1] = SPI_I2S_ReceiveData(SPIx);
	// #5. 断开总开关
	SPI_Cmd(SPIx, DISABLE);
}
```



# 5.7_[SPI]W25Q64实验(上)

<img src="./STM32面试考点.assets/image-20250310214345871.png" alt="image-20250310214345871" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250310214405577.png" alt="image-20250310214405577" style="zoom:60%;" />

## 代码修改

重映射代码，SPI初始化代码修改模式

### 保存功能编程接口

```c
void App_W25Q64_SaveByte(uint8_t Byte); // 使用W25Q64保存一个字节
uint8_t App_W25Q64_LoadByte(void); 		// 把保存的字节读出来
```

## w25q64存储结构

1.介绍w25q64的存储结构，包括块、扇区和页的定义。 

2.存储容量为8MB，分为128个块，每个块64KB。 

3.每个扇区4KB，可进一步分为16个页，每个页256字节。

<img src="./STM32面试考点.assets/image-20250320151543586.png" alt="image-20250320151543586" style="zoom:67%;" />

## 存储方式

1.擦除操作：以扇区为单位，最小擦除单元为4KB。 

2.编程操作：以页为单位，最多写入256个字节。

3.擦除和编程操作都需要先进行写使能，确保数据安全，打开锁

4.等待空闲：通过读取状态寄存器SR1的BUSY标志位来判断操作是否完成（1执行，0空闲）

<img src="./STM32面试考点.assets/image-20250320151951774.png" alt="image-20250320151951774" style="zoom:67%;" />

### 写使能操作

<img src="./STM32面试考点.assets/image-20250320152253968.png" alt="image-20250320152253968" style="zoom:50%;" />

代码

<img src="./STM32面试考点.assets/image-20250320152617692.png" alt="image-20250320152617692" style="zoom: 40%;" />

```c
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer,1);

//这个SPI1就是我们要使用的单片机上的那个SPI接口就是SPI1。第二个参数呢，代表我们要发送的数据,也就是我们这个0x06。然后第三个参数代表我们接收数据的这么一个缓冲区，也就是说通过SPI总线接收到的这个数据，再给它存到这个buffer 0这个元素里边。最后一个参数代表我们要收发数据的一个数量，我们只需要收发一个字节的数量。
```

### 扇区擦除操作

发送24位地址，表示要擦除的扇区首地址

<img src="./STM32面试考点.assets/image-20250320153353525.png" alt="image-20250320153353525" style="zoom:40%;" />

### 等待空闲

等待空闲：通过循环读取状态寄存器SR1的值，判断BUSY标志位是否为0

<img src="./STM32面试考点.assets/image-20250320153521481.png" alt="image-20250320153521481" style="zoom:50%;" />

我们最好向这个接收缓冲区里面的元素，先赋一个初值0xff，意思就是我们在读取数据的过程中保证MOSI这条线上的电压呢，始终是一个高电压

### 页编程

执行的是页编程，一个页的最大范围就是256个字节，所以我们最多能够发送256个字节

<img src="./STM32面试考点.assets/image-20250320155111830.png" alt="image-20250320155111830" style="zoom:50%;" />

## 读取数据操作

<img src="./STM32面试考点.assets/image-20250320155543360.png" alt="image-20250320155543360" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250320155934532.png" alt="image-20250320155934532" style="zoom: 80%;" />

```c
// 读取一个字节
buffer[0] = 0x03; buffer[1] = 0x00; buffer[2] = 0x00; buffer[3] = 0x00;

GPIO_WriteBit(GPIOA, GPIO_Pin_12, Bit_RESET); // NSS=0
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 4); // 发0x03+24位地址
buffer[0] = 0xff;
App_SPI_MasterTransmitReceive(SPI1, buffer, buffer, 1); // 收一个字节
GPIO_WriteBit(GPIOA, GPIO_Pin_12, Bit_SET); // NSS=1

return buffer[0];
```

## 测试验证

<img src="./STM32面试考点.assets/image-20250320160410935.png" alt="image-20250320160410935" style="zoom:60%;" />



# 5.8_[SPI]W25Q64实验(下)

<img src="./STM32面试考点.assets/image-20250310215626631.png" alt="image-20250310215626631" style="zoom:60%;" />

# ——————————————

# 6.1_[中断]中断的概念

<img src="./STM32面试考点.assets/image-20250311113216208.png" alt="image-20250311113216208" style="zoom:60%;" />

## 中断的概念

中断是单片机应对突发事件的一种方式。 

单片机在常规程序执行过程中，如果发生中断，会暂停常规程序，转而处理中断

### 中断响应函数

当中断发生时，单片机会自动调用中断响应函数。每次中断发生，中断响应函数都会被调用一次。中断响应函数执行完成后，返回常规程序继续执行

<img src="./STM32面试考点.assets/image-20250320161559050.png" alt="image-20250320161559050" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250320163933531.png" alt="image-20250320163933531" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250320165129030.png" alt="image-20250320165129030" style="zoom: 67%;" />

## 合并程序

直接合并闪灯程序和串口数据接收程序会导致数据接收不及时，造成数据丢失。 

时序图展示了闪灯程序和串口数据接收程序的执行过程，说明数据丢失的原因。

<img src="./STM32面试考点.assets/image-20250320170218799.png" alt="image-20250320170218799" style="zoom:67%;" />

## 中断解决数据丢失问题

中断响应函数里面呢，我们首先调用user receive data，把这个数据从接收数据寄存器RDR里边给它读出来，

然后我们根据读到的这个数据呢，对我们的这个闪灯速度进行调节

<img src="./STM32面试考点.assets/image-20250320170425929.png" alt="image-20250320170425929" style="zoom: 80%;" />

# 6.2_[中断]中断优先级

<img src="./STM32面试考点.assets/image-20250311113249066.png" alt="image-20250311113249066" style="zoom:60%;" />

## STM32单片机的中断结构

中断结构框图：包括片上外设、中断时序图和中断向量表。 

- 片上外设：芯片内部的模块，负责执行独立功能。 

- 中断向量表：列出每种中断的中断响应函数

- NVIC：专门负责对这些中断进行管理，它会根据这个每一个中断的优先级来对我们的中断进行排排队，然后排队排在前面的中断优先被响应


<img src="./STM32面试考点.assets/image-20250320171000063.png" alt="image-20250320171000063" style="zoom:67%;" />

### NVIC模块

1.四个比特位表示中断优先级，分为抢占优先级和子优先级。 

2.抢占优先级：决定中断嵌套，数字越小优先级越高。 

3.子优先级：决定中断排队，数字越小优先级越高。

<img src="./STM32面试考点.assets/image-20250320173409698.png" alt="image-20250320173409698" style="zoom: 67%;" />

### 抢占优先级和中断嵌套

1. 中断嵌套：当前正在执行的中断被更高优先级的中断打断

2. 发生条件：新中断的抢占优先级比正在执行的中断的抢占优先级更高

下图是中断1和其他中断比较

<img src="./STM32面试考点.assets/image-20250320173853955.png" alt="image-20250320173853955" style="zoom:67%;" />

###  子优先级和中断排队

<img src="./STM32面试考点.assets/image-20250320174119943.png" alt="image-20250320174119943" style="zoom:50%;" />

## 练习

<img src="./STM32面试考点.assets/image-20250320180558496.png" alt="image-20250320180558496" style="zoom:60%;" />

1—>2—>5—>4—>6—>3

因为3、4、6的抢占优先级相同，需要排队，4、6的优先级更高，所以排队前面。此外4先来，所以4、6排序

# 6.3_[中断]串口中断编程实验

<img src="./STM32面试考点.assets/image-20250311113331073.png" alt="image-20250311113331073" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311113342303.png" alt="image-20250311113342303" style="zoom:60%;" />

## 闪灯代码编写

```c
LED灯初始化
    
void App_OnBoardLED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	
	GPIO_Init(GPIOC, &GPIO_InitStruct);
}
```

<img src="./STM32面试考点.assets/image-20250320202851005.png" alt="image-20250320202851005" style="zoom: 67%;" />

打开监视窗口

<img src="./STM32面试考点.assets/image-20250320203300410.png" alt="image-20250320203300410" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250320203311107.png" alt="image-20250320203311107" style="zoom: 67%;" />

改变闪烁间隔时间，测试

<img src="./STM32面试考点.assets/image-20250320203355240.png" alt="image-20250320203355240" style="zoom:67%;" />

## 串口初始化

初始化IO引脚：PA9初始化为复用推挽模式，PA10初始化为输入上拉模式。

初始化USART模块：使能USART1时钟，设置波特率、数据帧格式、硬件流控、通信方向等参数

<img src="./STM32面试考点.assets/image-20250320203606198.png" alt="image-20250320203606198" style="zoom:60%;" />

```c
void App_USART1_Init(void)
{
	// #1. 初始化IO引脚
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	// PA9 AF_PP
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// PA10 IPU
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 初始化USART1
	
	// 开启USART1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	// 初始化USART1
	USART_InitTypeDef USART_InitStruct;
	
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	
	USART_Init(USART1, &USART_InitStruct);
	
	// 闭合总开关
	USART_Cmd(USART1, ENABLE);
	
	// #3. 配置中断
	USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);
	
	// #4. 配置NVIC
	NVIC_InitTypeDef NVIC_InitStruct;
	
	NVIC_InitStruct.NVIC_IRQChannel = USART1_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 0; 
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 0;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	
	NVIC_Init(&NVIC_InitStruct);
}
```

## 中断配置

1.配置RxNE标志位：使能RxNE标志位触发USART全局中断。 

2.配置NVIC模块：设置中断优先级分组，使能USART1全局中断。是属于内核的一部分，不是片上外设

- 首先第一条就是去配置中断优先级分组，也就是把多少个比特位留给这个抢占优先级，把多少个比特位留给这里的子优先级。
- 第二条要配置的就是往这个四个比特位里面填入这个中断的一个中断优先级，这个中断优先级呢，就代表了这个中断的一个紧急程度。
- 那第三条我们要做的事情呢，就是闭合这里的开关，只有闭合这个开关之后，我们这个中断呢，才能被使能

3.图片中月牙型黑块 是 或逻辑门

```c
配置USART模块中断
    
void USART_ITConfig(USART_TypeDef *USARTx, // 串口的名称，USART1, USART2, …
						uint16_t USART_IT, // 标志位的名称
										   // USART_IT_TXE, USART_IT_TC, USART_IT_RxNE
										   // USART_IT_PE, USART_IT_ERR
				FunctionalState NewState); // 开关状态, ENABLE - 闭合   DISABLE - 断开
```

<img src="./STM32面试考点.assets/image-20250320205122009.png" alt="image-20250320205122009" style="zoom: 80%;" />

```c
配置中断优先级的分组
    
void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)
```

<img src="./STM32面试考点.assets/image-20250320205937620.png" alt="image-20250320205937620" style="zoom: 67%;" />

<img src="./STM32面试考点.assets/image-20250320210053980.png" alt="image-20250320210053980" style="zoom:67%;" />

## 中断响应函数编写

<img src="./STM32面试考点.assets/image-20250320210729156.png" alt="image-20250320210729156" style="zoom: 80%;" />

```c
void USART1_IRQHandler(void){ // 中断响应函数
	// 判断中断的产生原因
	if(USART_GetFlagStatus(USART1, USART_FLAG_RXNE) == SET){
		uint8_t byte = USART_ReceiveData(…);  // 读取数据，清除标志位
		if(byte == ‘0’) blinkInterval = 1000; // 慢
		if(byte == ‘1’) blinkInterval = 200;  // 中
		if(byte == ‘2’) blinkInterval = 50;   // 快
	}
}
```

<img src="./STM32面试考点.assets/image-20250320210826972.png" alt="image-20250320210826972" style="zoom:60%;" />



# ——————————————

# 7.1_[EXTI]工作原理

<img src="./STM32面试考点.assets/image-20250311113423601.png" alt="image-20250311113423601" style="zoom:60%;" />

### EXTI模块介绍

EXTI (External Interrupt and Event Controller) 外部中断和事件控制器

EXTI模块用于捕捉输入信号的变化并产生中断

<img src="./STM32面试考点.assets/image-20250320214854938.png" alt="image-20250320214854938" style="zoom:67%;" />

工作原理：EXTI模块通过捕捉输入信号的变化产生中断

## 应用示例

1.通过示例讲解了如何使用EXTI模块捕捉按钮动作瞬间，并控制板载LED的亮灭状态。 

2.示例中使用了while循环来不断检查按钮状态，但引入EXTI模块后，程序更加简洁。

<img src="./STM32面试考点.assets/image-20250320215423918.png" alt="image-20250320215423918" style="zoom:67%;" />

## EXTI线概念

1.EXTI线是EXTI模块的基本单位，每条线可以独立捕捉上升沿或下降沿。 

2.EXTI模块内部有20条线，可以同时捕捉20路输入信号的变化。16+4，只研究其中16条

<img src="./STM32面试考点.assets/image-20250320220346668.png" alt="image-20250320220346668" style="zoom: 67%;" />

## EXTI线内部结构

<img src="./STM32面试考点.assets/image-20250320220523422.png" alt="image-20250320220523422" style="zoom: 67%;" />

复用器用于选择IO引脚（结构图左边），边缘检测电路用于捕捉上升沿，或下降沿，或有变化，中断产生逻辑用于触发中断

分为硬件触发 and 软件触发（少用）

<img src="./STM32面试考点.assets/image-20250320220855397.png" alt="image-20250320220855397" style="zoom: 67%;" />

# 7.2_[EXTI]按钮实验

<img src="./STM32面试考点.assets/image-20250311113457002.png" alt="image-20250311113457002" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311113518954.png" alt="image-20250311113518954" style="zoom:60%;" />

## 电路搭建

<img src="./STM32面试考点.assets/image-20250323162018815.png" alt="image-20250323162018815" style="zoom:67%;" />

## ==器件初始化总代码==

```c
#include "stm32f10x.h"

void App_OnBoardLED_Init(void);
void App_Button_Init(void);

int main(void)
{
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	
	App_OnBoardLED_Init();
	App_Button_Init();
	
	while(1)
	{
	}
}

void App_OnBoardLED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOC, &GPIO_InitStruct);
	
	GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
}

void App_Button_Init(void)
{
	// #1. 初始化PA5和PA6
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	// PA5
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_5;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// PA6
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 为EXTI5和EXIT6分配引脚
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOA, GPIO_PinSource5);
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOA, GPIO_PinSource6);
	
	// #3. 初始化EXTI的线
	EXTI_InitTypeDef EXTI_InitStruct;
	
	EXTI_InitStruct.EXTI_Line = EXTI_Line5;
	EXTI_InitStruct.EXTI_Mode = EXTI_Mode_Interrupt;
	EXTI_InitStruct.EXTI_Trigger = EXTI_Trigger_Rising;
	EXTI_InitStruct.EXTI_LineCmd = ENABLE;
	EXTI_Init(&EXTI_InitStruct);
	
	EXTI_InitStruct.EXTI_Line = EXTI_Line6;
	EXTI_InitStruct.EXTI_Mode = EXTI_Mode_Interrupt;
	EXTI_InitStruct.EXTI_Trigger = EXTI_Trigger_Rising;
	EXTI_InitStruct.EXTI_LineCmd = ENABLE;
	EXTI_Init(&EXTI_InitStruct);
	
	// #4. 配置中断
	NVIC_InitTypeDef NVIC_InitStruct;
	
	NVIC_InitStruct.NVIC_IRQChannel = EXTI9_5_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 0;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 0;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	
	NVIC_Init(&NVIC_InitStruct);
}

```

## ESTI线路分配

配置复用器，选择IO引脚

### 编程接口

<img src="./STM32面试考点.assets/image-20250323164302670.png" alt="image-20250323164302670" style="zoom:67%;" />

## EXTI线路参数配置

配置ESTI线路的复用器、捕获边缘类型选择（上升沿or下降沿or双边沿）、开关

<img src="./STM32面试考点.assets/image-20250323164654378.png" alt="image-20250323164654378" style="zoom:70%;" />

### 编程接口

```c
void EXTI_Init(EXTI_InitTypeDef*EXTI_InitStruct); // 用来初始化EXTI的一条线

structEXTI_InitTypeDef{
	uint32_t EXTI_Line; 		     // 线编号 EXTI_Line0~EXTI_Line19，本次实验接到PA5和PA6
	EXTIMode_TpeDef EXIT_Mode; 	     /* 选择模式EXTI_Mode_Interrupt-中断
											   EXTI_Mode_Event-事件 */
	EXTITrigger_TypeDef EXTI_Trigger; /* 边沿 EXTI_Trigger_Rising-上升沿
											EXTI_Trigger_Falling-下降沿
											EXTI_Trigger_Rising_Falling-双边沿	*/
	FunctionalState EXTI_LineCmd; 	 // 开关 ENABLE -闭合，DISABLE -断开
}
```

捕捉按钮抬起的瞬间，所以设置为上升沿

<img src="./STM32面试考点.assets/image-20250323165955717.png" alt="image-20250323165955717" style="zoom:67%;" />

## NVIC模块配置

<img src="./STM32面试考点.assets/image-20250323172616086.png" alt="image-20250323172616086" style="zoom: 67%;" />

1.配置NVIC中断优先级分组；2.设置中断优先级；3.闭合中断开关，启用中断

中断名称

<img src="./STM32面试考点.assets/image-20250323173128677.png" alt="image-20250323173128677" style="zoom:67%;" />

头文件寻找一个枚举类型，找相应的这个EXIT9_5中断的一个名字

<img src="./STM32面试考点.assets/image-20250323173210004.png" alt="image-20250323173210004" style="zoom: 67%;" />

<img src="./STM32面试考点.assets/image-20250323173313132.png" alt="image-20250323173313132" style="zoom:67%;" />

## 中断响应函数编写

### 1.确定中断响应函数的名称

中断向量表 Vector Table Mapped to Address 0 at Reset

<img src="./STM32面试考点.assets/image-20250323193848214.png" alt="image-20250323193848214" style="zoom:67%;" />

### 2.判断哪条线路触发了中断

可以通过判断这个标志位的值来确定这个中断到底是哪条线触发的。标志位的值等于0的时候，这个函数返回的就是reset，如果这个标志位触发值等于1的时候，那么返回的就是set

<img src="./STM32面试考点.assets/image-20250323194122095.png" alt="image-20250323194122095" style="zoom:50%;" />

#### 编程接口

```c
FlagStatusEXTI_GetFlagStatus(uint32_t EXTI_Line); // 获取EXTI线的标志位的值

void EXTI9_5_IRQHandler(void){
	if(EXTI_GetFlagStatus(EXTI_Line5) == SET) // 线5触发的中断
	GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); // 亮灯
    
	if(EXTI_GetFlagStatus(EXTI_Line6) == SET) // 线6触发的中断
	GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET); // 灭灯
}
```

这里判断不能写else，因为因为这个线5和线6可能同时触发中断

### 3.清除标志位

因为如果我们不写这个0的话，那这个比特位始终就是1，那这个中断就会一直被触发，所以我们在使用完这个中断之后，一定要对它进行一个写0的操作

#### 编程接口

```c
void EXTI_ClearFlag(uint32_t EXTI_Line); // 清除EXTI线的标志位
```

<img src="./STM32面试考点.assets/image-20250323194900539.png" alt="image-20250323194900539" style="zoom: 67%;" />

```c
void EXTI9_5_IRQHandler(void){
	if(EXTI_GetFlagStatus(EXTI_Line5) == SET) { // 线5触发的中断
		EXTI_ClearFlag(EXTI_Line5); // 清除中断标志位
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET); // 亮灯
	}
    
	if(EXTI_GetFlagStatus(EXTI_Line6) == SET) { // 线6触发的中断
		EXTI_ClearFlag(EXTI_Line6); // 清除中断标志位
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET); // 灭灯
	}
}
```



# ——————————————

# 8.1_[时钟]时钟树

<img src="./STM32面试考点.assets/image-20250311113555194.png" alt="image-20250311113555194" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311113656115.png" alt="image-20250311113656115" style="zoom:60%;" />

## 时钟和时钟树的概念

时钟定义：高低变化的方波信号，类似于人的心跳。时钟树组成：由分频器、锁向环和复用器等零部件组成

<img src="./STM32面试考点.assets/image-20250324161506253.png" alt="image-20250324161506253" style="zoom:67%;" />

## 时钟树的基本零部件

1.分频器：对输入信号的频率做除法，固定或可选的分频系数。 

2.锁向环：对输入信号的频率做乘法，可选的倍频系数。 

3.复用器：从多路输入信号中选择一路作为输出。

## 时钟树的树根

1.树根位置：位于时钟树最底部，包括HSI、HSE、LSI和LSE四个时钟源。

2.HSI和HSE：高速内部和外部时钟源，频率较高。 大树

3.LSI和LSE：低速内部和外部时钟源，频率较低。 小树

4.命名规则：HS表示高速，LS表示低速，I表示内部，E表示外部。

5.内部时钟源精度不高，所以外接时钟源

<img src="./STM32面试考点.assets/image-20250324165435153.png" alt="image-20250324165435153" style="zoom:50%;" />

<img src="./STM32面试考点.assets/image-20250324165606788.png" alt="image-20250324165606788" style="zoom:67%;" />

## 时钟树的树干

1.树干功能：从树根获取时钟，进行加工，产生系统时钟(SYSCLK)。

2.系统时钟来源：直接来自HSI或HSE，或通过锁相环产生。

<img src="./STM32面试考点.assets/image-20250324194613942.png" alt="image-20250324194613942" style="zoom: 70%;" />

3.锁相环输入信号来源：来自HSI、HSE或HS。可以去灵活的控制这个sysclk的一个频率，我们只需要改变这个所向环的一个倍频系数，就可以在这里边产生不同频率的信号了

## 时钟树的树枝

1.树枝组成：AHB、APB1和APB2三条总线，每条总线上挂载不同的片上外设。

<img src="./STM32面试考点.assets/image-20250324200118801.png" alt="image-20250324200118801" style="zoom:67%;" />

2.AHB总线：挂载DMA1和DMA2。 3.APB1总线：挂载UART1、UART2、SPI1等外设。 4.APB2总线：挂载GPIOA到GPIOE等外设。

<img src="./STM32面试考点.assets/image-20250324200305207.png" alt="image-20250324200305207" style="zoom:67%;" />

每一根树枝呢，都有一个分频器

<img src="./STM32面试考点.assets/image-20250324200640150.png" alt="image-20250324200640150" style="zoom:67%;" />

# 8.2_[时钟]时钟树编程

<img src="./STM32面试考点.assets/image-20250311113742007.png" alt="image-20250311113742007" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311113752843.png" alt="image-20250311113752843" style="zoom:60%;" />

## 时钟树初始状态

时钟树的初始状态是指单片机上电或复位后的状态，了解初始状态对编程至关重要。

初始状态下，系统时钟(SISCLK)来源于HSI，频率为8MHz。 AHB分频器、APB2分频器和APB1分频器的分频系数初始均为1，因此HCLK、PCLK2和PCLK1的频率均为8MHz。

<img src="./STM32面试考点.assets/image-20250324203716036.png" alt="image-20250324203716036" style="zoom:67%;" />

通过实验验证时钟树的初始状态，使用Cortex-M3内核的频率为8MHz

#### 测试代码

```c
#include "stm32f10x.h"

void App_SystemClock_Init(void);

int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOC, &GPIO_InitStruct);
	
	while(1)
	{
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET);
		
		for(uint32_t i = 0; i<666666; i++); // 延迟500ms
		
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
		
		for(uint32_t i = 0; i<666666; i++); // 延迟500ms
	}
}
```

## 标准库启动代码

这里有一个reset handler，我们的单片机就是从这个reset handler这一行开始执行的

<img src="./STM32面试考点.assets/image-20250324204631461.png" alt="image-20250324204631461" style="zoom:67%;" />

所以在我们一行代码也不写的时候。其实它已经是通过这个system init对我们的单片机做了一个初始化了。因为这个芯片的频率越高，它的性能就越强，所以呢，标准库提供这么一段代码的作用，就是让我们在初始状态下就让这个芯片发挥一个最佳的性能。

<img src="./STM32面试考点.assets/image-20250324204724227.png" alt="image-20250324204724227" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250324205249028.png" alt="image-20250324205249028" style="zoom:67%;" />

汇编语言注释掉，英文分号。这样呢，我们单片机复位之后就是首先就是执行的这个main方法了，而不使用这个system init对我们那个时钟数进行配置。

<img src="./STM32面试考点.assets/image-20250324205347460.png" alt="image-20250324205347460" style="zoom:80%;" />

## 时钟树编程接口

复位时钟控制器 RCC = Reset And Clock Controller，理解为时钟数

带颜色的这些模块表示它们是可开关的，灰色底色的代表默认状态下代表当前状态下是关闭的。绿色底色的代表当前状态下这个模块是开启的

<img src="./STM32面试考点.assets/image-20250325104329551.png" alt="image-20250325104329551" style="zoom: 70%;" />

## 时钟树编程步骤

<img src="./STM32面试考点.assets/image-20250325110654214.png" alt="image-20250325110654214" style="zoom:80%;" />

1.开启HSE时钟，使用RCC_HSE_Config接口。 

```c
编程接口：
// HSE开关
void RCC_HSEConfig(uint32_t RCC_HSE); RCC_HSE_ON -开	RCC_HSE_OFF -关
// 获取RCC的状态
FlagStatusRCC_GetFlagStatus(uint8_t RCC_FLAG); RCC_FLAG_HSERDY -HSE就绪
```

```c
// #1. 开启HSE
RCC_HSEConfig(RCC_HSE_ON); // 开启HSE
while(RCC_GetFlagStatus(RCC_FLAG_HSERDY) == RESET); // 等待HSE就绪
```

2.配置锁相环参数，输入来源、倍频系数，使用RCC_PLL_Config接口，并启动PLL。 

```c
编程接口：
// 配置锁相环的参数
/* @参数 RCC_PLLSource选择锁相环的输入 RCC_PLLSource_HSE_Div1 -HSE
								    RCC_PLLSource_HSE_Div2 -HSE/2
									RCC_PLLSource_HSI_Div2 -HSI/2 */
void RCC_PLLConfig(uint32_t RCC_PLLSource, uint32_t RCC_PLLMul);
// 控制锁相环的开关
void RCC_PLLCmd(FucntionalStateNewState); // ENABLE -开 DISABLE -关
// 获取RCC的状态
FlagStatusRCC_GetFlagStatus(uint8_t RCC_FLAG); //RCC_FLAG_PLLRDY -PLL就绪
```

```c
// #2. 配置锁相环
RCC_PLLConfig(RCC_PLLSource_HSE_Div1, RCC_PLLMul_9); // HSE * 9
RCC_PLLCmd(ENABLE); // 开启PLL
while(RCC_GetFlagStatus(RCC_FLAG_PLLRDY) == RESET); // 等待PLL就绪
```

3.配置AHB、APB1和APB2分频器的分频系数。 

```c
编程接口：
// 设置AHB分频器的分频系数
void RCC_HCLKConfig(uint32_t RCC_SYSCLK); // RCC_SYSCLK_Div1 .. 512
// 设置APB1分频器的分频系数
void RCC_PCLK1Config(uint32_t RCC_HCLK); // RCC_HCLK_Div1 .. 16
// 设置APB2分频器的分频系数
void RCC_PCLK2Config(uint32_t RCC_HCLK); // RCC_HCLK_Div1 .. 16
```

```c
// #3. 配置AHB分频器、APB1分频器、APB2分频器
RCC_HCLKConfig(RCC_SYSCLK_Div1); // HCLK = SYSCLK / 1
RCC_PCLK1Config(RCC_HCLK_Div2);  // PCLK1 = HCLK  / 2
RCC_PCLK2Config(RCC_HCLK_Div1);  // PCLK2 = HCLK  / 1
```

4.切换系统时钟的来源，使用RCC_SYSCLK_Config接口。

```c
// 设置SYSCLK的来源
/* @参数 RCC_SYSCLKSource选择SYSCLK的来源 RCC_SYSCLKSource_HSI
									    RCC_SYSCLKSource_HSE
									    RCC_SYSCLKSource_PLLCLK */
void RCC_SYSCLKConfig(uint32_t RCC_SYSCLKSource);
// 获取SYSCLK的来源
// @返回值 0x00 -HSI   0x04 -HSE   0x08 -锁相环
uint8_t RCC_GetSYSCLKSource(void);
```

```c
// #4. 选择SYSCLK的来源
RCC_SYSCLKConfig(RCC_SYSCLKSource_PLLCLK);  // SYSCLK来自锁相环
while(RCC_GetSYSCLKSource() != 0x08); 		// 等待切换完成
```

## Flash指令预取

左边是单片机内部的一个结构框图

这个flash就相当于电脑里面的硬盘，是用来存储程序的，用户写的这个代码就是被烧录在这个flash模块当中。当这个单片机运行的时候，cortes-m3内核会从flash里边把这个代码一条一条读出来，然后放在这个内核里面去执行。但是这个flash模块读取的速度远远赶不上我们这个内核程序执行的速度，所以在实际运行的过程中flash模块就会拖后腿。

通过指令预取，flash模块提前把下一条要执行的指令放在这个缓冲区里，当这个内核需要指定的时候，它直接从缓冲区里边往外读，提高这个代码的执行速度。

本来在这个system init这个代码里边，这个指令预取已经帮我们开了，但是刚才我们把这两行给注释掉了，所以现在我们需要手动的去开启它

<img src="./STM32面试考点.assets/image-20250325200923449.png" alt="image-20250325200923449" style="zoom:80%;" />

配置Flash访问延迟，根据时钟频率调整等待周期。因为这个flash模块它的速度太慢了。它的这个接口的操作速度，不能大于24M赫兹

## 片上外设开关和复位

通过时钟树开启或关闭片上外设的时钟

![image-20250325212106517](./STM32面试考点.assets/image-20250325212106517.png)

enable就相当就相当于我们按下了这个复位按钮，而disable就相当于我们抬起了这个复位按钮

<img src="./STM32面试考点.assets/image-20250325212208824.png" alt="image-20250325212208824" style="zoom:80%;" />

# ——————————————

# 9.1_[定时器]时基单元

<img src="./STM32面试考点.assets/image-20250311113843082.png" alt="image-20250311113843082" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311113900089.png" alt="image-20250311113900089" style="zoom:60%;" />

## 定时器介绍

<img src="./STM32面试考点.assets/image-20250325214931416.png" alt="image-20250325214931416" style="zoom:67%;" />

### 参考手册

<img src="./STM32面试考点.assets/image-20250325215455768.png" alt="image-20250325215455768" style="zoom:67%;" />

1.定时器是STM32单片机内部的重要外设，用于测量时间间隔或产生定时脉冲。 

2.STM32单片机内部有3个定时器，分为高级定时器、通用定时器和基本定时器三种。

3.高级定时器功能最全，通用定时器是高级定时器的精简版，基本定时器则是进一步精简

STM32F1系列芯片内部最多有14个定时器，其中定时器一和定时器八是高级定时器，定时器六和定时器七是基本定时器，其余是通用定时器

<img src="./STM32面试考点.assets/image-20250325215832063.png" alt="image-20250325215832063" style="zoom:67%;" />

### 定时器内部结构框图

结构复杂，包括实际单元、输出比较、输入捕获和从模式控制器四个部分

<img src="./STM32面试考点.assets/image-20250325220106815.png" alt="image-20250325220106815" style="zoom:70%;" />

## 时基单元结构

时基单元由时钟源、预分频器、计数器、自动重装寄存器和重复计数器五个部分组成

<img src="./STM32面试考点.assets/image-20250325220428564.png" alt="image-20250325220428564" style="zoom:67%;" />

1.时钟源提供定时器的时钟信号，来自内部或外部。 

2.预分频器对时钟信号进行降频，分频系数可设。 

3.计数器对脉冲进行计数，计数周期由自动重装寄存器决定，决定定时周期长短（定时器会溢出），计数器cnt的一个计数周期就是 ARR的值+1

4.重复计数器设置重复计数的次数，计数器需要溢出指定次数后才会产生update事件

<img src="./STM32面试考点.assets/image-20250325221003419.png" alt="image-20250325221003419" style="zoom:50%;" />

## 计数器技术方向

计数器有三种技术方向：上计数（Count Up）、下计数（Count Down）和中心对齐（Center Line）

- 上计数从零开始计数，达到自动重装寄存器值后溢出。 
- 下计数从自动重装寄存器值开始计数，达到零后溢出。 
- 中心对齐先从上计数到自动重装寄存器值，再从零开始下计数。

<img src="./STM32面试考点.assets/image-20250325224422640.png" alt="image-20250325224422640" style="zoom:67%;" />

## 时钟来源

<img src="./STM32面试考点.assets/image-20250326085146133.png" alt="image-20250326085146133" style="zoom:67%;" />

时钟内部来源来自从模式控制器，外部来源来自时钟树

这个备频器的备频系数是自动决定的，有一个规则：比如TIM1为例，如果APB2分频器的分频系数等于1，那么上面的这个倍频器的倍频系数也就是1，否则的话，这个倍频系数就是2

<img src="./STM32面试考点.assets/image-20250326085522618.png" alt="image-20250326085522618" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250326090147638.png" alt="image-20250326090147638" style="zoom:67%;" />

这里的72M赫兹呢就是pclk1的值再乘以2啊，这个就是72M赫兹。

然后呢，设置这里的预分频器psc，把它设置成71，那么72M赫兹除以71+1，就等于1M赫兹，所以得到这个1us的分辨率

然后再来设置一下自动重装寄存器ARR的值，设置成999。那么定时周期就是999+1，也就是1000个us，等于1ms，所以它的溢出周期就是1ms

最后我们再来设置一下这个重复计数器rcr，把它设置成0，每溢出1次，产生一个update事件，那么这个update事件的周期就是1ms

## 寄存器预加载

 1.寄存器预加载是指向寄存器写入值时，该值先写入影子寄存器，待计数器溢出产生update事件时才生效。

<img src="./STM32面试考点.assets/image-20250326090627947.png" alt="image-20250326090627947" style="zoom:67%;" />

2.预加载机制防止在计数过程中突然改变寄存器值导致定时器跑飞。即cnt等于65535极限时候，它才会再次发生溢出，此时周期达不到预设值

<img src="./STM32面试考点.assets/image-20250326090908288.png" alt="image-20250326090908288" style="zoom:60%;" />

### 小结

<img src="./STM32面试考点.assets/image-20250326091018086.png" alt="image-20250326091018086" style="zoom: 60%;" />

3.自动重装寄存器等关键寄存器都支持预加载功能

# 9.2_[定时器]自制延迟函数-24

<img src="./STM32面试考点.assets/image-20250311113937886.png" alt="image-20250311113937886" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311114004640.png" alt="image-20250311114004640" style="zoom:60%;" />

## 延迟函数的原理

1.延迟函数的原理是利用单片机内部的系统滴答定时器。 

2.系统滴答定时器（SysTick）位于STM32内核中，用于产生定时中断。 

3.通过配置定时器的参数，可以控制中断的频率和定时器的计数周期。

<img src="./STM32面试考点.assets/image-20250326093259455.png" alt="image-20250326093259455" style="zoom:80%;" />

## 获取单片机当前时间

1.通过配置定时器产生1ms级中断，记录当前时间

2.声明一个无符号32位整形变量currentTick，用于记录当前时间（ms）

3.在中断响应函数中，每次中断发生时，currentTick的值增加1

<img src="./STM32面试考点.assets/image-20250326093434812.png" alt="image-20250326093434812" style="zoom:67%;" />

### 延迟函数代码

```c
volatileuint32_t currentTick= 0; // 记录当前的时间，单位ms

//
// @简介：延迟一段时间
// @参数 ms：要延迟的时间（单位ms）
//
void App_Delay(uint32_t ms)
{
	uint32_t expireTime= currentTick+ ms; // 延迟结束的时间
	while(currentTick< expireTime); // 等待延迟结束
}
```

## 初始化时基单元

方法:

1. 使能这个定时器三的一个时钟，也就是使能这个时钟
2. 配置定时器的参数，包括PSC值、ARR值、RCR值、计数器的计数方向

3. 就是闭合这里的开关，让这个定时器呢运行起来

<img src="./STM32面试考点.assets/image-20250326094352127.png" alt="image-20250326094352127" style="zoom:67%;" />

调用TimeBaseInit函数进行初始化，传入定时器名称和结构体指针

### 编程接口

```c
// 配置时基单元的参数
void TIM_TimeBaseInit(TIM_TypeDef*TIMx, TIM_TimeBaseInitTypeDef*TIM_TimeBaseInitStruct);

// 闭合时基单元的开关
void TIM_Cmd(TIM_TypeDef*TIMx, FunctionalStateNewState);
```

### 初始化时基单元+中断完整代码

```c
void App_TIM3_TimeBaseInit(void)
{
	// #1. 开启定时器3的时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	
	// #2. 配置时基单元的参数
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_Period = 999;
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStruct);
	
	// #3. 闭合时基单元的开关
	TIM_Cmd(TIM3, ENABLE); 
	
	// #4. 使能Update中断
	TIM_ITConfig(TIM3, TIM_IT_Update, ENABLE); 
	
	// #5. 配置NVIC模块
	NVIC_InitTypeDef NVIC_InitStruct;
	
	NVIC_InitStruct.NVIC_IRQChannel = TIM3_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 0;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority = 0;
	NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	
	NVIC_Init(&NVIC_InitStruct);
}
```

## 中断配置

闭合update开关

<img src="./STM32面试考点.assets/image-20250326211727898.png" alt="image-20250326211727898" style="zoom:70%;" />

```c
// @简介：使能/禁止定时器的中断
// @参数：TIM_IT 中断的名字。TIM_IT_Update, TIM_IT_Trigger, TIM_IT_CC1, ...
// @参数：NewState 使能/禁止。ENABLE - 使能，DISABLE - 禁止
void TIM_ITConfig(TIM_TypeDef*TIMx, uint16_t TIM_IT, FunctionalStateNewState);
```

使用中断需要使用NVIC模块：配置一下这个中断优先级分组，比如说我们选择这个中断优先级分组2，也就是两位抢占优先级，两位子优先级

```c
NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
```

配置NVIC模块，头文件找定时器中断的头文件名称

```c
NVIC_InitTypeDef NVIC_InitStruct;
	
NVIC_InitStruct.NVIC_IRQChannel = TIM3_IRQn;
NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 0;
NVIC_InitStruct.NVIC_IRQChannelSubPriority = 0;
NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
	
NVIC_Init(&NVIC_InitStruct);
```

## 中断响应函数的编写

中断向量表里面寻找中断响应函数

<img src="./STM32面试考点.assets/image-20250326212520417.png" alt="image-20250326212520417" style="zoom:80%;" />

在中断响应函数中，检查update标志位是否触发中断。 清除update标志位，并更新currentTick的值

<img src="./STM32面试考点.assets/image-20250326212705047.png" alt="image-20250326212705047" style="zoom:70%;" />

# 9.3_[定时器]输出比较-21

<img src="./STM32面试考点.assets/image-20250311131747880.png" alt="image-20250311131747880" style="zoom:60%;" />

## 定时器通道概念

定时器有四个通道，结构相同，以通道1为例进行讲解，通道由CCRx寄存器、输入捕获和输出比较组成。

输入捕获用于测量外部输入信号的时间参数；输出比较通过定时器产生精确的定时的方波信号

<img src="./STM32面试考点.assets/image-20250326221746503.png" alt="image-20250326221746503" style="zoom:67%;" />

## 输出比较工作原理（PWM）

PWM -Pulse Width Modulation 脉冲宽度调制：一种周期固定，占空比可调的信号。通过调节占空比，等效地调节信号的输出幅度

单片机只能输出高电压3.3V，或者是低电压0V。如果我们想要输出一个2V的信号，那么直接使用这个单片机呢，是没办法输出出来的。可以使用PWM信号通过调节占空比，使用面积等效的这个形式近似的输出一个2V的电压

<img src="./STM32面试考点.assets/image-20250326223543913.png" alt="image-20250326223543913" style="zoom:67%;" />

输出比较模块会对计数器cnt的值，还有ccr的值做比较。当这个计数器cnt的值小于这个ccr的值的时候，这个位置就会输出一个高电压，当它大于这个ccr值的时候呢，这个位置就会输出一个低电压

<img src="./STM32面试考点.assets/image-20250326223916348.png" alt="image-20250326223916348" style="zoom:67%;" />

## 输出比较8种工作模式

<img src="./STM32面试考点.assets/image-20250326224059330.png" alt="image-20250326224059330" style="zoom:70%;" />

前六种模式用的较少，后两种模式用于输出PWM信号。都是通过比较CNT和CCRX的值，输出高低电压

PM1模式：计数器值小于CCRx时输出高电压，否则输出低电压。 

PM2模式：计数器值小于CCRx时输出低电压，否则输出高电压。

<img src="./STM32面试考点.assets/image-20250326224533347.png" alt="image-20250326224533347" style="zoom:67%;" />

## 互补输出概念

定时器每个通道都有两个引脚，一个为正常输出，一个为互补输出

正常输出直接连接参考信号，互补输出是参考信号经过反向器的反相输出

![image-20250327075905515](./STM32面试考点.assets/image-20250327075905515.png)

### 同步buck电路（降压电源）

这两个开关呢，是交替导通的。比如说我们闭合上面的这个开关的时候，下面的这个开关呢就得断开，然后呢这个电源呢就通过这条路。给这里的这个电感和电容呢，进行充电。

而如果我们断开上面的开关，那么就得闭合下面的这个开关。然后这个电感和电容会通过这么一条回路放电。我们通过控制这个充电的时间和放电时间的这个比例呢，就可以调节右边的这个输出电压。

这两个开关呢，它们的这个控制逻辑正好是相反的，上面的导通的时候，下面就得断开，下面导通的时候，上面就得断开，所以我们在给这两个开关施加控制信号的时候，如果上面是高电压，那下面就得是低电压。如果上面是低电压，下面就得是高电压

最常见的就是我们控制电机的这么一个电路

<img src="./STM32面试考点.assets/image-20250327080327597.png" alt="image-20250327080327597" style="zoom: 67%;" />

## 极性选择

极性选择决定是否对输出信号进行取反。 

高级性：不取反，参考信号与正常输出相等。 低极性：取反，参考信号与正常输出相反。 

互补输出同样有极性选择，选择低极性时参考信号与互补输出相同

<img src="./STM32面试考点.assets/image-20250327080620175.png" alt="image-20250327080620175" style="zoom:67%;" />

# 9.4_[定时器]呼吸灯实验-36

<img src="./STM32面试考点.assets/image-20250311131821794.png" alt="image-20250311131821794" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311131930133.png" alt="image-20250311131930133" style="zoom:60%;" />

## 呼吸灯工作原理

解释呼吸灯的工作原理，通过调节PWM信号的占空比来控制LED的亮度。 使用正弦函数来描述LED亮度的变化，并将其转换为PWM信号的占空比

<img src="./STM32面试考点.assets/image-20250405213849081.png" alt="image-20250405213849081" style="zoom:67%;" />

## 定时器初始化

红色的LED把它连接到这个通道1的正常输出上，然后蓝色的LED把它接到这个互补输出上，这两个波形呢，正好是相反的

<img src="./STM32面试考点.assets/image-20250405214545074.png" alt="image-20250405214545074" style="zoom:50%;" />

引脚分布

<img src="./STM32面试考点.assets/image-20250405221935979.png" alt="image-20250405221935979" style="zoom:67%;" />

高级定时器，2MHz输出频率（PWM信号设置的周期是1ms，所以它的频率换算过来就是1000赫兹，比较低的频率）

<img src="./STM32面试考点.assets/image-20250405222310613.png" alt="image-20250405222310613" style="zoom:80%;" />

### 引脚代码初始化

```c
	// #1. 初始化IO引脚 PA8 PB13
	
	GPIO_InitTypeDef GPIO_InitStruct;
	
	// PA8 AF_PP
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_8;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// PB13
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	
	GPIO_Init(GPIOB, &GPIO_InitStruct);

```

### 时基单元代码配置

输出比较：CNT的值和CCRx的值比较，决定了高电压所占的时间，其实也就间接的决定了pwn信号的一个占空比；ARR自动重装寄存器的值决定定时器的周期

<img src="./STM32面试考点.assets/image-20250406163947949.png" alt="image-20250406163947949" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250406164045119.png" alt="image-20250406164045119" style="zoom:60%;" />

总共4步。对于自动重装寄存器、预分频器、重复计数器后面都有一个阴影，它表示寄存器的一个预加载，这个预加载功能呢，可以防止我们定时器的一个跑飞。黑色阴影默认开启，这个自动重装寄存器ARR来说，阴影是灰色的表示它的预加载默认是关闭的，需要手动去开启

<img src="./STM32面试考点.assets/image-20250406164706688.png" alt="image-20250406164706688" style="zoom:65%;" />

```c
	// #2. 配置时基单元
	
	// #2.1 开启TIM1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
	
	// #2.2 配置时基单元的参数
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;
	
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 999;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	
	TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
	
	// #2.3 开启ARR的预加载
	TIM_ARRPreloadConfig(TIM1, ENABLE);
	
	// #2.4 闭合时基单元的开关
	TIM_Cmd(TIM1, ENABLE);
```

## 输出比较配置

MOE主输出势能，只有使能了这个主输出，这个定时器的所有通道的信号才能够被使用

<img src="./STM32面试考点.assets/image-20250406165212808.png" alt="image-20250406165212808" style="zoom:67%;" />

编程接口

```c
// @简介：初始化输出比较通道1的参数
// @参数：TIM_OCInitStruct-初始化参数列表
void TIM_OC1Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct)
    
// @简介：闭合/断开MOE开关
// @参数：NewStateENABLE -闭合MOE，DISABEL -断开MOE
void TIM_CtrlPWMOutputs(TIM_TypeDef* TIMx, FunctionalState NewState)
```

### 初始输出比较代码

```c
	// #3. 初始化输出比较
	// #3.1 初始化输出比较通道1的参数
	
	TIM_OCInitTypeDef TIM_OCInitStruct;
	
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStruct.TIM_OCNPolarity = TIM_OCNPolarity_High;
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStruct.TIM_OutputNState = TIM_OutputNState_Enable;
	TIM_OCInitStruct.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStruct.TIM_Pulse = 0;
	
	TIM_OC1Init(TIM1, &TIM_OCInitStruct);
	
	// #3.2 闭合MOE总开关
	TIM_CtrlPWMOutputs(TIM1, ENABLE);
	
	TIM_CCPreloadControl(TIM1, ENABLE);
```

## 动态改变占空比



# 9.5_[定时器]输入捕获-16

<img src="./STM32面试考点.assets/image-20250311132047616.png" alt="image-20250311132047616" style="zoom:60%;" />

CCR寄存器：捕获寄存器

输入捕获的功能呢，就是去测量输入信号的一个时间参数，例如周期、脉宽、占空比等等

<img src="./STM32面试考点.assets/image-20250327081027095.png" alt="image-20250327081027095" style="zoom:67%;" />

## 输入捕获的基本工作原理

1.输入捕获通过捕捉输入信号的变化边缘（上升沿或下降沿）来测量时间参数。 

2.当输入捕获电路捕捉到信号变化时，会触发一个CCX事件，并将计数器CNT的值（拍照事件）保存到CCR寄存器中。 

3.通过读取CCR寄存器的值，可以计算出信号变化的时间。

### 输入捕获的实例应用

1.通过测量脉冲信号的脉宽，可以计算出脉冲的宽度。 

2.配置定时器的时钟频率和预分频器，设置计数器分辨率。 

3.将脉冲信号接入定时器的通道一和通道二，分别捕捉上升沿和下降沿。 

4.启动定时器，计数器开始递增，当边缘被捕捉时，计数器值被保存到CCR寄存器。两个CCR寄存器的值相减获得脉冲的宽度

<img src="./STM32面试考点.assets/image-20250327081644847.png" alt="image-20250327081644847" style="zoom:67%;" />

## 输入捕获的内部电路结构

交叉引用线

![image-20250327083621480](./STM32面试考点.assets/image-20250327083621480.png)

包括输入滤波、边缘检测、信号选择和分频器

### 输入滤波

输入滤波的作用是滤除输入信号上的毛刺，保证后续测量的准确性。 输入滤波模块是一个滤波器，用于滤除噪声影响

### 边缘检测

由前面的这个边缘检测电路，加上后边的这个复用器组成。可以选择检测上升沿或下降沿

### 信号选择

复用器有三个输入：TRC、直接和间接。RC来自定时器的从模式控制器，使用较少

直接来自通道本身，间接从对侧通道获取。通过交叉引用的方式，可以节省引脚资源

![image-20250327084237493](./STM32面试考点.assets/image-20250327084237493.png)

可以取直接上升沿/下降沿信号，也可以取间接上升沿/下降沿信号

只用一个引脚就可以让通道1捕获上升沿，让通道2捕获下降沿，就省出了这么一个引脚，这个引脚省出来可以干别的事情。比如说让这个引脚去驱动一个LED，或者是往这个引脚上接这么一枚按钮等

<img src="./STM32面试考点.assets/image-20250327084411074.png" alt="image-20250327084411074" style="zoom:67%;" />

### 分频器

分频器用于设置分频系数，控制触发事件的频率，执行拍照动作频率。选择1分频即可满足大多数需求。

如果选择的是二分屏的话。那么就得出现两次上升沿，这边才会产生一个ccs事件，并且进行拍照

# 9.6_[定时器]超声波测距实验-49

<img src="./STM32面试考点.assets/image-20250311132115480.png" alt="image-20250311132115480" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311132136585.png" alt="image-20250311132136585" style="zoom:60%;" />

## 超声波测距传感器HCSR04

### 工作原理

<img src="./STM32面试考点.assets/image-20250404201507499.png" alt="image-20250404201507499" style="zoom:60%;" />

### 使用方法

启动测量：向这个trigger引脚输入一个不小于10us的一个脉冲信号

这个超声波的频率呢是40kHz，会连续发八个周期，0.2ms左右 

<img src="./STM32面试考点.assets/image-20250404202001778.png" alt="image-20250404202001778" style="zoom:55%;" />

<img src="./STM32面试考点.assets/image-20250404205205012.png" alt="image-20250404205205012" style="zoom: 50%;" />

## 编码思路

### 串口初始化

初始化串口引脚

<img src="./STM32面试考点.assets/image-20250404205457672.png" alt="image-20250404205457672" style="zoom:50%;" />

开启时钟、设置usart模块参数、闭合总开关

```c
void App_USART1_Init(void)
{
	// #1. 初始化IO引脚
	GPIO_InitTypeDef GPIO_InitStruct;
	
	// PA9 AF_PP Tx
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_10MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// PA10 IPU Rx
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 初始化USART1
	// #2.1 开启USART1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	// #2.2 初始化USART1的参数
	USART_InitTypeDef USART_InitStruct;
	
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	
	USART_Init(USART1, &USART_InitStruct);
	
	// #2.3 闭合总开关
	USART_Cmd(USART1, ENABLE);
} 
```

### 基本思路

通道1，去捕获这个脉冲信号的上升沿，对应的时间被保存在这个ccr1；另一个通道去捕获输入信号的一个下降沿，对应的时间被保存在这个ccr2。然后两个捕获的结果做差乘以分辨率，就是这个脉冲的宽度

<img src="./STM32面试考点.assets/image-20250404214608089.png" alt="image-20250404214608089" style="zoom:60%;" />

测量流程：

1.对CNT清零，避免溢出情况

2.对cc1和cc2清零，标志位

3.开启定时器

4.发送脉冲

5.等待cc1和cc2，通过查cc1和cc2标志位，判断通道捕捉完成

6.关闭定时器

<img src="./STM32面试考点.assets/image-20250404215456399.png" alt="image-20250404215456399" style="zoom:60%;" />

### 初始化时基单元

台阶的宽度就是分辨率，而溢出一次的这个时间是周期

<img src="./STM32面试考点.assets/image-20250404221806185.png" alt="image-20250404221806185" style="zoom:60%;" />

相关资料显示HCSR04传感器的测距精度是3mm，将3mm ÷ 340m/s = 8us 的时间精度，所以每个台阶之间的间距只要小于这个8us 就可以了。比如取这个分辨率1us，这样呢就可以完全满足要求了

周期越长越好。时基单元的几个寄存器是16位的，范围是0~65535，要取一个最大的值，所以让ARR=65535。这样让它的周期呢达到一个最大值

使用定时器TIM1，是挂载在APB2总线上的，pclk2的当前频率是72MHz，而APB2分频器的分频系数是1分频，定时器TIM1前面的倍频系数是1倍频。然后计算出定时器TIM1的时钟频率就等于pclk2×1，也就是72MHz×1，最终得到72MHz

<img src="./STM32面试考点.assets/image-20250405110804136.png" alt="image-20250405110804136" style="zoom:60%;" />

#### 代码

```c
// #1. 初始化时基单元
RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);

TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct;

TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
TIM_TimeBaseInitStruct.TIM_Period = 65535;
TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;

TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
```

此处暂时不闭合时钟的开关

### 初始化输入捕获

接线图

<img src="./STM32面试考点.assets/image-20250405171341228.png" alt="image-20250405171341228" style="zoom: 67%;" />

1.先初始化使用的引脚

建议设置为输入下拉，可以提供一个默认的低电压，模拟这个脉冲信号的空闲状态

<img src="./STM32面试考点.assets/image-20250405171516823.png" alt="image-20250405171516823" style="zoom:67%;" />

2.初始化输入捕获本身

<img src="./STM32面试考点.assets/image-20250405171737208.png" alt="image-20250405171737208" style="zoom:60%;" />

input capture 编程接口

```c
// @参数：初始化输入捕获
void TIM_ICInit(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct)
```

#### 代码

```c
// #2. 初始化输入捕获
// #2.1. 初始化IO引脚 PA8 IPD
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_8;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPD;
GPIO_Init(GPIOA, &GPIO_InitStruct);

// #2.2. 初始化输入捕获的通道1

TIM_ICInitTypeDef TIM_ICInitStruct;

TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;
TIM_ICInitStruct.TIM_ICFilter = 0; // 不使用滤波器
TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising; // Falling下降沿，Rising上升沿
TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1; // DIV1代表除以1
TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI; // 配置复用器直接/间接/TRC
	
TIM_ICInit(TIM1, &TIM_ICInitStruct);
	
// #2.3. 初始化输入捕获的通道2
TIM_ICInitStruct.TIM_Channel = TIM_Channel_2;
TIM_ICInitStruct.TIM_ICFilter = 0;
TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Falling;
TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;
TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_IndirectTI;

TIM_ICInit(TIM1, &TIM_ICInitStruct);
```

### ==超声波测距代码==

<img src="./STM32面试考点.assets/image-20250404215456399.png" alt="image-20250404215456399" style="zoom:60%;" />

```c
int main(void)
{
	App_USART1_Init();
	App_HCSR04_Init();
	
	My_USART_SendString(USART1, "Hello world. \r\n");
	
	while(1)
	{
		// #1. 向CNT写0
		TIM_SetCounter(TIM1, 0);
		
		// #2. 清除CC1和CC2标志位
		TIM_ClearFlag(TIM1, TIM_FLAG_CC1);
		TIM_ClearFlag(TIM1, TIM_FLAG_CC2);
		
		// #3. 开启定时器
		TIM_Cmd(TIM1, ENABLE);
		
		// #4. 向TRIG引脚发送10us的脉冲
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
		DelayUs(10);
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
		
		// #5. 等待测量完成
		while(TIM_GetFlagStatus(TIM1, TIM_FLAG_CC1) == RESET);
		while(TIM_GetFlagStatus(TIM1, TIM_FLAG_CC2) == RESET);
		
		// #6. 关闭定时器
		TIM_Cmd(TIM1, DISABLE);
		
		uint16_t ccr1 = TIM_GetCapture1(TIM1);
		uint16_t ccr2 = TIM_GetCapture2(TIM1);
		
		float distance = (ccr2 - ccr1) * 1.0e-6f * 340.0f / 2;
		
		My_USART_Printf(USART1, "distance = %.4f\r\n", distance);
		
		Delay(100);
	}
}

void App_HCSR04_Init(void)
{
	// #3. 初始化Trig引脚
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
}
```

距离数值计算

<img src="./STM32面试考点.assets/image-20250405202107491.png" alt="image-20250405202107491" style="zoom:67%;" />

# 9.7_[定时器]从模式控制器-21

<img src="./STM32面试考点.assets/image-20250311132202453.png" alt="image-20250311132202453" style="zoom:60%;" />

## 从模式控制器

<img src="./STM32面试考点.assets/image-20250407213731195.png" alt="image-20250407213731195" style="zoom: 55%;" />

1.从模式控制器由4部分组成：从模式控制器本身、输入TRGI、输出TRGO、双向箭头（从模式控制器和下面的时基单元之间的一个相互的作用）

2.TRGI是触发输入，相当于红外接收头，用于接收控制信号。 

3.TRGO是触发输出，相当于红外遥控器，用于发送控制信号。

<img src="./STM32面试考点.assets/image-20250407230846869.png" alt="image-20250407230846869" style="zoom: 67%;" />

## 从模式的八种模式

<img src="./STM32面试考点.assets/image-20250412201536478.png" alt="image-20250412201536478" style="zoom: 80%;" />

1.从模式禁止：不使用trg I信号，关闭红外接收头。

2.编码器模式：用于测量电机转过的角度或角速度，以把编码器接入到定时器当中，通过从模式控制器去控制这个计数器cnt对这个编码器进行计数

3.复位模式：当TRGI输入上升沿时，对计数器cnt清零

<img src="./STM32面试考点.assets/image-20250407234315548.png" alt="image-20250407234315548" style="zoom:60%;" />

4.门模式：当TRGI输入高电压时，开关闭合，计数器cnt的值开始递增；低电压时，开关断开，cnt的值就会保持不变。 

<img src="./STM32面试考点.assets/image-20250407234527782.png" alt="image-20250407234527782" style="zoom:60%;" />

5.触发模式：当TRGI输入上升沿时，开关闭合。 

<img src="./STM32面试考点.assets/image-20250412201218575.png" alt="image-20250412201218575" style="zoom:60%;" />

6.外部时钟模式一：将TRGI输入的信号作为实际单元的时钟来源，之前可以认为定时器的时基单元的时钟来源是来自于时钟数

<img src="./STM32面试考点.assets/image-20250412201603671.png" alt="image-20250412201603671" style="zoom:60%;" />

例子：预分频器PSC的值设置成0，计数器CNT的技术方向设置成上技数，TRGI上每输入一个脉冲信号的时候，这个计数器CNT的值就会增加一

<img src="./STM32面试考点.assets/image-20250412201834273.png" alt="image-20250412201834273" style="zoom:60%;" />

## 主模式的八种模式

主模式：通过这个TRGO输出控制信号去控制别的模块，也就相当于这个遥控器的一个功能

<img src="./STM32面试考点.assets/image-20250412204520120.png" alt="image-20250412204520120" style="zoom:80%;" />

2.使能模式：根据开关状态决定TRGO输出的电压

当这个开关断开的时候，这个TRGO上输出的就是一个低电压，或者说是0。而当这个开关闭合的时候，这个TRGO上呢，就会输出一个高电压，或者说是1

<img src="./STM32面试考点.assets/image-20250412204504444.png" alt="image-20250412204504444" style="zoom:67%;" />

3.update模式：每当实际单元产生update事件时，TRGO输出一个脉冲

<img src="./STM32面试考点.assets/image-20250412204550898.png" alt="image-20250412204550898" style="zoom:67%;" />

## 定时器同时启停的例子

通过将前一个定时器的TRGO设置为使能模式，后一个定时器的TRGI设置为门模式，实现定时器的同时启停

<img src="./STM32面试考点.assets/image-20250412210445387.png" alt="image-20250412210445387" style="zoom:67%;" />

使能模式根据开关状态决定tr go输出的电压。 3.门模式根据trg I输入的电压控制开关的导通或断开。

## 定时器级联的例子

1.使用三个定时器分别模拟手表的秒针、分针、时针。

 2.第一个定时器输入时钟信号，经过分频后驱动计数器CNT，模拟秒针每秒跳动。 TRGO的update模式连接到第二个定时器的TRGI

3.第二个定时器接收第一个定时器的TRGO信号，每分钟增加1，模拟分针每分钟跳动。 

4.第三个定时器接收第二个定时器的TRGO信号，每小时增加1，模拟时针每小时跳动。

<img src="./STM32面试考点.assets/image-20250412211727376.png" alt="image-20250412211727376" style="zoom:67%;" />

# 9.8_[定时器]PWM参数测量原理-27

<img src="./STM32面试考点.assets/image-20250311132226150.png" alt="image-20250311132226150" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311132243568.png" alt="image-20250311132243568" style="zoom:60%;" />

## 测量参数

<img src="./STM32面试考点.assets/image-20250413111941661.png" alt="image-20250413111941661" style="zoom:67%;" />

### PWM信号参数解释

PDM信号定义：周期恒定，占空比可调的信号。 占空比定义：一个周期内高电压所占的比例。

### 输入捕获在PWM参数测量中的应用

1.输入捕获的基本原理：通过通道1和通道2捕获输入信号的上升沿和下降沿，3个时间点来计算周期和占空比。 

2.输入捕获在PDM参数测量中的复杂性：虽然可以直接计算，但方法较为复杂。

## 定时器结构框图对应

高级控制定时器

<img src="./STM32面试考点.assets/image-20250413112620684.png" alt="image-20250413112620684" style="zoom: 70%;" />

**预分频器PSC：**负责把输入的较高的时钟频率给降频成一个频率较低的信号。**计数器CNT：**负责对左边的脉冲进行计数递增或者是递减。**自动重装寄存器ARR：**负责设置定时的周期。**重复计数器RCR：**只有高级定时器才有，设置这个重复计数的次数

<img src="./STM32面试考点.assets/image-20250413112735793.png" alt="image-20250413112735793" style="zoom:67%;" />

## 输入捕获节点信号名称解释

从模式控制器的TRGI通过这个复用器可以选择很多种不同的来源。测量PWM参数的时候,使用到的就是使用这个TI1FPE作为的来源

<img src="./STM32面试考点.assets/image-20250413171346845.png" alt="image-20250413171346845" style="zoom:67%;" />

TI = Timer Input = 定时器输入
F  = Filtered         = 滤波后的
P  = Polarized     = 选择了极性的

<img src="./STM32面试考点.assets/image-20250413173232137.png" alt="image-20250413173232137" style="zoom: 45%;" />

## TRGI信号来源

TRGI信号的四种来源：外部参考信号IO引脚输入的ETR、TRC、TI1FP1、TI2FP2

外部参考信号ETR：通过ETR引脚输入，实际应用中较少使用

<img src="./STM32面试考点.assets/image-20250413173807482.png" alt="image-20250413173807482" style="zoom:60%;" />

TRC信号：来源可以分成两个部分：一部分来自于上面的这个ITR0到ITR3；另一部分呢是来自于这里的TI1F_ED，使用较少。其实这里的ITR0到ITR3表示的是从其他定时器来的TRGO信号，也就是这个从模式控制器的一个处罚输出，一般用于这个定时器的级联

<img src="./STM32面试考点.assets/image-20250413174329934.png" alt="image-20250413174329934" style="zoom:67%;" />

定时器的级联例子举例：

TI1FP1、TI2FP2：分别来自输入捕获通道1和通道2，然后滤波、分频、极性选择后的信号

<img src="./STM32面试考点.assets/image-20250414223943389.png" alt="image-20250414223943389" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250414231100571.png" alt="image-20250414231100571" style="zoom:50%;" />

## PDM参数测量原理

PWM信号通过这个通道1输入到定时器中，输入捕获的通道1去捕获输入信号的上升沿，而让输入捕获的通道2去捕获这个信号的下降沿。需要把这个通道1的直接，也就是TIEFP1这个信号作为TRGI一个来源，输入到这个从模式控制器的TRGI这里。

捕获的是输入信号的上升沿，所以这个TIEFP1会出现一个脉冲。然后把这个TRGI设置成复位模式，那当这里有上升沿的时候，那这个从模式控制器就会对计数器CNT进行清零，这里每出现一个上升沿计数器CNT的值就会被从模式控制器清零。

<img src="./STM32面试考点.assets/image-20250414232102990.png" alt="image-20250414232102990" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250415002517258.png" alt="image-20250415002517258" style="zoom: 33%;" />

<img src="./STM32面试考点.assets/image-20250415002649477.png" alt="image-20250415002649477" style="zoom:33%;" />

# 9.9_[定时器]PWM参数测量实验-47

<img src="./STM32面试考点.assets/image-20250311132326645.png" alt="image-20250311132326645" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311132443585.png" alt="image-20250311132443585" style="zoom:60%;" />

## PWM参数测量实验概述

使用定时器从模式控制器和输入捕获测量PWM参数

<img src="./STM32面试考点.assets/image-20250415233840873.png" alt="image-20250415233840873" style="zoom:40%;" />

## 串口初始化

初始化IO引脚和USART模块。设置PA9为AFPP模式，复用输出推挽模式

USART模块初始化：开启时钟、配置参数（数据帧格式、波特率、数据收发的方向等）、闭合总开关

<img src="./STM32面试考点.assets/image-20250416202344118.png" alt="image-20250416202344118" style="zoom:40%;" />

```c
void App_USART1_Init(void)
{
	// #1. 初始化IO引脚 PA9 AF_PP
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_10MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 初始化USART1模块
	// #2.1 开启USART1模块的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	// #2.2 配置USART1的参数
	USART_InitTypeDef USART_InitStruct = {0};
	
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	
	USART_Init(USART1, &USART_InitStruct);
	
	// #2.3 闭合总开关
	USART_Cmd(USART1, ENABLE);
}
```

## 产生PWM信号

定时器时基单元配置：开启时钟、配置参数、使能定时器

输出比较通道初始化：初始化PA6为复用输出推挽模式，配置输出比较参数

<img src="./STM32面试考点.assets/image-20250416205823920.png" alt="image-20250416205823920" style="zoom:46%;" />

```c
void App_TIM3_Init(void)
{
	// #1. 初始化时基单元
	// #1.1 开启TIM3的时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	
	// #1.2 配置时基单元的参数
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct = {0};
	
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up; // 上计数
	TIM_TimeBaseInitStruct.TIM_Period = 999;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStruct);
	
	// #1.3 开启ARR的预加载
	TIM_ARRPreloadConfig(TIM3, ENABLE);
	
	// #1.4 闭合时基单元的总开关
	TIM_Cmd(TIM3, ENABLE);
	
	// #2. 初始化输出比较的通道1
	// #2.1 初始化PA6
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2.2 配置OC1的参数
	TIM_OCInitTypeDef TIM_OCInitStruct = {0};
	
	TIM_OCInitStruct.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStruct.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStruct.TIM_OutputState = ENABLE;
	TIM_OCInitStruct.TIM_Pulse = 0; // CCR寄存器的初始值
	
	TIM_OC1Init(TIM3, &TIM_OCInitStruct);
	
	// #2.3 使能MOE
	TIM_CtrlPWMOutputs(TIM3, ENABLE);
	
	// #2.4 使能CCRx的预加载
	TIM_CCPreloadControl(TIM3, ENABLE);
}
```

## PWM参数测量

1.时基单元初始化：配置定时器1的预分频器PSC、自动重装寄存器ARR和、重复计数器RCR 

<img src="./STM32面试考点.assets/image-20250417205924696.png" alt="image-20250417205924696" style="zoom: 40%;" />

2.输入捕获初始化：初始化输入捕获通道1和通道2，选择上升沿和下降沿捕获。 

3.从模式控制器初始化：选择TIEFP1作为TRGI的输入信号，TRGI配置为复位模式

<img src="./STM32面试考点.assets/image-20250417205659783.png" alt="image-20250417205659783" style="zoom:45%;" />

```c
void App_TIM1_Init(void)
{
	// #1. 配置时基单元
	// #1.1 开启定时器1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
	
	// #1.2 配置时基单元的参数
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct = {0};
	
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 65535;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
	
	// #1.3 开启ARR的预加载
	TIM_ARRPreloadConfig(TIM1, ENABLE);
	
	// #1.4 闭合时基单元的总开关
	TIM_Cmd(TIM1, ENABLE);
	
	// #2. 初始化输入捕获
	// #2.1 初始化PA8引脚 IPD
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_8;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPD;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2.2 初始化输入捕获通道1
	TIM_ICInitTypeDef TIM_ICInitStruct = {0};
	
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStruct.TIM_ICFilter = 0;
	TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Rising;
	TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_DirectTI;
	TIM_ICInit(TIM1, &TIM_ICInitStruct);
	
	// #2.3 初始化输入捕获通道2
	TIM_ICInitStruct.TIM_Channel = TIM_Channel_2;
	TIM_ICInitStruct.TIM_ICFilter = 0;
	TIM_ICInitStruct.TIM_ICPolarity = TIM_ICPolarity_Falling;
	TIM_ICInitStruct.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStruct.TIM_ICSelection = TIM_ICSelection_IndirectTI;
	TIM_ICInit(TIM1, &TIM_ICInitStruct);
	
	// #3. 初始化从模式控制器
	TIM_SelectInputTrigger(TIM1, TIM_TS_TI1FP1);
	TIM_SelectSlaveMode(TIM1, TIM_SlaveMode_Reset);
}
```

信号TIEFP1，是输入到TRGI的信号，把通道1的选做TRGI的输入，捕获的是上升沿。所以每当这个PWM上出现一个上升沿的时候，TRGI输入这里就会有一个脉冲，脉冲里的上升沿会触发一个trigger事件，trigger事件会对计数器cnt进行复位。同时trigger事件也会让这里的这个标志位从0变成1（也意味着测量结束了，使用前清除trigger标志位），也会对这个定时器执行指定的动作。现在把这个ccr 2和ccr 1的值读出来，那ccr 1乘以分辨率就得到了这个PWM的周期。然后用ccr 2÷ccr 1就得到了这个占空比

<img src="./STM32面试考点.assets/image-20250417214254477.png" alt="image-20250417214254477" style="zoom:50%;" />

### 测量过程

```c
int main(void)
{
	App_USART1_Init();
	App_TIM3_Init();
	App_TIM1_Init();
	
//	My_USART_SendString(USART1, "你好世界");
	
	TIM_SetCompare1(TIM3, 200);
	
	while(1)
	{
		// #1. 清除Trigger标志位
		TIM_ClearFlag(TIM1, TIM_FLAG_Trigger);
		
		// #2. 等待Trigger标志位从0编程1
		while(TIM_GetFlagStatus(TIM1, TIM_FLAG_Trigger) == RESET);
		
		// #3. 计算
		uint16_t ccr1 = TIM_GetCapture1(TIM1);
		uint16_t ccr2 = TIM_GetCapture2(TIM1);
		
		float period = ccr1 * 1.0e-6f * 1.0e3f;
		float duty = ((float)ccr2) / ccr1 * 100.0f;
		
		My_USART_Printf(USART1, "周期=%.3fms, 占空比=%.2f%%\r\n", period, duty);
		
		Delay(100);
//		float t = GetTick() * 1.0e-3f;
//		float duty = 0.5*(sin(2*3.14*t) + 1);
//		uint16_t ccr1 = duty * 999;
	}
}
```

# ——————————————

# 10.1_[ADC]逐次逼近型ADC-29

<img src="./STM32面试考点.assets/image-20250311142310662.png" alt="image-20250311142310662" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311142321173.png" alt="image-20250311142321173" style="zoom:60%;" />

## 基本概念

Analog		Digital		Converter
模拟信号	     数字信号	    转换器

电压信号转化为数字信号

<img src="./STM32面试考点.assets/image-20250328223308848.png" alt="image-20250328223308848" style="zoom:60%;" />

## 单片机内ADC模块

单片机内部结构框图：包含adc 1和adc 2两个模块，通常只学习adc 1

adc模块类型：12位逐次逼近型adc

<img src="./STM32面试考点.assets/image-20250328230909132.png" alt="image-20250328230909132" style="zoom: 67%;" />

### 采样深度概念

采样深度定义：用多少位二进制数表示转换结果

采样深度的作用：衡量adc性能的重要指标。采样深度越高，价格越贵。

<img src="./STM32面试考点.assets/image-20250328234513806.png" alt="image-20250328234513806" style="zoom:60%;" />

## 逐次逼近型ADC简介

<img src="./STM32面试考点.assets/image-20250329180415442.png" alt="image-20250329180415442" style="zoom:67%;" />

工作原理：通过逐步逼近的方式转换模拟信号为数字信号。 

示例：使用天平称重的原理进行解释，逐步添加阀码逼近实际质量。 

内部结构：包括采样保持电路、比较器、结果寄存器、电压发生器（根据结果寄存器里边所写的这个值，去产生对应的电压）等

<img src="./STM32面试考点.assets/image-20250329180809374.png" alt="image-20250329180809374" style="zoom: 67%;" />

## 采样保持电路

采样阶段：闭合开关，对模拟信号进行采样充电，这个电容充饱电之后，这个电容两端的电压就是等于模拟信号两端的这个电压。

保持阶段：断开开关，保持电容两端的电压不变

工作原理：通过开关的闭合和断开，实现模拟信号的采样和保持

<img src="./STM32面试考点.assets/image-20250329181738765.png" alt="image-20250329181738765" style="zoom: 80%;" />

比较器它的正负输入端都是有这个虚短虚断的一个特点，所以这个电流不可能通过右边给它流出去

## 例子

<img src="./STM32面试考点.assets/image-20250329182107756.png" alt="image-20250329182107756" style="zoom:67%;" />

# 10.2_[ADC]ADC模块的结构框图-25

<img src="./STM32面试考点.assets/image-20250311142631550.png" alt="image-20250311142631550" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311142724194.png" alt="image-20250311142724194" style="zoom:60%;" />

## ADC模块的基本结构和原理

<img src="./STM32面试考点.assets/image-20250329194416751.png" alt="image-20250329194416751" style="zoom:80%;" />

## ADC多路复用的原理

多路复用的概念：使用一个ADC转换多路模拟信号

逐次逼近型ADC的组成：比较器、采样保持电路、结果寄存器、电压发生器

<img src="./STM32面试考点.assets/image-20250329194530346.png" alt="image-20250329194530346" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250329194741311.png" alt="image-20250329194741311" style="zoom:80%;" />

多路复用的实现：通过开关控制对不同通道的采样

<img src="./STM32面试考点.assets/image-20250329194835699.png" alt="image-20250329194835699" style="zoom:60%;" />

通道0到通道7

<img src="./STM32面试考点.assets/image-20250329195223772.png" alt="image-20250329195223772" style="zoom: 67%;" />

其中，这个通道14对应我们单片机内部的一个温度计，而通道15则对应我们单片机内部的一个参考电压

<img src="./STM32面试考点.assets/image-20250329195356576.png" alt="image-20250329195356576" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250329195453045.png" alt="image-20250329195453045" style="zoom:67%;" />

C8T6的单片机刚好没有刚好这个 PC0 到 PC3，就没有10—13的通道

<img src="./STM32面试考点.assets/image-20250329195653980.png" alt="image-20250329195653980" style="zoom: 80%;" />

## ADC常规序列和注入序列

### 常规序列

DR寄存器是用来保存常规序列的一个转换的结果

<img src="./STM32面试考点.assets/image-20250329195926393.png" alt="image-20250329195926393" style="zoom:80%;" />

常规序列的定义：ADC模块内的转换计划。每个开关闭合多长时间，那都是使用上面的这么一份计划来指定的

数字16表示的就是这个常规序列的一张表，一共有16行，每一行可以填入一项计划，所以16行最多可以填入16项计划

<img src="./STM32面试考点.assets/image-20250329200212648.png" alt="image-20250329200212648" style="zoom:70%;" />

外部触发信号：启动常规序列的信号。从这里输入一个上升沿的话，那么这个常规序列就会被执行一遍

梯形结构是复用器。这个复用器的左边有很多不同的信号，这就是外部触发信号的一些不同的来源，总共有八种不同的来源。其中前6种是跟定时器有关的；第7种来自于EXIT；而第8种则是软件启动

<img src="./STM32面试考点.assets/image-20250329200348979.png" alt="image-20250329200348979" style="zoom: 60%;" />

### TIM3_TRG0例子

<img src="./STM32面试考点.assets/image-20250329200736215.png" alt="image-20250329200736215" style="zoom: 67%;" />

#### 定时器3的结构图，时基单元

一系列以1ms为间隔的这么一个脉冲信号，然后我们再把这个脉冲信号输入到我们的这个ADC当中

<img src="./STM32面试考点.assets/image-20250329200852023.png" alt="image-20250329200852023" style="zoom:70%;" />

当这个外部触发信号，这里出现上升沿的时候，右边的这个常规序列就会被执行一遍

<img src="./STM32面试考点.assets/image-20250329202726295.png" alt="image-20250329202726295" style="zoom:70%;" />

### 注入序列

注入序列的定义：优先级高于常规序列的转换序列

<img src="./STM32面试考点.assets/image-20250329202935289.png" alt="image-20250329202935289" style="zoom:70%;" />

## 实际应用例子

先来看一下这个常规序列，首先会发来一个脉冲这里就包含了一个上升沿，所以就会出现一个上升沿。当这个外部触发信号，出现一个上升沿的时候，这个常规序列就会被启动，所以通道0、1、2，依次会被转换一遍

<img src="./STM32面试考点.assets/image-20250329203200704.png" alt="image-20250329203200704" style="zoom:67%;" />

注入序列被启动了，直接把这个常规序列给停掉，而去转换这个通道3、4，等到这个通道3、4转换完了，再继续转换通道1、2

# 10.3_[ADC]采样时间和转换时间-28

<img src="./STM32面试考点.assets/image-20250311142812788.png" alt="image-20250311142812788" style="zoom:60%;" />

## 采样时间和转换时间的概念

<img src="./STM32面试考点.assets/image-20250329204137448.png" alt="image-20250329204137448" style="zoom:80%;" />

1.采样时间：从模拟信号上取一个点下来所消耗的时间，即采样开关闭合的时间。 

2.转换时间：将采到的模拟信号转换成数字信号所消耗的时间。

## ADC模块的时钟频率计算

adc模块挂载在APP 2总线上，从pclk2取得时钟

实际例子中，系统时钟初始化为72MHz，pclk2也为72MHz，选择6分频后，输入到adc的时钟频率为12MHz

<img src="./STM32面试考点.assets/image-20250329205849649.png" alt="image-20250329205849649" style="zoom:70%;" />

<img src="./STM32面试考点.assets/image-20250329205915189.png" alt="image-20250329205915189" style="zoom: 80%;" />

## 转换时间的计算方法

逐次逼近型a dc的工作原理类似于天平，通过不断调节阀码的组合来逼近被测物体的实际质量

12位逐次逼近型a dc有12个比特位，需要尝试12次，每次尝试消耗一个周期。12倍周期加上0.5倍额外时间，结果为12.5倍周期。实际表示转换时间时，通常用12.5倍周期来表示

<img src="./STM32面试考点.assets/image-20250329210227978.png" alt="image-20250329210227978" style="zoom:70%;" />

## 采样时间和信号源内阻的关系

采样时间与信号源内阻有关，内阻越大，采样时间越长

<img src="./STM32面试考点.assets/image-20250329210453429.png" alt="image-20250329210453429" style="zoom:50%;" />

模型转化

<img src="./STM32面试考点.assets/image-20250329210515340.png" alt="image-20250329210515340" style="zoom:60%;" />

采样保持电路

<img src="./STM32面试考点.assets/image-20250329210646259.png" alt="image-20250329210646259" style="zoom:60%;" />

## 信号源内阻的计算方法

模块儿通过AO（Analog Output 模拟输出）这个引脚向外输出一个模拟信号，我们可以通过测量这个模拟信号的大小，来得出这个光照强度的大小

<img src="./STM32面试考点.assets/image-20250329210835627.png" alt="image-20250329210835627" style="zoom:50%;" />

光敏传感器模块的电路图

<img src="./STM32面试考点.assets/image-20250329210943781.png" alt="image-20250329210943781" style="zoom:67%;" />

通过电路图转换，计算模拟信号的内阻。使用戴维南等效定理，将二端网络等效为电压源串联电阻的形式

<img src="./STM32面试考点.assets/image-20250329211154727.png" alt="image-20250329211154727" style="zoom:67%;" />

## ADC采样时间

<img src="./STM32面试考点.assets/image-20250329213735251.png" alt="image-20250329213735251" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250329213803147.png" alt="image-20250329213803147" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250329213831371.png" alt="image-20250329213831371" style="zoom:60%;" />

R ADC采样电阻

<img src="./STM32面试考点.assets/image-20250329213923583.png" alt="image-20250329213923583" style="zoom:50%;" />

C ADC电容电阻

<img src="./STM32面试考点.assets/image-20250329213953985.png" alt="image-20250329213953985" style="zoom:50%;" />

计算结果

<img src="./STM32面试考点.assets/image-20250329214051119.png" alt="image-20250329214051119" style="zoom:60%;" />

# 10.4_[ADC]常规单通道转换-51

<img src="./STM32面试考点.assets/image-20250311143512304.png" alt="image-20250311143512304" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311143528340.png" alt="image-20250311143528340" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311150704865.png" alt="image-20250311150704865" style="zoom:60%;" />

## 常规序列的单通道转换

1.常规序列的单通道转换定义：使用ADC模块对单路模拟信号进行转换。 

2.ADC复用功能：同时转换多路模拟信号。 

3.单通道转换含义：只使用ADC对其中一路模拟信号进行转换。

## 实验安排和光敏传感器模块

<img src="./STM32面试考点.assets/image-20250329222158987.png" alt="image-20250329222158987" style="zoom:67%;" />

### ADC模块和通道选择

1. 比如选择这个通道0，那这个通道0对应单片机上的PA0引脚。所以应当把AO和PA0连接在一起。
2. 因为需要对这个通道0进行转换，所以应当把这个通道0填写到常规序列的第一行当中，然后再通过常规序列左边的这个外部触发信号，向常规序列发送一个脉冲，这个脉冲会启动这个常规序列。
3. 启动了这个常规序列后，按照常规序列里面的内容，我先闭合上面的这个开关，从PA0输入的这个模拟信号里面取一个点下来。
4. 取完这个点之后，再断开这里的开关，用后边的这个12位逐次逼近型ADC把踩到的这个点，转换成数字信号。
5. 然后转换的结果就会被保存到后边的这个DR寄存器中，最后再把这个寄存器里边的结果读出来

<img src="./STM32面试考点.assets/image-20250330115252534.png" alt="image-20250330115252534" style="zoom:70%;" />

## 1.初始化IO引脚

IO引脚初始化：将PA0引脚初始化为模拟模式

<img src="./STM32面试考点.assets/image-20250330170357760.png" alt="image-20250330170357760" style="zoom:50%;" />

查看GPIO_Mode的定义，按F12

<img src="./STM32面试考点.assets/image-20250330172431473.png" alt="image-20250330172431473" style="zoom:67%;" />

枚举类型的定义

<img src="./STM32面试考点.assets/image-20250330172453686.png" alt="image-20250330172453686" style="zoom:67%;" />

### ==总初始代码==

```c
void App_ADC1_Init(void)
{
	// #1. 初始化PA0引脚，模拟模式
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
    
    // #2. 配置ADC模块的时钟
	RCC_ADCCLKConfig(RCC_PCLK2_Div6); // 六分频
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
    
    // #3. 初始化ADC的基本参数	
	ADC_InitTypeDef ADC_InitStruct = {0};
	
	ADC_InitStruct.ADC_ContinuousConvMode = DISABLE; // 关闭连续模式
	ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right; // 右对齐
	ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; // 软件启动
	ADC_InitStruct.ADC_Mode = ADC_Mode_Independent; // 独立模式
	ADC_InitStruct.ADC_NbrOfChannel = 1; // 常规序列1个通道
	ADC_InitStruct.ADC_ScanConvMode = DISABLE;
	
	ADC_Init(ADC1, &ADC_InitStruct);
    
    // #4. 配置常规序列
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_13Cycles5);
	ADC_ExternalTrigConvCmd(ADC1, ENABLE);
	
	// #5. 闭合ADC的总开关
	ADC_Cmd(ADC1, ENABLE);
}
```

## 2.ADC模块时钟配置

ADC模块的时钟信号比较特殊，需要特殊配置。ADC模块的时钟信号来源于PCLK2，并通过分频器进行分频，选择分频系数为6，将PCLK2的频率降低到12MHz。 通过RCC模块的编程接口开启ADC1的时钟。

### 编程接口

Div  代表除的意思

```c
//设置分频器的分频系数（6分频）
RCC_ADCCLKConfig(RCC_PCLK2_Div6);
//使能ADC1的时钟
RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
```

<img src="./STM32面试考点.assets/image-20250330174536833.png" alt="image-20250330174536833" style="zoom: 67%;" />

## 3.ADC模块编程

编程接口分类：分为通用、常规序列、注入序列和校准相关接口

![image-20250330175215137](./STM32面试考点.assets/image-20250330175215137.png)

标志位：

EOC（end of convert），转化完成会从0—>1

JEOC（injected end of convert），当这个注入序列转换完成，就会从从0—>1

### ADC基本参数初始化

```c
// #3. 初始化ADC的基本参数
	
ADC_InitTypeDef ADC_InitStruct = {0};
	
ADC_InitStruct.ADC_ContinuousConvMode = DISABLE; // 关闭连续模式
ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right; // 右对齐
ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; // 软件启动
ADC_InitStruct.ADC_Mode = ADC_Mode_Independent; // 独立模式
ADC_InitStruct.ADC_NbrOfChannel = 1; // 常规序列1个通道
ADC_InitStruct.ADC_ScanConvMode = DISABLE;
	
ADC_Init(ADC1, &ADC_InitStruct);
```

结构体声明，6个成员

<img src="./STM32面试考点.assets/image-20250330201530651.png" alt="image-20250330201530651" style="zoom:60%;" />

习惯使用右对齐方式

<img src="./STM32面试考点.assets/image-20250330201348705.png" alt="image-20250330201348705" style="zoom:60%;" />

## 4.常规序列配置

将通道0写入常规序列的第一行，设置采样时间，要闭合这个外部触发信号这里的开关

<img src="./STM32面试考点.assets/image-20250330213701918.png" alt="image-20250330213701918" style="zoom:67%;" />

[ADC总初始代码](##==总初始代码==)

## 5.闭合ADC的总开关

## 软件启动

1.总开关闭合：通过adc_cmd接口闭合ADC的总开关。 

2.软件启动：通过adc_software_start_ctrl_cmd接口发送软件启动脉冲。 

3.等待转换完成：通过adc_get_flag_status接口查询EOC标志位，等待转换完成。 

4.读取转换结果：通过adc_get_convert_value接口读取DR寄存器的值。

5.通过数值计算转化，电压 = dr * 分辨率

<img src="./STM32面试考点.assets/image-20250330214658285.png" alt="image-20250330214658285" style="zoom:67%;" />

```c
while(1)
{
	// #1. 清除EOC标志位
	ADC_ClearFlag(ADC1, ADC_FLAG_EOC);
	
	// #2. 通过软件启动的方式发送脉冲
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
	
	// #3. 等待常规序列转换完成
	while(ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) == RESET);
	
	// #4. 读取转换的结果
	uint16_t dr = ADC_GetConversionValue(ADC1);
	
	// #5. 把结果转换成电压
	float voltage = dr * (3.3f / 4095);
	
	if(voltage > 1.5)
	{
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
	}
	else
	{
		GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET);
	}
}
```

<img src="./STM32面试考点.assets/image-20250330215746433.png" alt="image-20250330215746433" style="zoom: 50%;" />

## 板载LED初始化

```c
void App_OnBoardLED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	
	GPIO_Init(GPIOC, &GPIO_InitStruct);
}

```

# 10.5_[ADC]定时器触发-46

<img src="./STM32面试考点.assets/image-20250311150739465.png" alt="image-20250311150739465" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311150834105.png" alt="image-20250311150834105" style="zoom:60%;" />

## 定时器触发概念

定时器触发概念：使用定时器产生外部触发信号来启动ADC转换常规序列

<img src="./STM32面试考点.assets/image-20250330220939008.png" alt="image-20250330220939008" style="zoom:70%;" />

#### 定时器TRGO信号介绍

1.TRGO信号定义：定时器的触发输出信号，用于控制其他定时器或外设。 

2.TRGO信号模式：8种工作模式，包括UPDATE模式。 

3.UPDATE模式：每当定时器产生UPDATE事件时，TRGO上输出一个脉冲，注入序列就执行一遍

<img src="./STM32面试考点.assets/image-20250330221211791.png" alt="image-20250330221211791" style="zoom:70%;" />

## 电路连接

<img src="./STM32面试考点.assets/image-20250330222642431.png" alt="image-20250330222642431" style="zoom: 50%;" />

## 串口初始化

1.IO引脚初始化：初始化PA9为复用输出推挽模式。 

2.时钟配置：开启USART1时钟。 

3.参数配置：设置波特率、硬件流控、数据位、停止位等

4.闭合总开关

<img src="./STM32面试考点.assets/image-20250330224202959.png" alt="image-20250330224202959" style="zoom:60%;" />

```c
void App_USART1_Init(void)
{
	// #1. 初始化IO引脚，PA9
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 开启USART1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	// #3. 配置串口的参数
	USART_InitTypeDef USART_InitStruct = {0};
	
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStruct.USART_Mode = USART_Mode_Tx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART1, &USART_InitStruct);
	
	// #4. 闭合串口的总开关
	USART_Cmd(USART1, ENABLE);
	
}
```

## 定时器1的TRGO配置

1. TRGO产生思路：

需要把这个TRGO设置成update模式，update模式下每当下面的这个时基单元产生一个update事件的时候，TRGO这个位置就会出现一个脉冲信号

2. 定时器初始化步骤：使能定时器时钟、配置时基单元的参数、设置TRGO模式、开启定时器开关

<img src="./STM32面试考点.assets/image-20250331090914241.png" alt="image-20250331090914241" style="zoom:70%;" />

### TIM1的初始化代码

```c
void App_TIM1_Init(void)
{
	// #1. 使能TIM1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
	
	// #2. 配置TIM1的时基单元
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct = {0};
	
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 999;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	
	TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
	
	// #3. TRGO:Update
	TIM_SelectOutputTrigger(TIM1, TIM_TRGOSource_Update);
	
	// #4. 开启定时器1的总开关
	TIM_Cmd(TIM1, ENABLE);
}
```

## 初始化ADC（注入序列）

### 注入序列编程接口

1.注入序列长度配置：adc injected sequence length config。 

2.注入序列通道配置：adc injected channel config。 

3.外部触发信号控制：adc external trigger injected convert cmd。 

4.中心触发信号配置：adc is center trigger injected CON config。 

5.软件启动注入序列：adc software start injected ctrl cmd。 

6.读取注入序列结果：adc get injected CON value。

![image-20250331092351051](./STM32面试考点.assets/image-20250331092351051.png)

### 初始化ADC代码（注入序列）

ADC初始化步骤：初始化IO引脚、配置时钟、设置基本参数、配置注入序列额外参数、闭合总开关

配置ADC模块的时钟

<img src="./STM32面试考点.assets/image-20250331092954236.png" alt="image-20250331092954236" style="zoom: 40%;" />

配置ADC的基本参数

<img src="./STM32面试考点.assets/image-20250331093233706.png" alt="image-20250331093233706" style="zoom:50%;" />

```c
void App_ADC_Init(void)
{
	// #1. 初始化IO引脚
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AIN;
	
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 配置ADC模块的时钟
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	
	// #3. 配置ADC的基本参数
	
	ADC_InitTypeDef ADC_InitStruct = {0};
	
	ADC_InitStruct.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStruct.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStruct.ADC_NbrOfChannel = 1;
	ADC_InitStruct.ADC_ScanConvMode = DISABLE;
	
	ADC_Init(ADC1, &ADC_InitStruct);
	
	// #4. 配置注入序列的额外参数
	ADC_InjectedSequencerLengthConfig(ADC1,1);
	
	ADC_ExternalTrigInjectedConvConfig(ADC1, ADC_ExternalTrigInjecConv_T1_TRGO);
	
	ADC_ExternalTrigInjectedConvCmd(ADC1, ENABLE);
	
	ADC_InjectedChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_13Cycles5);
	
	// #5. 闭合总开关
	ADC_Cmd(ADC1, ENABLE);
}
```

## 数据读取和发送

我们把这个通道0写入到了注入序列的第一行，所以转换的结果是保存在JDR1这个寄存器里边。

我们选择的定时器1的TRGO作为外部触发信号的一个来源，是自动产生的每1ms会发送一个脉冲，所以就没必要手动的去产生这个TRGO信号了，我们所需要做的就是等待这个转换的完成，等到这个JEOC标志位从0变成1，读出这个数据之后，再把这个数据转换成电压值

```c
while(1)
{
	// #1. 等待注入序列转换完成
	while(ADC_GetFlagStatus(ADC1, ADC_FLAG_JEOC) == RESET);
	
	// #2. 读取转换的结果
	uint16_t jdr1 = ADC_GetInjectedConversionValue(ADC1, ADC_InjectedChannel_1);
	
	// #3. 清除JEOC标志位
	ADC_ClearFlag(ADC1, ADC_FLAG_JEOC);
	
	// #4. 把结果转换成电压
	float voltage = jdr1 * (3.3f / 4095);
	
	// #5. 通过串口把结果发送出发去
	My_USART_Printf(USART1, "%.3f\n", voltage);
		
}
```

### VOFA+ 1.3.10使用

设置参数

<img src="./STM32面试考点.assets/image-20250331101738266.png" alt="image-20250331101738266" style="zoom:50%;" />

 波形图显示

<img src="./STM32面试考点.assets/image-20250331102310475.png" alt="image-20250331102310475" style="zoom:60%;" />

y轴

<img src="./STM32面试考点.assets/image-20250331102352131.png" alt="image-20250331102352131" style="zoom:67%;" />

# 10.6_[ADC]扫描模式-46

<img src="./STM32面试考点.assets/image-20250311150904728.png" alt="image-20250311150904728" style="zoom:60%;" />

<img src="./STM32面试考点.assets/image-20250311150914591.png" alt="image-20250311150914591" style="zoom:60%;" />

## 扫描模式介绍

扫描模式的作用：使能ADC同时对多路模拟信号进行转换。

想要对这3个通道呢依次进行转换，如果不使用这个扫描模式的话，即使L设置成了3，而且前三行里面也填入了内容，但是adc仍然认为我们只是想对第一个通道进行转换，也就是对这里的通道3进行转换。只有使能了这个扫描模式

<img src="./STM32面试考点.assets/image-20250331102746607.png" alt="image-20250331102746607" style="zoom:60%;" />

## 电位器及其电路搭建

电位器的原理：滑动变阻器，通过旋转旋钮改变电阻比例

引脚三接高电压，引脚一接地，引脚二接ADC通道

<img src="./STM32面试考点.assets/image-20250331102913187.png" alt="image-20250331102913187" style="zoom:55%;" />

### 电位器内阻计算与采样时间设置

信号源内阻等于电阻r1和r2并联的值，计算出这个采样时间呢，大概就是10.24倍的周期

<img src="./STM32面试考点.assets/image-20250331103503156.png" alt="image-20250331103503156" style="zoom:67%;" />

<img src="./STM32面试考点.assets/image-20250331105058370.png" alt="image-20250331105058370" style="zoom: 33%;" />

<img src="./STM32面试考点.assets/image-20250331105238236.png" alt="image-20250331105238236" style="zoom: 50%;" />

<img src="./STM32面试考点.assets/image-20250331105331343.png" alt="image-20250331105331343" style="zoom:50%;" />

## 串口初始化与测试

<img src="./STM32面试考点.assets/image-20250331222326607.png" alt="image-20250331222326607" style="zoom:60%;" />

```c
void App_USART1_Init(void)
{
	// #1. 初始化IO引脚，PA9 AF_PP
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_2MHz; 
	
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 开启USART1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	
	// #3. 配置USART1的参数
	USART_InitTypeDef USART_InitStruct = {0};
	
	USART_InitStruct.USART_BaudRate = 115200;
	USART_InitStruct.USART_Mode = USART_Mode_Tx;
	USART_InitStruct.USART_Parity = USART_Parity_No;
	USART_InitStruct.USART_StopBits = USART_StopBits_1;
	USART_InitStruct.USART_WordLength = USART_WordLength_8b;
	
	USART_Init(USART1, &USART_InitStruct);
	
	// #4. 闭合USART1的总开关
	USART_Cmd(USART1, ENABLE);
}
```

## 程序思路

两个电位器给接到了ADC的这个PA0和PA1上，也就是通道0和通道1。然后把通道0和通道1写入了下面的这个注入序列当中，注入序列的执行是需要左边有一个启动信号，就是这里的外部触发信号，这时需要使用定时器1去产生一个触发信号，是以1ms为间隔的这么一个脉冲。

<img src="./STM32面试考点.assets/image-20250401002458139.png" alt="image-20250401002458139" style="zoom:60%;" />

## 定时器初始化与外部触发设置

用定时器1的从模式控制器，把定时器1的从模式控制器的TRGO设置成update模式，在这个update模式下，只要这里每产生一个update事件，那么TRGO上会对外输出一个脉冲。所以，这个问题就转换成了怎么去设置这里的时基单元，让这个update事件每1ms发生1次

<img src="./STM32面试考点.assets/image-20250401115406799.png" alt="image-20250401115406799" style="zoom:70%;" />

<img src="./STM32面试考点.assets/image-20250404115214046.png" alt="image-20250404115214046" style="zoom:60%;" />

### TIMI1初始化代码

```c
void App_TIM1_Init(void)
{
	// #1. 开启定时器1的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE);
	
	// #2. 设置时基单元的参数
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStruct = {0};
	
	TIM_TimeBaseInitStruct.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStruct.TIM_Period = 999;
	TIM_TimeBaseInitStruct.TIM_Prescaler = 71;
	TIM_TimeBaseInitStruct.TIM_RepetitionCounter = 0;
	
	TIM_TimeBaseInit(TIM1, &TIM_TimeBaseInitStruct);
	
	// #3. 将TRGO设置为Update模式
	TIM_SelectOutputTrigger(TIM1, TIM_TRGOSource_Update);
	
	// #4. 闭合TIM1的总开关
	TIM_Cmd(TIM1, ENABLE);
}
```

## ADC初始化与注入序列配置

1.把PA0和PA1都给设置成模拟模式AIN

2.ADC配置一下时钟，分频器设置（6分频），保证输入到ADC模块的时钟频率必须是小于14MHz才可以。开启时钟

3.为ADC1初始化基本参数

4.注入序列的长度，序列的内容，外部触发的选择，开关

等待注入序列执行完成，然后把这个结果读出来就可以了。通过JEOC标志位的值从0变成1的时候，说明这个注入序列执行完成了

<img src="./STM32面试考点.assets/image-20250404142506774.png" alt="image-20250404142506774" style="zoom:50%;" />

```c
void App_ADC1_Init(void)
{
	// #1. 初始化IO引脚 PA0 PA1 -> AIN
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStruct = {0};
	
	GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_0;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AIN;
	
	GPIO_Init(GPIOA, &GPIO_InitStruct);
	
	// #2. 配置ADC的时钟
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	
	// #3. 为ADC1初始化基本参数
	
	ADC_InitTypeDef ADC_InitStruct = {0};
	
	ADC_InitStruct.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStruct.ADC_Mode = ADC_Mode_Independent; // 关于双ADC模式
	ADC_InitStruct.ADC_NbrOfChannel = 1; // 关于常规序列里的16行
	ADC_InitStruct.ADC_ScanConvMode = ENABLE;
	
	ADC_Init(ADC1, &ADC_InitStruct);
	
	// #4. 配置注入序列
	ADC_InjectedSequencerLengthConfig(ADC1, 2);
	
	ADC_InjectedChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_13Cycles5);
	ADC_InjectedChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_13Cycles5);
	
	ADC_ExternalTrigInjectedConvConfig(ADC1, ADC_ExternalTrigInjecConv_T1_TRGO);
	
	ADC_ExternalTrigInjectedConvCmd(ADC1, ENABLE);
	
	// #5. 闭合ADC的总开关
	ADC_Cmd(ADC1, ENABLE);
}
```

## 数据读取与发送

```c
int main(void)
{
	App_USART1_Init();
	
	App_TIM1_Init();
	
	App_ADC1_Init();
	
//	My_USART_Printf(USART1, "Hello world. \r\n");
	
	while(1)
	{
		// #1. 等待转换完成 JEOC
		while(ADC_GetFlagStatus(ADC1, ADC_FLAG_JEOC) == RESET);
		
		// #2. 把转化的结果读出来
		uint16_t jdr1 = ADC_GetInjectedConversionValue(ADC1, ADC_InjectedChannel_1);
		uint16_t jdr2 = ADC_GetInjectedConversionValue(ADC1, ADC_InjectedChannel_2);
		
		float v1, v2;
		
		v1 = jdr1 * (3.3f / 4095);
		v2 = jdr2 * (3.3f / 4095);
		
		// #3. 把结果通过串口发送出去
		My_USART_Printf(USART1, "%.3f,%.3f\n", v1, v2);
	}
}
```

VOFA软件图像输出，串口参数设置

<img src="./STM32面试考点.assets/image-20250404165554071.png" alt="image-20250404165554071" style="zoom:50%;" />

数据格式

<img src="./STM32面试考点.assets/image-20250404165716130.png" alt="image-20250404165716130" style="zoom: 80%;" />
