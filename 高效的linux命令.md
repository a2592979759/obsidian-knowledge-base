---
tags:
  - linux
date: 2026-08-26
title: linux系统移植命令
author: lqq
---

find . -name "arm-xilinx-linux-gnueabi-gcc"
grep -nr txt

直接单独编译uboot:
petalinux-build -c u-boot
打包成BOOT.bin:
petalinux-package --boot --fsbl --fpga --u-boot --force 

可以使用grep -nr ***进行搜索和查看
在顶层Makefile导出的ARCH CPU BOARD VENDOR SOC CPUDIR BOARDDIR   : 在config.mk中
在config.mk导出的CONFIG_SYS_ARCH  CONFIG_SYS_CPU  CONFIG_SYS_BOARD CONFIG_SYS_VENDOR CONFIG_SYS_SOC ： 在.config中


一、通过配置文件(最终生成.config)来修改uboot
编译u-boot已经具备了交叉编译环境之后
方式一：（不推荐）可以直接在顶层 Makefile 中直接给 ARCH 和 CORSS_COMPILE 赋值
方式二：（推荐）创建 shell 脚本。在 uboot 根目录下创建一个名为 zynq_zc702.sh 的 shell 脚本，然后在 shell 脚本里面输入如下内容： 
zynq_zc702.sh 文件： 
#!/bin/bash 
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- xilinx_zynq_virt_defconfig 
make V=1 ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- -j8 
给 zynq_zc702.sh 这个文件可执行权限，使用 zynq_zc702.sh 脚本编译 uboot的时候每次都会清理一下工程，然后全部重新编译，编译的时候直接执行这个脚本就行了


find -name zynq_zc702.h
在目录 include/configs 下添加启明星开发板对应的头文件。我们的启明星开发板是参考 xilinx 的 zc702开发板的硬件设计的，我们应当以 zynq_zc702.h 文件为模板进行修改，但是这里并没有 zynq_zc702.h 头文件。所以这里我们只能自定义一个 zynq_zc702.h 文件，在终端输入如下命令： 
cd include/configs/ 
vim zynq_zc702.h
修改内容:
#ifndef __CONFIG_ZYNQ_ZC70X_H 
#define __CONFIG_ZYNQ_ZC70X_H 
 
#include <configs/zynq-common.h> 
 
#endif /* __CONFIG_ZYNQ_ZC70X_H */ 


find -name zynq-common.h				
cp zynq-common.h altk-common.h 
vim altk-common.h 
修改的内容主要是第 200 行起的 CONFIG_EXTRA_ENV_SETTINGS 内容
对于根文件系统，一般都是直接放到 SD 卡的 ext4 分区，或者使用 INITRAMFS 格式，而不是通过加载 ramdisk_image 文件启动的

uboot 中每个板子都有一个对应的文件夹来存放板级文件，比如开发板上外设驱动文件等。Xilinx 的ZYNQ系列芯片的所有板级文件夹都存放在 board/xilinx/zynq目录下，在这个目录下有个名为 zynq-zc702的
文件夹，这个文件夹就是 Xilinx 官方 ZC702 开发板的板级文件夹。复制 zynq-zc702，将其重命名为 zynqaltk，命令如下：

cd board/xilinx/zynq/ 
cp -r zynq-zc702 zynq-altk 
进入 zynq-altk 目录中，可以看到只有一个名为“ps7_init_gpl.c”的文件，该文件是 PS 的初始化文件，可用于开发板。


uboot 支持设备树，每个开发板都有一个对应的设备树文件。Xilinx 的 ZYNQ 系列芯片的所有设备树文件夹都存放在 arch/arm/dts 目录下，在这个目录下有个名为 zynq-zc702.dts 的文件，该文件是 ZC702 开发板
的设备树文件。这里我们就不参照 zynq-zc702.dts 文件，而是参照 zynq-zed.dts 文件，这是因为 zynq-zed.dts是在 zynq-zc702.dts 文件基础上修改而来，能极大的方便我们的移植。我们将 zynq-zed.dts 重命名为 zynqaltk.dts，命令如下：

