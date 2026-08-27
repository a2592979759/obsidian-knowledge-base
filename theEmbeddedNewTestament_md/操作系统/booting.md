---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Linux/booting.md
created: 2026-08-27
---

## Linux 启动过程(Booting Process)

![[_assets/linux-boot-process.png]]

### 1. BIOS

* BIOS 代表基本输入输出系统(Basic Input/Output System)
* 执行一些系统完整性检查
* 搜索、加载并执行引导加载程序(boot loader)。
* 它会在软盘、光盘或硬盘中寻找引导加载程序。你可以在 BIOS 启动期间按键(通常是 F12 或 F2，取决于你的系统)来改变启动顺序。
* 一旦检测到引导加载程序并将其加载到内存中，BIOS 就将控制权交给它。

所以，简单来说，BIOS 加载并执行 MBR 引导加载程序。

### 2. MBR

* MBR 代表主引导记录(Master Boot Record)。
* 它位于可启动磁盘的第 1 个扇区。通常是 /dev/hda 或 /dev/sda
* MBR 大小小于 512 字节。它有三个组成部分: 1) 前 446 字节的主引导加载程序信息 2) 接下来 64 字节的分区表信息 3) 最后 2 字节的 mbr 校验。
* 它包含关于 GRUB(或旧系统中的 LILO)的信息。

所以，简单来说，MBR 加载并执行 GRUB 引导加载程序。

### 3. GRUB
```shell
#boot=/dev/sda
default=0
timeout=5
splashimage=(hd0,0)/boot/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.18-194.el5PAE)
          root (hd0,0)
          kernel /boot/vmlinuz-2.6.18-194.el5PAE ro root=LABEL=/
          initrd /boot/initrd-2.6.18-194.el5PAE.img
```
* GRUB 代表大统一引导加载程序(Grand Unified Bootloader)。
* 如果你的系统安装了多个内核镜像，你可以选择执行哪一个。
* GRUB 显示一个启动画面，等待几秒，如果你不输入任何内容，它就加载 grub 配置文件中指定的默认内核镜像。
* GRUB 了解文件系统(较旧的 Linux 加载程序 LILO 不理解文件系统)。
* Grub 配置文件是 /boot/grub/grub.conf(/etc/grub.conf 是指向它的链接)。以下是 CentOS 的 grub.conf 示例。
* 如你从上文中所见，它包含内核和 initrd 镜像。

所以，简单来说，GRUB 只是加载并执行内核(Kernel)和 initrd 镜像。

### 4. 内核(Kernel)

* 挂载 grub.conf 中 "root=" 指定的根文件系统
* 内核执行 /sbin/init 程序
* 由于 init 是 Linux 内核执行的第一个程序，它的进程 ID(PID)为 1。执行 'ps -ef | grep init' 并检查其 pid。
* initrd 代表初始 RAM 盘(Initial RAM Disk)。
* initrd 被内核用作临时根文件系统，直到内核启动完成并挂载真正的根文件系统。它还包含编译在内的必要驱动，用于访问硬盘分区和其他硬件。

### 5. Init

* 查看 /etc/inittab 文件以决定 Linux 运行级别(run level)。
* 以下是可用的运行级别
```
    0 – 停机(halt)
    1 – 单用户模式(Single user mode)
    2 – 多用户，无 NFS
    3 – 完整多用户模式
    4 – 未使用
    5 – X11
    6 – 重启(reboot)
```
* Init 从 /etc/inittab 中识别默认的初始级别，并使用它加载所有合适的程序。
* 在你的系统上执行 'grep initdefault /etc/inittab' 以识别默认运行级别
* 如果你想惹上麻烦，可以把默认运行级别设为 0 或 6。既然你知道 0 和 6 意味着什么，可能你不会那样做。
* 通常你会把默认运行级别设为 3 或 5。

### 6. 运行级别程序(Runlevel programs)

* 当 Linux 系统启动时，你可能会看到各种服务正在启动。例如，它可能会显示 "starting sendmail …. OK"。那些就是运行级别程序，从你的运行级别所定义的运行级别目录中执行。
* 根据你的默认初始级别设置，系统将执行以下目录之一中的程序。
```
运行级别 0 – /etc/rc.d/rc0.d/
运行级别 1 – /etc/rc.d/rc1.d/
运行级别 2 – /etc/rc.d/rc2.d/
运行级别 3 – /etc/rc.d/rc3.d/
运行级别 4 – /etc/rc.d/rc4.d/
运行级别 5 – /etc/rc.d/rc5.d/
运行级别 6 – /etc/rc.d/rc6.d/
```
* 请注意，在 /etc 下也有这些目录的符号链接。所以，/etc/rc0.d 链接到 /etc/rc.d/rc0.d。
* 在 /etc/rc.d/rc*.d/ 目录下，你会看到以 S 和 K 开头的程序。
* 以 S 开头的程序在启动期间使用。S 代表 startup(启动)。
* 以 K 开头的程序在关机期间使用。K 代表 kill(终止)。
* 在程序名中 S 和 K 旁边有数字。这些是程序应该被启动或终止的序号。
* 例如，S12syslog 是启动 syslog 守护进程，其序号为 12。S80sendmail 是启动 sendmail 守护进程，其序号为 80。所以，syslog 程序会在 sendmail 之前启动。

以上就是 Linux 启动过程中发生的事情。

## [Linux 启动过程的所有阶段](https://github.com/nu11secur1ty/All-Stages-of-Linux-Booting-Process-)

这个 GitHub 仓库提供了更多关于 Linux 启动过程的细节。

## 参考(Reference)

https://leetcode.com/discuss/interview-question/124638/what-happens-in-the-background-from-the-time-you-press-the-Power-button-until-the-Linux-login-prompt-appears/
