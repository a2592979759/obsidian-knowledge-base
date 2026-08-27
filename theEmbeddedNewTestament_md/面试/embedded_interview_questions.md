---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Concept/embedded_interview_questions.md"
created: 2026-08-27
---

> ## 🚀 交互式练习这些嵌入式面试题
>
> 在 **[EmbeddedInterviewLab](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=embedded_qbank)** 上获取**由社区评分的、可搜索题库**，包含模型答案、点赞，以及真实的"我被问到过"数据。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=concept_qa&utm_content=embedded_qbank)** &nbsp;·&nbsp; 💻 **[带 AI 反馈练习编程题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=coding&utm_content=embedded_qbank)**

---

嵌入式面试题

操作系统

描述继承（inheritance）、多线程（multithreading）、看门狗定时器（watchdog timer）等。
继承（Inheritance）：继承允许我们通过另一个类来定义一个类，这使得创建和维护应用程序更容易。继承允许复用代码的功能，并允许快速实现。
多线程（Multithreading）：多线程是程序在同一时间管理多个用户使用的能力，甚至能够在不运行多个程序副本的情况下，管理同一用户的多个请求。
 看门狗定时器（WDT）：WDT 是一种硬件，可用于自动检测软件异常并在出现异常时复位处理器。
什么是虚拟内存（virtual memory）和缓存（caches）（也请阅读缓存一致性, Cache coherency）
机器中的物理内存是稀缺的，所以虚拟内存是一种扩展它的优化
内存以页（pages）的形式存储在硬盘上，并使用缓存
什么是优先级反转（priority inversion）、可重入（reentrancy）、自旋锁（spinlocks）
优先级反转（Priority Inversion）：低优先级任务持有信号量运行，高优先级任务在等待信号量。中等优先级任务发生中断、运行，然后高优先级才能运行
解决方案是把低优先级的优先级提升到最高，这样它就不会被中优先级抢占（这称为优先级继承, priority inheritance）
可重入（Reentrancy）：当一个函数在执行过程中被中断，如果它能够返回并恢复函数而不发生任何变化，则该函数是可重入的
不能使用全局/静态数据
自旋锁（Spinlock）：一个任务在等待某个被锁定的资源，并持续检查它是否解锁，从而消耗了大部分 CPU 资源
好的经验法则——在用户代码中避免使用它们
什么是原子编程（atomic programming）/无锁操作（non-locking operation）？
原子操作（Atomic operations）是被保证保持其值安全的操作
原子性（Atomicity）

在计算机编程中，如果一个操作被保证与可能同时发生的其他操作隔离，那么它被认为是原子的。换句话说，原子操作是不可分割的。