cd arch/arm/dts 
cp zynq-zed.dts zynq-altk.dts 

修改 arch/arm/dts 下的 Makefile
vim arch/arm/dts/Makefile
输入/zynq找到对应的部分

将 zynq-zc702.dtb 替换为 zynq-altk.dtb 或者直接添加zynq-altk.dtb？


使用./zynq_zc702.sh编译完成之后可以使用grep -nR "zynq_altk.h"进行验证，这样就可以检验是否修改完成。
文件 include/configs/altk-common.h 中的宏 CONFIG_EXTRA_ENV_SETTINGS 保存着环境变量 bootcmd 和 bootargs的默认值

总结一下 uboot 移植的过程： 
1、不管是购买的开发板还是自己做的开发板，基本都是参考半导体厂商的 dmeo 板，而半导体厂商会在他们自己的开发板上移植好 uboot、linux kernel 和 systemfs 等，最终制作好 BSP 包提供给用户。我们可以在官方提供的 BSP 包的基础上添加我们的板子，也就是俗称的移植。 
2、我们购买的开发板或者自己做的板子一般都不会原封不动的照抄半导体厂商的 demo 板，都会根据实际的情况来做修改，既然有修改就必然涉及到 uboot 下驱动的移植。 
3、一般 uboot 中需要解决串口、QSPI、EMMC 或 SD 卡、网络和 LCD 驱动，因为 uboot 的主要目的就是启动 Linux 内核，所以不需要考虑太多的外设驱动。 
4、在 uboot 中添加自己的板子信息，根据自己板子的实际情况来修改 uboot 中的驱动。
二、通过图形化配置(最终生成.config)来修改uboot
在uboot根目录下使用：make menuconfig
要添加一个驱动
则cd到对应目录，然后参考类似的.c文件进行Kconfig以及代码的添加
比如使用:grep -nr ***来对比添加




Linux内核
便捷编译
新建名为“zynq.sh”的 shell 脚本，然后在这个shell 脚本里面输入如下所示内容： 
示例代码 zynq.sh 文件内容 
 1 #!/bin/sh 
 2 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
 3 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- xilinx_zynq_defconfig 
 4 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- menuconfig 
 5 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- all -j8
 
Linux 内核最终是需要和根文件系统打交道的，需要挂载根文件系
统，并且执行根文件系统中的 init 程序
 

内核移植
修改顶层 Makefile，直接在顶层 Makefile 文件里面定义 ARCH 和 CROSS_COMPILE 这两个的变量值为
arm 和 arm-xilinx-linux-gnueabi-

在编译 Linux 内核之前要先配置 Linux 内核。每个板子都有其对应的默认配置文件，这些默认配置文件保存在 arch/arm/configs 目录中。xilinx_zynq_defconfig 作为 ZYNQ EVK 开发板所使用的默认配置文件。 
进入到 Ubuntu 中的 Linux 源码根目录下，执行如下命令配置 Linux 内核： 
make clean //第一次编译 Linux 内核之前先清理一下 
make xilinx_zynq_defconfig //配置 Linux 内核 
make -j8 //编译 Linux 内核
Linux 内核编译完成以后会在 arch/arm/boot 目录下生成 zImage 镜像文件，如果使用设备树的话还会在
arch/arm/boot/dts 目录下生成开发板对应的.dtb(设备树)文件，比如 zynq-zc702.dtb 就是 Xilinx 官方的ZYNQ 
ZC702 EVK 开发板对应的设备树文件。至此我们得到两个文件： 
① Linux 内核镜像文件：zImage。 
② Xilinx 官方 ZYNQ ZC702 EVK 开发板对应的设备树文件：zynq-zc702.dtb。


