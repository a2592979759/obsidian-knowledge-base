---
tags:
  - uboot
  - command
author: lqq
---
uboot启动之后的命令解析
~~~bash
help / ? 		帮助命令
help + 其他 	查看命令的使用说明
bdinfo 			查看板子信息
printenv / pri  查看当前板子的环境变量
setenv		    设置环境变量,如果环境变量不存在则自动创建,存在则修改替换设置的环境变量 设置字符串形式的环境变量最好使用''最大程度抑制展开,setenv bootdelay 3,setenv bootargs " " or ' '
saveenv         保存设置的环境变量
uboot内存操作命令
uboot 网络操作命令
ipaddr 开发板ip地址，可以使用dhcp命令从路由器获取IP地址
ethaddr 开发板的mac地址，一定要设置:setenv ethaddr "00:11:22:33:44:55"
gatewayip 网关地址
netmask   子网掩码
serverip  服务器地址，既主机地址用于调试代码
dhcp 用于自动获取ip
ping 用于测试网络
网络文件系统nfs
简单文件传输tftp(在配置好ip之后)
tftpboot 0x10000000 zImage 将zImage加载到内存0x10000000处的位置,后续需要使用则在0x10000000处的位置取用

uboot mmc操作指令
mmc info 查看mmc信息
mmc rescan 重新加载mmc设备
mmc list   查看mmc设备
FSL_SDHC:0
FSL_SDHC:1(eMMc)
mmc dev 切换为dev 0
mmc part 查看mmc分区情况
mmc dev 1 0 切换为dev1的分区0
mmc read 0x80800000 600 10 读取分区中第600个块开始的10个块的数据到DDR 0x80800000地址

在线更新uboot 
mmc dev 1 0 切换到emmc的第0分区
tftp 0x80800000 u-boot.imx 从tftp下载u-boot.imx到内存0x80800000地址
mmc write 0x80800000 2 2c6（下载的uboot大小） 将内存的数据写到emmc的第0分区第2个块
mmc partconf 1 1 0 0 分区设置
reset 重启

fat文件系统操作
fstype查询分区的文件系统格式
fstype mmc 1:0
fstype mmc 1:1				查询0 1 2个分区信息
fstype mmc 1:2
fatinfo查询分区的文件系统信息
fatinfo mmc 1:1 查询emmc分区1的文件系统信息
fatls 查询目录和文件
fatls mmc 1:1 查询emmc分区1中所有的目录和文件 
fatload mmc 1:1 0x10000000 zImage 读文件到Dram中(该文件已经位于emmc中)
fatwrite mmc 1:1 0x10000000 zImage 将数据写到文件中

EXT格式文件系统操作命令

常用与boot操作相关的命令有
bootz、bootm、bootelf、boot、bootvx
1、通过网络启动
tftp 0x10000000 zImage
bootz 0x10000000
2、通过emmc启动
首先必须确定emmc中存在该文件
fatls mmc 1:1
fatload mmc 1:1 0x10000000 zImage
bootz 0x10000000
启动uImage镜像文件,bootm

boot其实是读取环境变量bootcmd来启动linux系统,bootcmd环境变量中保存着引导命令，是启动的命令集合,相当于在倒计时后自动执行boot命令加载环境变量，启动linux操作系统
setenv bootcmd "fatload mmc 1:1 0x10000000 zImage;bootz 0x10000000"			'    '最大限度的限制展开
命令之间可以使用 ;   or  &&
跳转执行 go
go addr 跳到指定的地址处执行应用
tftp 0x10000000 boot.bin 
go 0x10000000

run 用于运行环境变量中定义的命令
run bootcmd 

version 用于查看uboot的版本号