什么是并发（concurrency）、并行（parallelism）和多线程（multithreading）？
并发（Concurrency）基本上是在我们谈论至少两个或更多任务时适用的。当一个应用能够在同一时间虚拟地执行两个任务时，我们称之为并发应用。虽然这里的任务看起来像是同时运行，但本质上它们"可能"并不。它们利用了操作系统的 CPU 时间片（time-slicing）特性，每个任务运行其任务的一部分，然后进入等待状态。当第一个任务处于等待状态时，CPU 被分配给第二个任务来完成它的那部分任务。操作系统基于任务的优先级，从而轮流将 CPU 和其他计算资源（例如内存）分配给所有任务，给它们完成的机会。对最终用户而言，看起来所有任务都在并行运行。这被称为并发。
并行（Parallelism）不要求存在两个任务。它实际上是利用 CPU 的多核（multi-core）基础设施，在同一时间物理地运行任务的一部分或多个任务，通过将每个核心分配给每个任务或子任务。并行本质上需要具有多个处理单元的硬件。在单核 CPU 中，你可以获得并发，但不能获得并行。
进程（Thread）和线程（Process）的区别
一个进程（process）拥有执行程序所需的所有资源——内存空间、寄存器、栈空间、安全性等
一个线程（thread）是一条执行线，存在于一个进程内部
互斥量（Mutexes）和信号量（semaphores）？它们之间的主要区别是什么？二进制信号量和互斥量有什么区别？互斥量中的锁是如何实现的？
互斥量用于对资源的独占访问。二进制信号量（Binary semaphore）应该用于同步（即"嘿，各位！这件事发生了！"）。
信号量在任务之间进行信令（signal）
B 给出信号量，A 取走它；一个任务不能自己给出又取走同一个信号量
https://stackoverflow.com/questions/62814/difficult ference-between-binary-semaphore-and-mutex 
什么是抖动（thrashing）？什么是过度分页（excessive paging）？如众人所认为的那些主要区域
有大于 RAM 大小的虚拟内存
所以操作系统不断地在虚拟内存中交换内容进进出出
抖动（Thrashing）是指这会消耗大部分处理能力
什么是动态加载（dynamic loading）？什么是静态加载（static loading）？何时使用动态加载？有哪些优点？给出一个何时使用动态加载的例子？
动态加载是把代码动态地加载到内存中
静态加载是在编译时就全部解析好
常规操作系统（regular OS）和实时操作系统（real-time OS）的区别
RTOS 用于时间关键（time critical）系统
RTOS 中的任务调度是基于优先级的，常规 OS 是面向高吞吐量（high throughput）
RTOS 中的内核是可抢占的（preemptable）
如果高优先级请求到来，它们可以抢占 OS 的请求
OS 概念——什么是互斥（mutual exclusion）？
当两个进程试图进入一个临界区（critical region）时，其中一个会被阻塞
OS 程序的优先级？
EL0 - 通用应用
EL1 - OS
EL2 - 虚拟机监控器（hypervisor）
EL3 - 安全相关
描述临界区（critical section）
https://leetcode.com/discuss/interview-question/124638/what-happens-in-the-background-from-the-time-you-press-the-Power-button-until-the-Linux-login-prompt-appears/ 

 C/C++ 概念
C 中的 static 是什么？
静态变量（Static variable）是在整个函数调用过程中保持其值的变量。

函数内部的静态变量在调用之间保持其值。
静态全局变量或函数只在声明它的文件中"可见"。

什么是 volatile 关键字？
易变变量（volatile variable）可以意外改变，编译器不能对它做任何假设

http://www.drdobbs.com/cpp/volatile-the-multithreaded-programmers-b/184403766 

链表（linked list）和数组（array）的区别？何时使用链表？
链表位于堆内存（heap memory）中，并且是动态分配的。

链表依赖引用（references），每个节点由数据和指向前后元素的引用组成。

数组是存储在连续内存（sequential memory）中的连续内存块。

数组是基于索引的数据结构，每个元素都关联一个索引。




比较基准（BASIS FOR COMPARISON）

数组（ARRAY）

链表（LINKED LIST）

基础（Basic）

它是固定数量数据项的一致集合。

它是包含可变数量数据项的有序集合。

大小（Size）

在声明时指定。

无需指定；在执行期间增长和收缩。

存储分配（Storage Allocation）

元素位置在编译时分配。

元素位置在运行时分配。

元素的顺序（Order of the elements）

连续存储。

随机存储。

访问元素（Accessing the element）

直接或随机访问，即指定数组索引或下标。

顺序访问，即通过指针从列表的第一个节点开始遍历。

元素的插入和删除（Insertion and deletion of element）

相对较慢，因为需要移位。

更容易、快速且高效。

搜索（Searching）

二分查找（binary search）和线性查找（linear search）。

线性查找（linear search）。

所需内存（Memory required）

较少。

较多。

内存利用率（Memory Utilization）

低效

高效


-->何时使用链表

你需要从列表中进行常数时间的插入/删除（例如在时间可预测性绝对关键的实时计算中）
你不知道列表中会有多少项。对于数组，如果数组变得太大，你可能需要重新声明并复制内存。
你不需要随机访问任何元素
你希望能够将项插入到列表中间（例如优先队列, priority queue）

