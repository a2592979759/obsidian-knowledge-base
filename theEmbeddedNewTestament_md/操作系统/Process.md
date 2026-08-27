---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Process/Process.md
created: 2026-08-27
---

## OS 中的进程管理(Process Management)

[***Linux 进程(Linux Processes)***](https://www.bogotobogo.com/Linux/linux_process_and_signals.php)

这篇来自 [bogotobogo.com](bogotobogo.com) 的博客详细描述了 Linux 操作系统中的进程。强烈推荐。

### 进程内容(Process Contents)

[***进程、地址空间与上下文切换(Processes, Address Spaces, and Context Switches)***](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/6_Processes.pdf)

来自 IIT 的关于进程的深刻幻灯片。

### 进程控制块(Process Control Block, PCB)

### 上下文切换(Context Switch)

上下文切换开销(Context Switching Overheads)
- 影响上下文切换时间的直接因素
  - 定时器中断延迟
  - 保存/恢复上下文
  - 找到下一个要执行的进程
- 间接因素
  - 需要重新加载 TLB
  - 缓存局部性丢失(因此产生更多缓存未命中)
  - 处理器流水线冲刷

## 资源(Resources)

[NYU CS 2250 进程管理课程资料](https://cs.nyu.edu/~gottlieb/courses/2000s/2000-01-spring/os/chapters/chapter-2.html)

[Utexas CS372 幻灯片](https://www.cs.utexas.edu/~lorenzo/corsi/cs372/03F/notes/9-9.pdf)

[操作系统笔记：进程管理](https://applied-programming.github.io/Operating-Systems-Notes/2-Process-Management/)

[操作系统学习指南：进程管理](http://faculty.salina.k-state.edu/tim/ossg/Process/process.html)

[操作系统进程管理](https://www.studytonight.com/operating-system/operating-system-processes)

[进程是如何工作的](https://www.usna.edu/Users/cs/crabbe/SI411/current/processes/processes.html)

[北佛罗里达大学幻灯片](https://www.unf.edu/public/cop4610/ree/Notes/PPT/PPT8E/CH%2003%20-OS8e.pdf)

[进程、地址空间与上下文切换 IIT Madras 幻灯片](http://www.cse.iitm.ac.in/~chester/courses/15o_os/slides/6_Processes.pdf)

[UIUC CS 241 高质量课程资料](https://courses.engr.illinois.edu/cs241/sp2012/)