将 arch/arm/configs 目录下的 xilinx_zynq_defconfig 重新复制一份，命名为 alientek_zynq_defconfig，命令如下： 
cd arch/arm/configs 
cp xilinx_zynq_defconfig alientek_zynq_defconfig 
使用如下命令来配置 Linux 内核： 
make alientek_zynq_defconfig 

cd arch/arm/boot/dts 
cp zynq-zed.dts zynq-alientek.dts 

zynq-alientek.dts 修改好以后还需要修改arch/arm/boot/dts/Makefile， 
找 到 “ dtb-$(CONFIG_ARCH_ZYNQ)”配置项，在此配置项中加入“zynq-alientek.dtb \”
我们已经在 Linux 内核里面已经添加了开发板的配置，接下接进行编译测试。我们可以创建一个名为 zynq.sh 的编译脚本，脚本内容如下： 
1 #!/bin/sh 
2 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
3 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- alientek_zynq_defconfig 
4 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- menuconfig 
5 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- all -j8 


执行“make clean”清理工程以后.config 文件就会被删除掉，因此我们所有的配置内容都会丢失，所以要对.config的文件进行备份

简单总结一下移植步骤： 
① 在 Linux 内核中查找可以参考的板子，一般都是半导体厂商自己做的开发板。 
② 编译出参考板子对应的 zImage 和.dtb 文件。 
③ 使用参考板子的 zImage 文件和.dtb 文件在我们所使用的板子上启动 Linux 内核，看能否启动。 
④ 如果能启动的话就万事大吉，如果不能启动就需要调试 Linux 内核和修改设备树。不过一般都会参考
半导体官方的开发板设计自己的硬件，所以大部分情况下都会启动起来。启动 Linux 内核用到的外设不多，一般就 DRAM(Uboot 都初始化好的)和串口。 
⑤ 修改相应的驱动，像 USB、EMMC、SD卡等驱动官方的 Linux内核都是已经提供好了，基本不会出问题。重点是网络驱动，因为 Linux 驱动开发一般都要通过网络调试代码，所以一定要确保网络驱动工
作正常。如果是处理器内部 MAC+外部 PHY 这种网络方案的话，一般网络驱动都很好处理，因为在Linux 内核中是有外部 PHY 通用驱动的。只要设置好复位引脚、PHY 地址信息基本上都可以驱动起来。 
⑥ Linux 内核启动以后需要根文件系统，如果没有根文件系统的话肯定会崩溃，所以确定 Linux 内核移植成功以后就要开始根文件系统的构建。

仔细做好复盘笔记，归纳总结，整理出重点


Kernel文件夹中的脚本文件
kmake.sh 用于编译得到zImage和顶层的system.dtb,文件内容如下：
ake ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage -j10
cp arch/arm/boot/zImage zImage
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- system-top.dtb -j10
cp arch/arm/boot/dts/system-top.dtb system.dtb

Kernel文件夹中的文件
kset.sh 用于打开menuconfig进行图形化的配置,文件内容如下：
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

Kernel文件夹中的文件
mmake.sh 用于在顶层编译出模块文件.ko,文件内容如下：
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules -j16
可以修改其中的交叉编译环境变量

使用Device Tree Compiler(dtc)将生成的dtb文件转换为dts
dtc -I dtb -O dts -o system.dts system.dtb
将dts转换为dtb
dtc -I dts -O dtb -o systemTest.dtb system.dts

通过调用stat 可以查看file的属性信息
stat filename
  File: fork
  Size: 8520      	Blocks: 24         IO Block: 4096   regular file
Device: 804h/2052d	Inode: 2246710     Links: 1
Access: (0775/-rwxrwxr-x)  Uid: ( 1001/    book)   Gid: ( 1001/    book)
Access: 2025-02-25 01:40:12.536492225 -0500
Modify: 2025-02-25 01:40:01.636491985 -0500
Change: 2025-02-25 01:40:01.636491985 -0500
Birth: -
find . -name "arm-xilinx-linux-gnueabi-gcc"
grep -nr txt