什么是悬空指针（dangling pointers）？在哪里使用它们？
悬空指针指向已经被释放的内存。存储已被释放。尝试访问它可能会导致段错误（Segmentation fault）。指向已被删除（或释放）的内存位置的指针称为悬空指针。
内存泄漏（memory leak）是尚未被释放的内存，现在无法访问（或释放）它，因为已经没有方式能再访问到它了。
悬空指针不是被有意使用的。它们根本不应该被使用，因为它们会导致程序崩溃和段错误。此外，它们还会不必要地占用内存空间。所以必须小心，在删除它所指向的变量之前，释放或置空（NULLIFY）该指针。
什么是结构体（structures）和联合体（unions）？何时使用哪个？大小？
结构体中的每个成员在该段中都有各自的内存位置
联合体的大小是其最大成员的大小
一次只能有一个成员有值
联合体在以下情况下使用：
我们创建一个判别联合（discriminated union）。这可能是你想到的"空间优化"。
还需要额外的一点数据来判断联合体的哪个成员是"有效的"（其中含有有效数据）
什么是 free()？free 怎么知道要释放多少内存？
https://stackoverflow.com/questions/1957099/how-do-free-and-malloc-work-in-c
当你 malloc 一个块时，它实际上分配的内存比你请求的要多一点。这些额外内存用于存储信息，例如已分配块的大小，以及在块链中指向下一个空闲/已使用块的链接。


当你 free 你的指针时，它使用该地址来找到它添加到所分配块开头（通常）的特殊信息。如果你传入一个不同的地址，它会访问包含垃圾数据的内存，因此其行为是未定义的（但最常导致崩溃）。


类（class）和对象（object）的区别？类还是对象创建内存？基本上要学习类和对象的每个细节，而不仅仅是定义。
类是静态的，它们只是定义对象的属性
当一个类被实例化（instantiated）时，它就成为一个对象并占用内存
类（Class）：导致面向对象编程的 C++ 的构建块是类。它是一种用户定义的数据类型，持有自己的数据成员和成员函数，可以通过创建该类的实例来访问和使用。一个类就像对象的一张蓝图（blueprint）。

对象（Object）是类的一个实例。当定义一个类时，不会分配内存，但当它被实例化（即创建了一个对象）时，就会分配内存。

例子：如果频道（channel）是一个类，那么 Star Sports、BBC 和 ESPN 就是它的对象。

一个类 "CAR"：它的对象是 Hyundai、Ford、Suzuki。它们会有相同的方法但不同的设计——这就是如何将对象和类与现实世界联系起来。

什么是虚函数（virtual functions）？它们如何使用？为什么使用？何时使用？举例？
为了实现运行时多态（runtime polymorphism）
它们在基类（base classes）中定义——并在派生类（derived classes）中重写（overridden）
例如，输入 vehicle……但不知道是 car、bus 还是 truck
虚函数是声明在基类内并由派生类重新定义（重写）的成员函数。当你使用指向基类的指针或引用来引用派生类对象时，可以为该对象调用虚函数并执行派生类版本的函数。

