---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Company/lyft.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用社区排名的嵌入式题库、带 AI 反馈的编码练习以及系统设计指南进行准备。
>
> 👉 **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)** &nbsp;·&nbsp; **[探索面试准备 →](https://embeddedinterviewlab.com/interview?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_company)**

---

### Lyft Level 5 面试题（Lyft Level 5 questions）

```免责声明：所有信息都来自公开的在线资源！```

### 电话面试（Phone screen）

**问题：中断（Interrupts）是嵌入式系统的重要组成部分。因此，许多编译器厂商为标准 C 提供扩展以支持中断。通常，这个新关键字是 __interrupt。下面的代码使用 __interrupt 来定义一个中断服务例程。请点评这段代码。**
```
    __interrupt double compute_area(double radius) {

        double area = PI * radius * radius;
        printf(“nArea = %f”, area);

        return area;
    }
```
**答案（Answer）：**

这个函数问题太多，几乎很难知道该从何处说起。

* 中断服务例程（ISR, Interrupt Service Routine）不能返回值。如果你不明白这一点，那你不会被录用。

* ISR 不能传递参数。如果你漏掉了这一点，你的就业前景请参考第 (a) 条。

* 在许多处理器/编译器上，浮点运算（floating point operations）不一定可重入（re-entrant）。在某些情况下需要压入额外的寄存器，在另一些情况下则根本无法在 ISR 中做浮点运算。此外，鉴于一般经验法则是 ISR 应该短小精悍，人们不禁怀疑在这里做浮点数学运算是否明智。

* 与第 (c) 点类似，printf() 经常在可重入性（reentrancy）和性能上出问题。如果你漏掉了第 (c) 与第 (d) 点，我不会太苛责你。不用说，如果你答对这两点，你的就业前景会越来越好。

**__interrupt**
7.5.4 __interrupt 关键字
编译器通过添加 __interrupt 关键字扩展 C/C++ 语言，该关键字指定某个函数被当作中断函数处理。此关键字表示一个 IRQ 中断。替代关键字 "interrupt" 也可使用，但在严格的 ANSI C 或 C++ 模式下除外。

请注意，第 7.9.21 节描述的中断函数属性是声明中断函数的推荐语法。

处理中断的函数遵循特殊的寄存器保存规则和特殊的返回序列。实现强调安全性。中断例程并不假定各种 CPU 寄存器和状态位的 C 运行时约定正在生效；相反，它会重新建立运行时环境所假定的任何值。当 C/C++ 代码被中断时，中断例程必须保留该例程或该例程调用的任何函数所使用的所有机器寄存器的内容。当你对函数定义使用 __interrupt 关键字时，编译器会依据中断函数的规则和中断的特定返回序列生成寄存器保存。

你只能将 __interrupt 关键字用于定义为返回 void 且没有参数的函数。中断函数的函数体可以有局部变量，并且可以自由使用栈（stack）或全局变量。例如：

__interrupt void int_handler() { unsigned int flags; ... }
名称 c_int00 是 C/C++ 入口点。此名称保留给系统复位中断（system reset interrupt）。这个特殊的中断例程初始化系统并调用 main() 函数。因为它没有调用者，c_int00 不保存任何寄存器。

注意

Hwi 对象与 __interrupt 关键字
当 SYS/BIOS Hwi 对象与 C 函数结合使用时，绝不能使用 __interrupt 关键字。Hwi_enter/Hwi_exit 宏和 Hwi 分派器（dispatcher）已经包含此功能，使用 C 修饰符可能导致不期望的冲突。

```c++
/*
实现一个函数，用于控制加热器（heater）的状态，以在 2 度迟滞（hysteresis）范围内达到期望温度。迟滞用于防止加热器过度切换，即：如果加热器已经开启，它应保持开启，直到当前温度达到期望温度 + 2；如果加热器已经关闭，它应保持关闭，直到当前温度达到期望温度 - 2。

示例）
初始时，期望温度为 70，当前温度为 65，因此加热器开启。当前温度将开始上升。加热器应保持开启，直到当前温度达到 71。

加热器初始为关闭状态。

参数：
setTemp - 设定温度（set temperature）
curTemp - 当前温度（current temperature）

返回：
一个布尔值，指示加热器是否应处于激活状态
*/

#include <stdint.h>
#include <iostream>

using namespace std;

bool isHeatingRequired(int16_t setTemp, int16_t curTemp, int hyter){

    static bool curr_state = false; // true, 68, 70
    int16_t low_b = (setTemp - hyter) > 0 ? setTemp - hyter : 0;
    int16_t up_b = (setTemp + hyter) <= 0x7FFF ? setTemp + hyter : 0x7FFF;
  
  
    // 编写实现代码
    switch(curr_state) { 
      case false: 
        if (curTemp < low_b)
          curr_state = true;
        else
          curr_state = false;
        break;
      case true:
        if (curTemp > up_b)
          curr_state = false;
        else 
          curr_state = true;
        break;
      default:
        break;
    }

    if (curr_state == true)
      std::cout << "Keep heating" << endl;
  
    return curr_state; 
}

void setHeaterState(bool b) {};

int main(){
    int16_t currTemp = 70;
    bool heatingRequired = isHeatingRequired(68, currTemp);
    setHeaterState(heatingRequired);

    return 0;
}
```
## 其他被问到的题目（来自多个来源）（Other questions asked）
#### 热身题（Warm up question）：
```c++
class Stream:
    def read(self, n: int) -> String:
        ...

你可以对 stream 调用 read()，它会返回一个大小为 n 的字符串。

调用可能返回比 n 短的字符串，此时 stream 中的字符串已被完全消费。

示例：
s = Stream("Hello World!")
s.read(4)  # => 返回 "Hell"
s.read(3)  # => 返回 "o W"
s.read(4)  # => 返回 "orld"
s.read(5)  # => 返回 "!" - stream 已完全耗尽
s.read(3)  # => 返回 "" - stream 已完全耗尽
```
#### 第 2 部分问题（Part 2 Problem）：
```c++
创建一个名为 MultiStream 的新类，它可以存储其他 stream，并拥有一个由其他 stream 支撑的 read() 方法。

将 Stream 对象视为黑盒（black boxes）——你不能修改它们的实现。

class MultiStream:
    def read(self, n: int) -> String:
        ...
    def add(self, stream: Stream) -> None:
        ...
    def remove(self, stream: Stream) -> None:
        ...

stream1: "ABCDE"
stream2: "12345"

Multisteam ms

ms.add(s1)
ms.add(s2)
ms.read(3)->"ABC" 
ms.read(5)->"DE123"
ms.read(6)->"45"
ms.read(3)->""
s1="ABCDE"
s2="12345"
s3="abcd"
"ABCDE12345abcd"
ms.read(4)->ABCD
ms.remove(s2) 
ms.read(5)->"Eabcd"
示例
ms.read(4)->ABCD
ms.remove(s1) 
ms.read(5)->"12345"
s1="ABCD"
s2="ABCD"
s1s2s3
remove(2)
s1s3
remove(2)
s1

```

### 解答（Solution）
```c++
#include <iostream>
#include <vector>
using namespace std;

// 要执行 C++，请定义 "int main()"
class reader{
    private:
        string s;
        int pos;
    
    public:  
        string read(int n)
        {
            if(pos == s.size())
                return "";

            int i;
            int sz = s.size();
            string ret;

            for(i = pos; i < min(pos+n,sz); i++)
                ret += s[i];
            
            pos = i;

            return ret;
        }

        reader(){}
        reader(string st, int k)
        {
            s = st;
            pos = k;
        }
};


class multiStream{
   private:
        vector<reader> ms;
        vector<bool> visited;

   public:
        string read(int n)
        {
            string ret;

            for(int i = 0; i< ms.size(); i++)
            {
                if(visited[i] == false)
                {
                    ret += ms[i].read(n-ret.size()); 
                    if(ret.size()==n)
                        return ret;
                }
            }
            return ret;
        }
  
        void addStream(string s)
        {
            ms.push_back(reader(s,0));
            visited.push_back(false);
        }
        
        void removeStream(int n)
        {
            if(n > ms.size())
            {
                return; 
            }
            visited[n] = true;
        }
};


int main() {
  // reader obj("Hello World!", 0);
  // cout<<obj.read(4)<<"\n";
  // cout<<obj.read(3)<<"\n";
  // cout<<obj.read(4)<<"\n";
  // cout<<obj.read(5)<<"\n";
  // cout<<obj.read(3)<<"\n";
  multiStream obj;
  obj.addStream("ABCDE");
  obj.addStream("12345");
  obj.addStream("abcde");
  cout<<obj.read(4)<<"\n";
  cout<<obj.read(5)<<"\n";
  obj.removeStream(1);
  cout<<obj.read(5)<<"\n";
  obj.removeStream(2);
  cout<<obj.read(3)<<"\n";
  
  
  return 0;
}
```

### 列表版本（List Version）
```c++
class Stream {
    private:
       
    public:
        vector<int> arr;
        int size;
        int pos;
    
        Stream() {}
        Stream(vector<int> new_arr) {
            arr = new_arr;
            size = new_arr.size();
            pos = 0;
        }
    
        int read(vector<int> &ret, int &n) {
            while (n-- > 0 && pos < size) {
                ret.push_back(arr[pos]);
                pos ++;
            }
            return pos == size;
        }
};

int key_hash(vector<int> v) {
    int ret;
    return ret;
}

class multiStream{
   private:
        list<Stream> streams{};
        //unordered_map<Stream, list<Stream>::iterator> umap;
        list<Stream>::iterator curr;
   public:
     multiStream() {
         curr = streams.begin();
     }
    
     vector<int> read(int n)
     {
         vector<int> ret;
         while (curr != streams.end() && n > 0) {
             int incr = curr->read(ret, n);
             if (incr)
                 ++curr;
        }
        for (auto n : ret) {
            cout << n;
        }
         cout << endl;
        return ret;
     }
  
     void addStream(vector<int> s)
     {
         Stream new_stream(s);
         streams.push_back(new_stream);
         if (!streams.empty())
             curr = streams.begin();
     }
     
     void removeStream(int s)
     {
        auto it = streams.begin();
        while (s--) {
            ++ it;
        }
        streams.erase(it);
     }
};


int main() {
  // reader obj("Hello World!", 0);
  // cout<<obj.read(4)<<"\n";
  // cout<<obj.read(3)<<"\n";
  // cout<<obj.read(4)<<"\n";
  // cout<<obj.read(5)<<"\n";
  // cout<<obj.read(3)<<"\n";
  multiStream obj;
  obj.addStream({1,2,3,4,5});
  obj.addStream({6,7,8,9,10});
  obj.addStream({1,2,3});
  vector<int> ret;
  ret = obj.read(4);
    cout << ret.size() << endl;
  ret = obj.read(5);
    cout << ret.size() << endl;
  obj.removeStream(2);
  ret = obj.read(5);
    cout << ret.size() << endl;
  /*ret = obj.read(5); 
    cout << ret.size() << endl;
  ret = obj.read(5);
    cout << ret.size() << endl;
  obj.removeStream(1);
  ret = obj.read(5);
    cout << ret.size() << endl;*/
  //obj.removeStream(2);
  //cout<<obj.read(3)<<"\n";
  
  return 0;
}
```

## 相关页面
- [[lyft]] —— Lyft
- [[amazon]] —— Amazon
- [[tesla]] —— Tesla
- [[zoox]] —— Zoox
- [[commonBehavior]] —— 常见行为面试题

返回索引 [[00-索引]]