直接单独编译uboot:
petalinux-build -c u-boot
打包成BOOT.bin:
petalinux-package --boot --fsbl --fpga --u-boot --force 

可以使用grep -nr ***进行搜索和查看
在顶层Makefile导出的ARCH CPU BOARD VENDOR SOC CPUDIR BOARDDIR   : 在config.mk中
在config.mk导出的CONFIG_SYS_ARCH  CONFIG_SYS_CPU  CONFIG_SYS_BOARD CONFIG_SYS_VENDOR CONFIG_SYS_SOC ： 在.config中


一、通过配置文件(最终生成.config)来修改uboot
编译u-boot已经具备了交叉编译环境之后
方式一：（不推荐）可以直接在顶层 Makefile 中直接给 ARCH 和 CORSS_COMPILE 赋值
方式二：（推荐）创建 shell 脚本。在 uboot 根目录下创建一个名为 zynq_zc702.sh 的 shell 脚本，然后在 shell 脚本里面输入如下内容： 
zynq_zc702.sh 文件： 
#!/bin/bash 
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- xilinx_zynq_virt_defconfig 
make V=1 ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- -j8 
给 zynq_zc702.sh 这个文件可执行权限，使用 zynq_zc702.sh 脚本编译 uboot的时候每次都会清理一下工程，然后全部重新编译，编译的时候直接执行这个脚本就行了


find -name zynq_zc702.h
在目录 include/configs 下添加启明星开发板对应的头文件。我们的启明星开发板是参考 xilinx 的 zc702开发板的硬件设计的，我们应当以 zynq_zc702.h 文件为模板进行修改，但是这里并没有 zynq_zc702.h 头文件。所以这里我们只能自定义一个 zynq_zc702.h 文件，在终端输入如下命令： 
cd include/configs/ 
vim zynq_zc702.h
修改内容:
#ifndef __CONFIG_ZYNQ_ZC70X_H 
#define __CONFIG_ZYNQ_ZC70X_H 
 
#include <configs/zynq-common.h> 
 
#endif /* __CONFIG_ZYNQ_ZC70X_H */ 


find -name zynq-common.h				
cp zynq-common.h altk-common.h 
vim altk-common.h 
修改的内容主要是第 200 行起的 CONFIG_EXTRA_ENV_SETTINGS 内容
对于根文件系统，一般都是直接放到 SD 卡的 ext4 分区，或者使用 INITRAMFS 格式，而不是通过加载 ramdisk_image 文件启动的

uboot 中每个板子都有一个对应的文件夹来存放板级文件，比如开发板上外设驱动文件等。Xilinx 的ZYNQ系列芯片的所有板级文件夹都存放在 board/xilinx/zynq目录下，在这个目录下有个名为 zynq-zc702的
文件夹，这个文件夹就是 Xilinx 官方 ZC702 开发板的板级文件夹。复制 zynq-zc702，将其重命名为 zynqaltk，命令如下：

cd board/xilinx/zynq/ 
cp -r zynq-zc702 zynq-altk 
进入 zynq-altk 目录中，可以看到只有一个名为“ps7_init_gpl.c”的文件，该文件是 PS 的初始化文件，可用于开发板。


uboot 支持设备树，每个开发板都有一个对应的设备树文件。Xilinx 的 ZYNQ 系列芯片的所有设备树文件夹都存放在 arch/arm/dts 目录下，在这个目录下有个名为 zynq-zc702.dts 的文件，该文件是 ZC702 开发板
的设备树文件。这里我们就不参照 zynq-zc702.dts 文件，而是参照 zynq-zed.dts 文件，这是因为 zynq-zed.dts是在 zynq-zc702.dts 文件基础上修改而来，能极大的方便我们的移植。我们将 zynq-zed.dts 重命名为 zynqaltk.dts，命令如下：