虚函数确保为对象调用正确的函数，无论用于函数调用的引用（或指针）类型是什么。
它们主要用于实现运行时多态。
函数在基类中用 virtual 关键字声明。
函数调用的解析在运行时完成。
乘法是如何实现的？
将结果设为 0
重复
将第二个乘数左移，直到最右边的位与第一个乘数中最左边的 1 对齐
将第二个乘数在该位置加到结果上
从第一个乘数中移除那个 1
直到第一个乘数为 0
结果是正确的
设计一个电梯系统
我开始思考在这个系统中应该追踪多少个按钮（包括电梯内的按钮，以及每层楼外面的按钮）。
之后，考虑如何实现一个 ISR（中断服务例程, interrupt service routine）以及应该使用什么数据结构（可能是一个队列来存放下一个目标楼层，以及一个数组来存放所有按钮的状态）来追踪它们，并使用这些信息来控制电梯。
后置递增（post increment）如何工作。
你先使用该值，然后再递增它。
一位提问者问如何修改 malloc 以保证它按 32 字节对齐
ptr & ~0x1F 会将最后 5 位设为 0，使其按 32 字节对齐
C 程序中的断点（breakpoints）是如何工作的？
http://www.nynaeve.net/?p=80 
调试器（debugger）是如何工作的？
ISR 与中断向量表（interrupt vector table）
ISR 地址存储在向量表中
浮点转整型（Float to int conversion）
程序的内存映射（memory map）
来自链接脚本（linker script）的输出
Bss 是未初始化数据
Data 是已初始化数据
Text 是名称、符号等
如果我们声明的变量数量多于处理器上可用的寄存器？它们会存储在哪里？
存储在某些内存块中，并在适当时加载回寄存器
必须移除此时内存中的某些东西
如何处理泛型函数（Generic functions），例如 void 指针
接受 void 指针的函数应该只接受一种类型
数据结构/类内存填充（memory padding）
// char         1 字节
// short int    2 字节
// int          4 字节
// double       8 字节
// pointer      32 位机器上 4 字节，64 位机器上 8 字节
内存被填充到最近的倍数
为了优化对字节寻址内存（byte addressable memory）的内存访问
宏（Macro）和内联（Inline）的区别？                                
宏用于在整个代码中重复函数，所以你在顶部使用 #define，预处理器通常会处理这件事
内联函数告诉编译器该函数也在那个位置被定义——这样你通过不跳转到另一个位置、再跳转回来而节省了时间
内联函数比宏提供了以下优点。
由于它们是函数，编译器会检查参数的类型是否正确。
多次调用没有风险。但宏有风险，当参数是一个表达式时可能是危险的。
它们可以包含多行代码，而不需要尾随的反斜杠（backslashes）。
内联函数有自己的变量作用域，并且可以返回一个值。
与宏相比，内联函数更容易调试代码。
如何避免上溢/下溢（overflow/underflow）
使用像 long double 这样的类型以获得更高的精度和更多的位
如何在两个字节序（endianess）不同的机器之间通过网络发送数据
https://www.ibm.com/developerworks/aix/library/au-endianc/index.html 
https://barrgroup.com/Embedded-Systems/How-To/Big-Endian-Little-Endian
编程（Coding）
编写一个 C 程序来反转句子中的单词
我用指针算术来解决这个问题。首先我反转整个句子中的字符，然后在下一步中，反转每个单词。
编写一个 C 程序来编码一个 32 位数字中的位，使得最高有效 16 位被反转，但低 16 位保持不变。然后被要求将其推广到任意数量的位。
用这个来全部反转：http://www.geeksforgeeks.org/write-an-efficient-c-program-to-reverse-bits-of-a-number/ 
左移 16 位以使高 16 位被反转
与 0x0000FFFF 与运算原数字以清零高 16 位
异或 c 和 d 
计算一个整数中置位（set bits）的个数
只需在一个 while 循环中做 n &= (n-1) 并递增计数，返回计数
从一个数中减去 1 会翻转所有的位（从右到左），直到最右边的置位（包括最右边的置位）。所以如果我们把一个数减 1 再与其自身按位与（n & (n-1)），我们就取消了最右边的置位。如果我们在一个循环中做 n & (n-1) 并统计循环执行的次数，我们就能得到置位的个数。
这个解法的美妙之处在于循环的次数等于给定整数中置位的个数。
http://www.geeksforgeeks.org/count-set-bits-in-an-integer/
位操作问题——检测 1 的模式，写掩码在 32 位整数中插入 1 的模式，交换相邻的奇偶位（答案链接：https://www.geeksforgeeks.org/swap-all-odd-and-even-bits/）
字符串和数组操作——反转字符串，反转字符串中的单词，在数组中查找重复项  
Write
找到字符串中第一个不重复的字符。即输入 "abbcdcaea" 会返回 "d"
https://www.glassdoor.com/Interview/white-board-find-the-first-non-recurring-character-in-a-string-i-e-input-abbcdcaea-would-return-d-QTN_1942494.htm
使用链表实现一个带有 push/pop 功能的队列/FIFO
使用链表创建一个自定义的 malloc 和 free 函数
不使用临时变量交换两个指针的值
用位运算 3 次 x ^ y……x=, y=, 然后 x-
http://www.geeksforgeeks.org/swap-two-numbers-without-using-temporary-variable/
编写一个函数，判断给定变量是否是 2 的幂
((x != 0) && !(x & (x - 1)));
对 0 以外的数有效，所以还需要另一个检查来处理 0
编程题（现场）（反转一个链表）
接受一个 "数独棋盘（sudoku board）" 的二维数组的函数，并检查它是否是一个可行的棋盘
解释并描述二叉搜索树（binary search tree）是如何工作的
最坏 O(n)，平均 O(log n)
给定一个从 1 到 100 的列表，说出判断是否有重复项的所有不同方法。哪种最高效？
双重循环 O(n^2)
对所有元素求和，如果大于 n(n+1)/2 则有重复 O(n)
边遍历边将元素加入一个集合，如果已存在则有重复 O(n)
加入一个哈希映射（hashmap），统计每个数出现的次数 O(n)
排序，并比较每个元素与其前一个元素 O(n)
另一种不使用 sizeof() 求 sizeof 和数据类型的方法？
((&(var)+1) - &var)
编写一个 C++ 程序来查找链表中的环。
编写 C 代码对字符串进行哈希，并通过实现链表来处理冲突解决。这段代码是线程安全的吗？
如何将字符串转换为整数
Strtol 方法
编写一个程序删除节点，只给一个指向循环链表（Circular linked list）中节点的指针。
编写自己的 strstr 函数
返回 str2 在 str2 中第一次出现的位置
编写一个程序将给定的单链表转换为二叉搜索树（BST）
1) 找到链表中点并使其成为根。
2) 对左半部分和右半部分递归地做同样的事。
      a) 找到左半部分的中点并使其成为步骤 1 中所创建根的左子节点。
      b) 找到右半部分的中点并使其成为步骤 1 中所创建根的右子节点。