uboot命令中sf命令的使用方法
uboot中如果支持spi/qspi flash 那么可以使用sf的 erase,read,write,update等命令来操作spi flash
sf read 用来读取flash数据到内存
sf write 写内存数据到flash
sf erase 擦除指定位置，指定长度的flash内容,擦除后内容全1
具体用法：
sf probe  初始化flash设备并进行片选
sf read addr offset len	
sf write addr offset len	把内存addr处的数据写入flash的偏移offset写入数据长度为len
sf erase offset len			擦除偏移offset处到len之间的擦除块,擦除操作是以erase block为单位的，要求offset和len参数必须是erase block对齐的
sf update addr offset len  从addr开始偏移offset的位置，擦除并写len的byte
在使用sf read、sf write、sf update之前一定要调用sf probe
从sf命令,可以看出几点
1、spi flash 没有oob数据存在，也就是不用考虑edc eec，也没有坏块管理概念
2、支持byte级的读写操作，支持随机访问
如何验证读写效果？
结合uboot的md命令,sf read、sf write命令均涉及到内存操作,可以使用md查看内存数据
md 0x10000000 0x100 打印从0x10000000开始长度范围为256字节的内存数据;
姚太伟给的两条指令：
1、sf probe 0:0 && sf erase 0x0 0xA00000(大小由加载到内存的文件定)
sf write 0x10000000 0x0 0xA00000
2、sf probe 0:0 && sf update 0x10000000 0x0 0xA00000 

在uboot下进行文件系统分区和文件系统的格式化


在linux下建立新的文件系统步骤
fdisk创建分区
mkfs格式化分区(创建文件系统)
mount挂载文件系统
修改etc/fstab文件永久挂载文件系统

在uboot下进行操作
fatinfo mmc 0:1
会显示指定mmc设备和分区的文件系统信息
fatsformat mmc 0:1 格式化emmc的第一个分区为fat文件系统
fatload mmc 0:1 0x43000000 rootfs.gz.rar
从emmc的第一个分区(假设为根文件系统)加载rootfs.gz.rar到内存0x43000000
修改bootcmd 和bootarg环境变量
setenv bootcmd ‘fatload mmc 0:1 0x43000000 uImage ;bootm 0x43000000'
setenv bootargs 'console = ttyAMA0,115200 root=/dev/mmcblk0p1 rw'
这里bootcmd指定了从emmc的第一个分区加载uimage并启动,bootargs指定了启动参数,包括控制台输出和根文件系统的设备以及分区

姚太伟给的格式化分区指令：
1、emmc分区 fdisk "dev/sd0",70,10,10,10  //后四个参数代表四个分区的百分比
分完区后，会在/dev目录下产生4个设备文件：sd0p1、sd0p2、sd0p3、sd0p4
2、格式化：format "dosfs","/dev/sd0p1"   //格式化为dosFs文件系统
		   format "dosfs","/dev/sd0p2" 
		   format "dosfs","/dev/sd0p3" 
		   format "dosfs","/dev/sd0p4" 
或者	   format "dosfs","/dev/sd0p1"	//格式化成高可靠性hrFs文件系统
3、挂载    mount  "dosfs","/dev/sd0p1","sdcard0"
		   mount  "dosfs","/dev/sd0p2","sdcard1"
		   mount  "dosfs","/dev/sd0p3","sdcard2"
		   mount  "dosfs","/dev/sd0p4","sdcard3"
		   
kimi：
1、卸载emmc(如果已挂载)
umount /dev/mmcblk0*
2、使用fdisk创建分区
fdisk /dev/mmcblk0*
help fdisk
3、根据需要的文件系统类型使用相应的命令来格式化分区
FAT32分区
mkdosfs -F 32 /dev/mmcblk0p1
ext4分区
mkfs.ext4 /dev/mmcblk0p1
4、挂载文件系统
mkdir /mnt/emmc
mount /dev/mmcblk0p1/mnt/emmc
将Linux根文件系统解压到emmc分区根目录
tar -xvf rootfs.tar.gz -C /mnt/emmc
6、同步文件
sync		   
		   
7z030		   
~~~