cd arch/arm/dts 
cp zynq-zed.dts zynq-altk.dts 

修改 arch/arm/dts 下的 Makefile
vim arch/arm/dts/Makefile
输入/zynq找到对应的部分

将 zynq-zc702.dtb 替换为 zynq-altk.dtb 或者直接添加zynq-altk.dtb？


使用./zynq_zc702.sh编译完成之后可以使用grep -nR "zynq_altk.h"进行验证，这样就可以检验是否修改完成。
文件 include/configs/altk-common.h 中的宏 CONFIG_EXTRA_ENV_SETTINGS 保存着环境变量 bootcmd 和 bootargs的默认值

总结一下 uboot 移植的过程： 
1、不管是购买的开发板还是自己做的开发板，基本都是参考半导体厂商的 dmeo 板，而半导体厂商会在他们自己的开发板上移植好 uboot、linux kernel 和 systemfs 等，最终制作好 BSP 包提供给用户。我们可以在官方提供的 BSP 包的基础上添加我们的板子，也就是俗称的移植。 
2、我们购买的开发板或者自己做的板子一般都不会原封不动的照抄半导体厂商的 demo 板，都会根据实际的情况来做修改，既然有修改就必然涉及到 uboot 下驱动的移植。 
3、一般 uboot 中需要解决串口、QSPI、EMMC 或 SD 卡、网络和 LCD 驱动，因为 uboot 的主要目的就是启动 Linux 内核，所以不需要考虑太多的外设驱动。 
4、在 uboot 中添加自己的板子信息，根据自己板子的实际情况来修改 uboot 中的驱动。
二、通过图形化配置(最终生成.config)来修改uboot
在uboot根目录下使用：make menuconfig
要添加一个驱动
则cd到对应目录，然后参考类似的.c文件进行Kconfig以及代码的添加
比如使用:grep -nr ***来对比添加




Linux内核
便捷编译
新建名为“zynq.sh”的 shell 脚本，然后在这个shell 脚本里面输入如下所示内容： 
示例代码 zynq.sh 文件内容 
 1 #!/bin/sh 
 2 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
 3 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- xilinx_zynq_defconfig 
 4 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- menuconfig 
 5 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- all -j8
 
Linux 内核最终是需要和根文件系统打交道的，需要挂载根文件系
统，并且执行根文件系统中的 init 程序
 

内核移植
修改顶层 Makefile，直接在顶层 Makefile 文件里面定义 ARCH 和 CROSS_COMPILE 这两个的变量值为
arm 和 arm-xilinx-linux-gnueabi-

在编译 Linux 内核之前要先配置 Linux 内核。每个板子都有其对应的默认配置文件，这些默认配置文件保存在 arch/arm/configs 目录中。xilinx_zynq_defconfig 作为 ZYNQ EVK 开发板所使用的默认配置文件。 
进入到 Ubuntu 中的 Linux 源码根目录下，执行如下命令配置 Linux 内核： 
make clean //第一次编译 Linux 内核之前先清理一下 
make xilinx_zynq_defconfig //配置 Linux 内核 
make -j8 //编译 Linux 内核
Linux 内核编译完成以后会在 arch/arm/boot 目录下生成 zImage 镜像文件，如果使用设备树的话还会在
arch/arm/boot/dts 目录下生成开发板对应的.dtb(设备树)文件，比如 zynq-zc702.dtb 就是 Xilinx 官方的ZYNQ 
ZC702 EVK 开发板对应的设备树文件。至此我们得到两个文件： 
① Linux 内核镜像文件：zImage。 
② Xilinx 官方 ZYNQ ZC702 EVK 开发板对应的设备树文件：zynq-zc702.dtb。