交换大端（Big Endian）到小端（Little Endian）以及反向
https://stackoverflow.com/questions/2182002/convert-big-endian-to-little-endian-in-c-without-using-provided-func
编写一个程序实现 memcpy()
将内存从一个位置复制到另一个位置，共 x 字节
从链表末尾删除第 n 个节点
 编写一个程序来共享资源 - https://www.thecrazyprogrammer.com/2016/09/producer-consumer-problem-c.html 
编写一个程序来识别大端（Big Endian）与小端（Little Endian）
位操作问题 -
反转一个整数的位
检查给定数字的奇偶性（parity）
交换给定整数中的位（位置已提供）
为以下内容编写你自己的函数 -
Sizeof 函数
Memcpy
Memmove
Malloc
Free
被要求用 C 实现一个（字符串）字典（存储和搜索）。
在一个大元素数组中查找最小和最大的数字。

通用（General）
判断旋转圆盘的方向
正交解码（Quadrature decoding）
一个脑筋急转弯：用两根绳子找出 45 分钟。已知一根绳子完全烧完需要 1 小时，且燃烧速率不一致。
从两端点燃第一根绳子，只从一端点燃第二根绳子。
当第一根完全烧完时，30 分钟过去，第二根还剩 30 分钟。
现在从两端点燃已经烧了 30 分钟的第二根绳子，这将以两倍的速度烧完一根 30 分钟的绳子，使其在 15 分钟内烧完。
总共就是 45 分钟。
什么是中断（interrupts）？如果处理器上的外部中断引脚较少，如何接入多个中断？
解释 CDMA、LTE 等。
TCP 和 UDP 的区别是什么。
回调函数（Call back functions）
是作为参数传入并在某个事件发生后被调用的函数

