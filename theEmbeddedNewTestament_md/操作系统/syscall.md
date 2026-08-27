---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Linux/syscall.md
created: 2026-08-27
---

## Linux 系统调用(system call)机制

### 为什么我们需要系统调用?

系统调用是用户空间应用程序用来请求操作系统内核服务的方法。这是因为用户空间应用程序由于权限差异不能直接访问操作系统内核中的资源。

- 系统调用基本上是用户应用程序与操作系统内核服务交互的 API。
- 可请求的操作系统内核服务包括: 设备 I/O、进程创建、硬件访问、内存分配等等。
- 系统调用会产生_软件中断(software interrupt)_，使 CPU 从__用户模式(user mode)__转换到__内核模式(kernel mode)__。每个系统调用都有各自的系统调用编号，以便在软件中断的 ISR 例程中处理。
- 系统调用的存在是为了保护内核空间，使用户空间应用程序无法直接干扰系统资源，防止恶意尝试修改或破坏系统。

系统调用分层图:

![[_assets/syscall.png]]

用户空间进程请求操作系统服务:

![[_assets/syscall_2.png]]

### 以 kill() 系统调用为例:

- 用户空间方法 XXX，其对应的系统调用层方法是 sys_XXX。例如 kill() -> sys_kill()。
- unistd.h (_/kernel/include/uapi/asm-generic/unistd.h_) 包含所有系统调用软件中断编号信息。
- 

### 参考(Reference)

https://www.slideshare.net/VandanaSalve/introduction-to-char-device-driver

https://www.slideshare.net/garyyeh165/linux-char-device-driver 

https://www.ptt.cc/bbs/b97902HW/M.1268953601.A.BA9.html

https://www.ptt.cc/bbs/b97902HW/M.1268932130.A.0CF.html

http://gityuan.com/2016/05/21/syscall/

http://hwchiu.logdown.com/posts/1733-c-pipe

http://wiki.csie.ncku.edu.tw/embedded/ARMv8

http://linux.vbird.org/linux_basic/0440processcontrol.php

_Advanced Programming in the UNIX Environment 3rd Edition_