将 arch/arm/configs 目录下的 xilinx_zynq_defconfig 重新复制一份，命名为 alientek_zynq_defconfig，命令如下： 
cd arch/arm/configs 
cp xilinx_zynq_defconfig alientek_zynq_defconfig 
使用如下命令来配置 Linux 内核： 
make alientek_zynq_defconfig 

cd arch/arm/boot/dts 
cp zynq-zed.dts zynq-alientek.dts 


zynq-alientek.dts 修改好以后还需要修改arch/arm/boot/dts/Makefile， 
找 到 “ dtb-$(CONFIG_ARCH_ZYNQ)”配置项，在此配置项中加入“zynq-alientek.dtb \”
我们已经在 Linux 内核里面已经添加了开发板的配置，接下接进行编译测试。我们可以创建一个名为 zynq.sh 的编译脚本，脚本内容如下： 
1 #!/bin/sh 
2 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- distclean 
3 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- alientek_zynq_defconfig 
4 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- menuconfig 
5 make ARCH=arm CROSS_COMPILE=arm-xilinx-linux-gnueabi- all -j8 


执行“make clean”清理工程以后.config 文件就会被删除掉，因此我们所有的配置内容都会丢失，所以要对.config的文件进行备份

简单总结一下移植步骤： 
① 在 Linux 内核中查找可以参考的板子，一般都是半导体厂商自己做的开发板。 
② 编译出参考板子对应的 zImage 和.dtb 文件。 
③ 使用参考板子的 zImage 文件和.dtb 文件在我们所使用的板子上启动 Linux 内核，看能否启动。 
④ 如果能启动的话就万事大吉，如果不能启动就需要调试 Linux 内核和修改设备树。不过一般都会参考
半导体官方的开发板设计自己的硬件，所以大部分情况下都会启动起来。启动 Linux 内核用到的外设不多，一般就 DRAM(Uboot 都初始化好的)和串口。 
⑤ 修改相应的驱动，像 USB、EMMC、SD卡等驱动官方的 Linux内核都是已经提供好了，基本不会出问题。重点是网络驱动，因为 Linux 驱动开发一般都要通过网络调试代码，所以一定要确保网络驱动工
作正常。如果是处理器内部 MAC+外部 PHY 这种网络方案的话，一般网络驱动都很好处理，因为在Linux 内核中是有外部 PHY 通用驱动的。只要设置好复位引脚、PHY 地址信息基本上都可以驱动起来。 
⑥ Linux 内核启动以后需要根文件系统，如果没有根文件系统的话肯定会崩溃，所以确定 Linux 内核移植成功以后就要开始根文件系统的构建。

仔细做好复盘笔记，归纳总结，整理出重点


Kernel文件夹中的脚本文件
kmake.sh 用于编译得到zImage和顶层的system.dtb,文件内容如下：
ake ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage -j10
cp arch/arm/boot/zImage zImage
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- system-top.dtb -j10
cp arch/arm/boot/dts/system-top.dtb system.dtb

Kernel文件夹中的文件
kset.sh 用于打开menuconfig进行图形化的配置,文件内容如下：
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

Kernel文件夹中的文件
mmake.sh 用于在顶层编译出模块文件.ko,文件内容如下：
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules -j16
可以修改其中的交叉编译环境变量

使用Device Tree Compiler(dtc)将生成的dtb文件转换为dts
dtc -I dtb -O dts -o system.dts system.dtb
将dts转换为dtb
dtc -I dts -O dtb -o systemTest.dtb system.dts

通过调用stat 可以查看file的属性信息
stat filename
  File: fork
  Size: 8520      	Blocks: 24         IO Block: 4096   regular file
Device: 804h/2052d	Inode: 2246710     Links: 1
Access: (0775/-rwxrwxr-x)  Uid: ( 1001/    book)   Gid: ( 1001/    book)
Access: 2025-02-25 01:40:12.536492225 -0500
Modify: 2025-02-25 01:40:01.636491985 -0500
Change: 2025-02-25 01:40:01.636491985 -0500
Birth: -