UART
通用异步收发器（Universal Asynchronous Receiver/Transmitter）
串行数据（Serial data）
2 根线
Rx & Tx
没有时钟信号来同步数据
添加停止位和起始位来标识
两个 UART 端口必须工作在相似的波特率（baud rate）
不支持多个主/从，只能两个设备

SPI
串行外设接口（Serial Peripheral Interface）
一个主机可以控制多个从机
MISO、MOSI、CLK、SS
数据可以连续流式传输
         无停止/起始位


I2C
SDA/SCL
与时钟信号同步
起始 + 地址帧 + ACK + 数据帧 + ACK + 停止
双向传输

CAN
控制器局域网（Controller Area Network）
异步串行通信
CSMA/CD（冲突检测, collision detection），但对冲突使用优先级级别
常态约 2.5V，CANH 在 3.5V，CANL 约 1.5V，差分信号（differential signal）

低功耗蓝牙（Bluetooth Low Energy，BLE）
从机连接是独占的（exclusive）
主机可以连接多个从机，但从机只能连接一个主机
GATT 配置文件（profile）
服务（Services）——功能
特征（Characteristics）——数据点
广播模式（Advertising mode）——向附近的外围设备发送信息
连接模式（Connected mode）——一对一连接，共享数据
100m 范围（蓝牙 5 编码 PHY 可以将其延伸得更远）
点对点（Point to point）、网状网络（mesh network）（蓝牙 Mesh 有单独的规范）
网格（mesh）的优势是它们可以在无需专用网关的情况下彼此通信。

4G/LTE
CDMA 使用码分复用（code division multiplexing）
LTE 是基于 IP 的系统（以数据为中心）
IP 隧道（IP tunnel）
高速/低延迟
5G
目标是能够同时连接到成千上万的设备
提高吞吐量，降低延迟
更高的频率以实现更快的传输
尚未正式发布
排序算法

排序

时间复杂度

空间复杂度


最好

最坏

平均

最好

最坏

平均


冒泡排序（Bubble Sort）

O(n^2)

O(n^2)

O(n^2)

O(1)

O(1)

O(1)

完成一些预排序

自适应冒泡（Adaptive Bubble）

O(n)

O(n^2)

O(n^2)

O(1)

O(1)

O(1)


选择排序（Selection Sort）

O(n^2)

O(n^2)

O(n^2)

O(1)

O(1)

O(1)

适合小数组

插入排序（Insertion Sort）

O(n^2)

O(n^2)

O(n^2)

O(1)

O(1)

O(1)

对部分排序的数据效果好

自适应插入（Adaptive Insertion）

O(n^2)

O(n^2)

O(n^2)

O(1)

O(1)

O(1)


桶排序（Bucket Sort）

O(n)

O(n)

O(n)

O(1)

O(1)

O(1)

适合小范围数据

快速排序（Quick Sort）

O(n log n)

O(n^2)

O(n log n)

O(1)

O(log n)

O(log n)

最快的排序，适合先开始并用插入排序处理内部循环

归并排序（Merge Sort）

O(n log n)

O(n log n)

O(n log n)

O(n)

O(n)

O(n)


堆排序（Heap Sort）

O(n log n)

O(n log n)

O(n log n)

O(1)

O(1)

O(1)

下沉（fixdown）、上浮（fixup, log n）、建堆（build heap, n）堆排序（建堆并从底部下沉）


额外的固件面试题——来自 glassdoor

不使用临时变量交换两个变量
如何反转一个字节中的所有位？
C 函数接受两个位索引和一个整数，反转该整数中这两个索引之间的位。
说出几种不同类型的寄存器。
什么是优先级反转？有哪些避免它的方法？
在资源受限环境中，C 中递归函数调用的缺点。
什么是接口（interface）？
什么是抽象类（abstract class）？——C++ 概念
JTAG 电缆中有多少引脚？—— 20
你会使用哪些技术来降低嵌入式系统中的功耗？
睡眠模式（sleep modes）、刷新操作（refresh operation）
你会使用什么样的数据结构来存储来自串行接收线的数据？
队列（queue）—— FIFO
     12. 编写一个 C 程序来确定微处理器的字节序（endianness）

     13. 检测链表中的环

     14. 信号量、互斥量和自旋锁的区别

     15. UART 可能发生通信错误的方式有哪些？

     16. 遍历二叉树（Binary Tree）

     17. 在数组中使用二分查找（Binary Search）查找一个元素

     18. 开关去抖动（switch debouncing）软件逻辑

     19. 全局变量存储在哪里？

     20. 线程间通信、进程间通信

     21. 什么是 extern 变量？

C 中的 5 个内存段（Memory Segments）：

1. 代码段（Code Segment）
代码段，也称为文本段（text segment），是包含经常执行的代码的内存区域。
代码段通常是只读的，以避免被编程错误（如缓冲区溢出, buffer-overflow）覆盖的风险。
代码段不包含程序变量，如局部变量（在 C 中也称为自动变量, automatic variables）、全局变量等。
根据 C 的实现不同，代码段也可以包含只读的字符串字面量。例如，当你执行 printf("Hello, world") 时，字符串 "Hello, world" 会被创建在代码/文本段中。你可以使用 Linux 操作系统中的 size 命令来验证这一点。
进一步阅读
数据段（Data Segment）
数据段分为以下两部分，通常位于堆区域之下，或在某些实现中位于栈之上，但数据段永远不会位于堆和栈区域之间。

2. 未初始化数据段（Uninitialized data segment）
这个段也称为 bss。
这是包含以下内容的内存部分：
未初始化的全局变量（包括指针变量）
未初始化的常量全局变量。
未初始化的局部静态变量。
任何未初始化的全局或静态局部变量都将存储在未初始化数据段中
例如：全局变量 int globalVar; 或静态局部变量 static int localStatic; 将存储在未初始化数据段中。
如果你声明一个全局变量并将其初始化为 0 或 NULL，那么它仍然会进入未初始化数据段或 bss。
进一步阅读
3. 已初始化数据段（Initialized data segment）
这个段存储：
已初始化的全局变量（包括指针变量）
已初始化的常量全局变量。
已初始化的局部静态变量。
例如：全局变量 int globalVar = 1; 或静态局部变量 static int localStatic = 1; 将存储在已初始化数据段中。
这个段可以进一步分类为已初始化只读区域和已初始化读写区域。已初始化的常量全局变量将进入已初始化只读区域，而值在运行时可以修改的变量将进入已初始化读写区域。
这个段的大小由程序源代码中值的大小决定，并且在运行时不会改变。
进一步阅读
4. 栈段（Stack Segment）
栈段用于存储在函数内部创建的变量（函数可以是 main 函数或用户定义函数），变量如
函数的局部变量（包括指针变量）
传递给函数的参数
返回地址
存储在栈中的变量会在函数执行结束时立即被移除。
进一步阅读
5. 堆段（Heap Segment）
这个段用于支持动态内存分配。如果程序员想要动态分配一些内存，那么在 C 中是通过 malloc、calloc 或 realloc 方法完成的。
例如，当 int* ptr = malloc(sizeof(int) * 2) 时，堆中会分配 8 个字节，该位置的内存地址会被返回并存储在 ptr 变量中。ptr 变量取决于其声明/使用方式，可能在栈上或数据段中。

---

## 相关页面

- [[Common_embedded_interview]]
- [[Concept_questions]]
- [[CAN_interview_questions]]
- [[I2C_interview_questions]]
- [[prepare]]

返回索引 [[00-索引]]
