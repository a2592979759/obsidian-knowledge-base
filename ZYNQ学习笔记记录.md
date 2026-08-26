



# ZYNQ学习笔记记录

##  搭建Ubuntu的开发环境

**VMware Workstation 16可以使用的秘钥:**

```ZF3R0-FHED2-M80TY-8QYGC-NPKYF
ZF3R0-FHED2-M80TY-8QYGC-NPKYF
```

**踩坑：ubuntu搭建好之后，需要注意配置网络服务**

VMNet0: 桥接模式设置
wifi与虚拟机网口（VMNet0）要在同一个网段，可以ping通主机以及联网
确保虚拟机的网络配置（IP地址、子网掩码、网关）与宿主机在同一网段。
如果使用DHCP，虚拟机会自动获取一个与宿主机同一网段的IP地址。
VMNet1: NAT模式设置
适用于虚拟机需要访问外部网络，但不需要直接暴露在外部网络中的场景。
虚拟机可以访问外网，但无法与宿主机所在网段的其他设备直接通信。
VMNet8：仅主机模式
适用于虚拟机与宿主机之间需要通信，但不需要访问外部网络的场景。
虚拟机无法访问外网，也无法与其他设备通信。

**开启 Ubuntu 下的 FTP 服务** 

~~~
sudo apt-get install vsftpd 
sudo vi /etc/vsftpd.conf
确保vsftpd.conf文件中
local_enable=YES 
write_enable=YES 
然后重启ftp服务
sudo /etc/init.d/vsftpd restart 
~~~

**Windows 下 FTP 客户端安装**

[https://mobaxterm.mobatek.net/download-home-edition.html](https://mobaxterm.mobatek.net/download-home-edition.html)



**Ubuntu 和 Windows 文件本地共享**

在 Windows的 G盘里新建一个名为“share”的文件夹，然后在虚拟机设置里面打开共享文件夹，后续使用cp 命令 或 mv 命令文件到该目录即可



**Ubuntu 系统搭建 tftp 服务器**

~~~
sudo apt install tftp-hpa tftpd-hpa 
sudo mkdir -p /tftpboot 
sudo chmod 777 /tftpboot 
chmod 666 /etc/default/tftpd-hpa  	文件属性改为可读可写
将内容修改为
# /etc/default/tftpd-hpa 
TFTP_USERNAME="tftp" 
TFTP_DIRECTORY="/tftpboot" 
TFTP_ADDRESS=":69" 
TFTP_OPTIONS="-l -c -s" 
以后我们就将所有需要通过TFTP传输的文件都放到/tftpboot文件夹里面。
sudo service tftpd-hpa restart 

~~~

**Ubuntu 下 NFS 和 SSH 服务开启** 

~~~
sudo apt install nfs-kernel-server
mkdir -p workspace/nfs 
cd workspace/nfs 
pwd

使用前需要先配置 nfs，NFS 允许挂载的目录及权限在文件/etc/exports 中进行定义，使用如下命令打开
nfs 配置文件/etc/exports：
sudo vi /etc/exports 
在后面添加：
/home/sqd/workspace/nfs *(rw,sync,no_root_squash) 
*代表允许所有的网络段访问，rw 是可读写权限，sync 是文件同步写入存储器，no_root_squash 是 nfs 客户端分享目录使
用者的权限。
sudo systemctl start nfs-kernel-server.service 
显示共享的目录：
showmount -e
使用 exportfs 命令使改动生效：
sudo exportfs -rv

sudo apt install openssh-server
安装 ssh 服务，ssh 的配置文件为/etc/ssh/sshd_config，使用默认配置即可。
~~~

**MobaXterm 软件安装**

https://mobaxterm.mobatek.net/download-home-edition.html



**Petalinux的安装(要特别注意，petalinx的版本与vivaodo的版本要保持一致)**

Ubuntu版本：

~~~c
Ubuntu 18.04 64 
Ubuntu 16.04.4 64 
~~~

安装依赖库和软件

2020.2详情参考启明星v2.0的pdf文档,本文档说明的为2018.3的安装

2020.2：

~~~c
sudo apt-get install iproute2 gawk python3 python build-essential gcc git make net-tools libncurses5-dev tftpd zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget git-core diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib automake zlib1g:i386 screen pax gzip cpio python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 
~~~

除了上面使用命令的方法安装依赖库，也可以使用 Xilinx提供的脚本 plnx-env-setup.sh安装。该脚本可 以从[https://www.xilinx.com/support/answers/73296.html]( https://www.xilinx.com/support/answers/73296.html ) 处下载

2018.3：

~~~
sudo apt-get install tofrodos iproute2 gawk gcc g++ git make net-tools libncurses5-dev tftpd zlib1g:i386 libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip automake
~~~

修改 bash：

Petalinux 工具需要主机系统的/bin/sh 是 bash，而 Ubuntu 默认的/bin/sh 是 dash

~~~
sudo dpkg-reconfigure dash
然后选择否
~~~

Petalinux的安装路径：[https://www.amd.com/zh-cn/products/software/adaptive-socs-and-fpgas/embedded-software/petalinux-sdk.html](https://www.amd.com/zh-cn/products/software/adaptive-socs-and-fpgas/embedded-software/petalinux-sdk.html)

或者直接可以在小梅哥的路径下进行直接的下载，可以减少下载浪费的时间。

对于 Petalinux 这种体积庞大的工具，我们将其放在/opt 目录下。 在/opt 目录下新建专门存放 Petalinux 的文件夹

~~~
sudo chown -R $USER:$USER /opt 
mkdir -p /opt/Petalinux/2018.3 
sudo mv /mnt/hgfs/file/petalinux.run /opt/Petalinux/2018.3 		共享文件夹将Petalinux的安装包拿过来
./petalinux.run 
查看协议按q其余全部按y即可
等待时间较久......

设置环境变量：
是该命令只对当前终端有效，重新打开终端后需要重新执行这
一步：
source settings.sh 
验证下工作环境是否已设置：
echo $PETALINUX 

使用 echo 命令将 Petalinux 环境变量的配置命令重定向到.bashrc 配置文件中
echo "export PLNX_PATH=$PETALINUX" >> ~/.bashrc
echo "alias splnx='source \$PLNX_PATH/settings.sh'" >> ~/.bashrc
echo "export PATH=\$PATH:\$PLNX_PATH/tools/linux-i386/gcc-arm-linuxgnueabi/bin" >> ~/.bashrc  这里是交叉编译环境所在的位置

在 Ubuntu 系统中，进入到 petalinux 安装目录（opt/petalinux/2018.3）,打开终端。jtag 驱动程序存放于“tools”目录下，因此，我们输入以下命令进入到jtag 驱动程序所在位置： 
cd tools/xsct/SDK/2018.3/data/xicom/cable_drivers/lin64/install_script/install_drivers 
进入该目录后输入如下命令，安装 jtag 驱动程序：sudo ./install_drivers

~~~





## 使用Petalinux进行系统搭建

使用 Vivado 搭建好硬件平台后，通过几个命令就完成了 Linux 系统的定制，极其方便。 

设计流程不是按部就班的每一步都执行一遍，可以根据使用场景有选择的执行。

**Petalinux一 般的设计流程如下：**  

1. 通过 Vivado 创建硬件平台，得到 xsa 文件或者hdf文件； 
2. 运行 source 安装路径>/settings.sh，设置 Petalinux 运行环境 
3.  通过 petalinux-create -t project 创建 petalinux 工程；  
4. 使用 petalinux-config --get-hw-description，将 xsa 文件或者hdf文件导入到 petalinux 工程当中并配置 petalinux 工程；  
5. 使用 petalinux-config -c kernel 配置 Linux 内核； 
6.  使用 petalinux-config -c rootfs 配置 Linux 根文件系统；  
7. 配置设备树文件；  
8. 使用 petalinux-build 编译整个工程；  
9.  使用 petalinux-package --boot 制作 BOOT.BIN 启动文件；  
10.  制作 SD 启动卡，将 BOOT.BIN 和 image.ub 以及根文件系统部署到 SD 卡中；  
11. 将 SD 卡插入开发板，并将开发板启动模式设置为从 SD 卡启动；  
12. 开发板连接串口线并上电启动，串口上位机打印启动信息，登录进入 Linux 系统。

**具体实现流程**

配置环境变量

~~~
source /opt/Petalinux/2018.3/settings.sh 				//直接配置永久的环境变量 >>  ~/.bashrc
~~~

~~~
echo $PETALINUX 
显示 Petalinux 的安装目录，表明工作环境已设置。现在可以使用 Petalinux 工具了。鉴于每次打开终端使用 Petalinux 都需要设置相应的环境变量，且只能用于当前终端中。为了方便，我们使用 echo 命令将 Petalinux 环境变量的配置命令
重定向到.bashrc 配置文件中。代码如下： 
echo "export PLNX_PATH=$PETALINUX" >> ~/.bashrc 
echo "alias splnx='source \$PLNX_PATH/settings.sh'" >> ~/.bashrc 
echo "export PATH=\$PATH:\$PLNX_PATH/tools/linux-i386/gcc-arm-linuxgnueabi/bin >> ~/.bashrc 
~~~



在/opt/petalinux/2018.3/目录中创建一个名为“ALIENTEK-ZYNQ” 的 Petalinux 工程

~~~
petalinux-create -t project --template zynq -n ALIENTEK-ZYNQ
~~~

修改了Vivado 工程，重新生成 xsa 文件或者hdf后，可以重新执行“petalinux-config --get-hw-description < xsa 文件所在的位 置>”以重新配置 Petalinux 工程。

~~~ 
cd ALIENTEK-ZYNQ 
petalinux-config --get-hw-description ../hdf/
如果后面想重新配置，只需输入“petalinux-config”命令即可重新配置。
~~~

进入：Image Packaging Configuration

~~~
第一个选项便是根文件系统的类型的配置，默认为 INITRD，一般默认即可。如果我们需要运行Ubuntu 或 Debian 的根文件系统时，就需要配置成 EXT4(SD/eMMC/ SATA/USB)，NFS 挂载启动需要配置成 NFS。 
注：INITRD 类型的根文件系统每次重新启动 linux 系统都是全新的、未改动过的，也就是说启动系统后进行的所有修改掉电后就全部丢失了，再次重新启动还是之前未修改过的根文件系统，选择“EXT4”可以将根文件系统放在 SD 卡、eMMC 的 ext4 分区，这样启动系统后进行的所有修改掉电后就不会丢失了。

可以直接输入：
/INITRD 进行路径查找

注：内存文件系统（Memory File System），通常指的是一种文件系统，它将文件存储在内存中而不是传统的存储介质（如硬盘或SSD）上。这种文件系统的主要特点是速度快，因为内存的访问速度远远超过任何类型的磁盘存储。

踩坑：可以先通过INITRD启动系统之后进行EMMC或SD卡的格式化与分区，再选择SD/EMMC的启动方式即可。文件系统似乎并没有什么太大区别按NFS修改的文件系统照样可以运行在SD第二分区里面。
~~~

**配置与编译uboot**

~~~
petalinux-config -c u-boot
petalinux-build -c u-boot
~~~



**定制 Linux 内核**

~~~
petalinux-config -c kernel
一般均保持默认。
Petalinux 对内核版本有要求，读者如需使用其他的内核版本可以在网上查找关于 Petalinux 使用非默
认内核版本的方法。一般使用默认内核版本就可以了。 
这里使用的内核 Xilinx 官方已经做好了基础配置，如无特定需求，无需更改。
~~~

**配置 Linux 根文件系统** 

~~~
petalinux-config -c rootfs 
如果不需要配置可不执行该命令
~~~

**配置设备树文件** （system-top.dts）

~~~
在Zynq系统移植中，设备树文件之间的关系如下：
1. zynq-7000.dtsi
这是Zynq-7000系列处理器的通用设备树头文件，描述了PS（Processor System）端的硬件外设配置信息。它为所有基于Zynq-7000系列的开发板提供了基础的硬件描述，例如处理器、内存控制器、外设接口等。
2. system-top.dts
这是顶层设备树文件，通常由PetaLinux工具链生成。它通过#include指令引用其他设备树文件（如zynq-7000.dtsi、pl.dtsi、pcw.dtsi等），用于描述整个系统的硬件配置。system-top.dts是系统移植和硬件配置的核心文件，它定义了系统中所有硬件资源的配置和连接关系。
3. system-user.dtsi
这是一个用户自定义的设备树文件，用于添加或修改硬件描述信息。它通常包含用户特定的外设配置、引脚定义或其他自定义硬件信息。在系统移植过程中，system-user.dtsi会被system-top.dts引用，以实现对系统硬件的进一步定制。
4. zynq-zc702.dts
这是针对Zynq ZC702开发板的设备树文件，专门用于描述该开发板的硬件配置。它基于zynq-7000.dtsi，并针对ZC702开发板的特定硬件（如外设连接、引脚分配等）进行了扩展和定制。

system-top.dts 是顶层文件，它通过#include指令引用以下文件：
zynq-7000.dtsi：提供PS端的通用硬件描述。
pl.dtsi：描述PL（Programmable Logic）端的硬件配置。
pcw.dtsi：描述在Vivado中配置的PS外设。
system-user.dtsi：用户自定义的硬件描述。
zynq-zc702.dts 是针对特定开发板（如ZC702）的设备树文件，它也可以通过#include指令引用zynq-7000.dtsi，并进一步定制开发板的硬件配置。
在Zynq系统移植中：
如果使用PetaLinux工具链，通常以system-top.dts作为顶层文件，通过引用其他设备树文件（如zynq-7000.dtsi、pl.dtsi、system-user.dtsi等）来构建完整的硬件描述。
如果是针对特定开发板（如ZC702），可以直接使用zynq-zc702.dts，它基于zynq-7000.dtsi并进行了定制。
~~~

可以参考其余人写的设备树，并且查找对应的设备驱动，来进行驱动代码的编写。

~~~
vi project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi 
~~~

**编译 Petalinux 工程** 

~~~
petalinux-build
该命令将生成设备树 DTB 文件、fsbl 文件、U-Boot 文件、boot.scr 文件、Linux 内核和根文件系统文件。
编译完成后，生成的镜像文件将位于工程的 images 目录下。需要说明的是 fsbl、U-Boot 这两个我们在工程
中并没有配置，这是因为 Petalinux 会根据xsa/hdf文件和配置 petalinux 工程自动配置 fsbl 和 uboot，
如无特需要求，不需要再手动配置。 
~~~

**制作 BOOT.BIN 启动文件**

~~~
petalinux-package --boot --fsbl --fpga --u-boot --force 

--fsbl”用于指定 fsbl 文件所在位置，后面接文件对应的路径信息，如果不指定文件位置，默认
对应的是 images/linux/zynq_fsbl.elf
~~~



**制作 SD / EMMC启动卡**

可以直接在ubuntu环境下格式化好，也可以在INITRD启动之后，在启动的Linux环境下进行格式化？（待验证）

可以直接使用SD卡作为启动方式，也可以直接选择qspiFlash作为启动方式（两个均是作为BOOT.bin的存放区域）

Image Packaging Configuration子菜单根文件系统的 类型的配置使用的是默认的 INITRD，所以只需要一个使用 FAT32 文件系统的分区就可以了。当设置为SD/EMMC ,“EXT4”则需要设置为另一个存放根文件系统的分区。 



将 SD 卡插入到读卡器中、并将读卡器插入电脑并连接到 Ubuntu 系统，在 Ubuntu 系统中找到 SD 卡所 对应的设备节点， SD 卡对应的设备节点一般为/dev/sdb

~~~
umount /dev/sdb* 
sudo fdisk /dev/sdb 
~~~

~~~
1、输入 p 可以看到当前的分区表，有两个分区，一个 FAT32的分区和一个 exFAT分区。在开始新分区之前需要将以前的分区删除，键入“d”，然后输入 1，删除 1 分区，再次键入“d”删除第 2 个分区。
2、新建分区。输入“n”创建一个新分区。通过选择“p”使其为主，使用默认分区号 1 和第一个扇区 2048。设置最后一个扇区，也就是设置第一个分区的大小，一般设置 500M 足够了，通过输入“+500M”，为该分区预留 500MB，如果提示分区包含 vfat 签名并询问是否移除该签名，则输入“y”。
3、设置分区类型，输入“t”，然后输入“c”，设置为“W95 FAT32 (LBA)”，输入“a”，设为引导分区。
创建第二个分区，通过键入“n”来创建根文件系统分区。后面一路默认就可以了。
现在输入“p”检查分区表，会看到刚刚创建的 2 个分区。如果没问题，键入“w”以写入到 SD/EMMC并退出。
~~~

完成了分区创建后，开始格式化分区：

将第一个分区格式化成 FAT32 分区并命名为 boot，将第二个分区格式化成 ext4 分区并命名为 rootfs。

~~~
sudo mkfs.vfat -F 32 -n boot /dev/sdb1 
sudo mkfs.ext4 -L rootfs /dev/sdb2
~~~

~~~
要将rootfs.tar.gz文件放到第二个分区并将其作为根文件系统，你需要按照以下步骤操作：
挂载分区：
首先，你需要将第二个分区（ext4格式，设备文件为/dev/sdb2）挂载到一个临时目录。创建一个挂载点，并将其挂载：
sudo mkdir /mnt/rootfs
sudo mount /dev/sdb2 /mnt/rootfs
解压文件系统：
将rootfs.tar.gz文件解压到挂载点。确保你已经下载了rootfs.tar.gz文件，并且它位于当前目录或你知道其路径：
sudo tar -xzvf rootfs.tar.gz -C /mnt/rootfs
检查文件系统：
在将分区用作根文件系统之前，检查解压后的文件系统是否完整：
sudo fsck.ext4 /dev/sdb2
卸载分区：
完成文件系统解压后，卸载分区：
sudo umount /mnt/rootfs
重启系统：
完成上述步骤后，重启系统。如果一切配置正确，系统应该能够从新的根文件系统启动：
sudo reboot
~~~



格式化分区之后挂载分区（重新插拔读卡器或者使用 mount 命令进行挂载）。 我们将该工程 image/linux 目录下的 BOOT.BIN、 和 image.ub 文件拷贝到名为 boot 的分区/dev/sdb1，然后使用 umount 命令卸载 SD 卡。
EMMC则在uboot阶段通过写MMC的方式写入？（Uboot配置阶段再细说）



## 在Uboot阶段所做的工作

进入 uboot 的命令行模式以后输入“help”或者“？”，然后按下回车即可查看当前 uboot 所支持的命令

~~~
help tftpboot 
?    tftpboot 
~~~

信息查询命令

~~~
bdinfo			DRAM 的起始地址和大小、启动参数保存起始地址、波特率、sp(堆栈指针)起始地址等信息。
printenv
version
~~~



**bootargs和bootcmd**

~~~
网络启动：
bootargs=console=ttyPS0,115200 root=/dev/nfs rw nfsroot=192.168.1.9:/home/NFSfile,nfsvers=3 ip=192.168.1.7:192.168.1.9:192.168.1.1:255.255.255.0::eth0:off

SD/EMMC启动：
setenv bootargs “console=ttyPS0,115200 earlyprintk root=/dev/mmcblk0p2 rw rootwait”
saveenv
setenv bootcmd   "run newsdboot"
setenv newsdboot "fatload mmc 0:1 0x10000000 image.ub

~~~



**内存操作命令**

md 命令用于显示内存值，使用help查看作用

~~~
help md
~~~

nm 命令用于修改指定地址的内存值，mm 命令也是修改指定地址内存值的，使用 mm 修改内存值的时候地址会自增使用help查看作用

~~~
help nm				q退出修改的值
help mm
~~~

mw 命令 用于使用一个指定的数据填充一段内存

~~~
mw.l 8000000 0A0A0A0A 10
~~~

cp 是数据拷贝命令，用于将 DRAM 中的数据从一段内存拷贝到另一段内存中

~~~
cp.l 8000000 8000100 10
~~~

cmp 是比较命令，用于比较两段内存的数据是否相等

~~~
cmp.l 8000000 8000100 10 
~~~

网络命令

~~~
连接网线，开发板上电后 uboot 默认通过 dhcp 获取网络 ip 地址（与路由器连接时有效），若与电脑直
连，可以通过以下命令手动设置： 
setenv ipaddr 192.168.1.7
setenv ethaddr 00:0a:35:00:1e:53 
setenv gatewayip 192.168.1.1 
setenv netmask 255.255.255.0 
setenv serverip 192.168.1.8 
saveenv 
~~~

dhcp用于从路由器获取 IP地址，前提是开发板连接到路由器，如果开发板是和电脑直连的，那么 dhcp命令就会失效。直接输入 dhcp 命令即可通过路由器获取到 IP 地址

nfs命令：

~~~
nfs 00000000 192.168.1.20:/home/wmq/workspace/nfs/zImage
~~~

命令中的“00000000”表示 zImage 保存地址，“192.168.1.20:/home/wmq/workspace/nfs/zImage”表示zImage 在 192.168.1.20 这个主机中，路径为/home/wmq/workspace/nfs/zImage。

od 命令或 xxd 命令来查看 Ubuntu 下的 zImage 文件，检查一下下载到开发板 DDR 中的数据是否 与 zImage 原文件一致

~~~
od -tx1 -vN 0x100 zImage
~~~

tftpboot 命令：

~~~
tftpboot 00000000 zImage
~~~

EMMC 和 SD 卡操作命令：

~~~
help mmc   查看所有mmc的命令
输出当前选中的 mmc info 设备的信息：
mmc info
扫描当前开发板上所有的 MMC设备，包括 EMMC和 SD卡：
mmc rescan
查看当前开发板一共有几个 MMC 设备：
mmc list
切换当前 MMC 设备:
mmc dev 1			切换到 eMMC，0 为 SD 卡，1 为 eMMC
SD 卡或者 EMMC 会有多个分区,查看分区：
mmc dev 0 					//切换到 SD 卡 
mmc part				   //查看 SD 卡分区
读取 mmc 设备的数据：
mmc dev 0 0 				  //切换到 SD 卡的 0 分区 
mmc read 00000000 800 10 //读取数据 
将数据写到 MMC 设备里面：
mmc write addr blk# cnt 
addr 是要写入 MMC 中的数据在 DRAM 中的起始地址，blk 是要写入 MMC 的块起始地址(十六进制)，cnt 是要写入的块大小，一个块为 512 字节。注意千万不要写 SD 卡或者 EMMC 的前两个块(扇区)，里面保存着分区表信息。 
设置 MMC 设备的分区：
mmc hwpartition  
~~~

FAT 格式文件系统操作命令

~~~
查询指定 MMC 指定分区的文件系统信息：
fatinfo mmc 0:1 
查询 FAT 格式设备的目录和文件信息：
fatls mmc 0:1 
查看MMC 设备某个分区的文件系统格式：
fstype mmc 0:1 
fstype mmc 0:2 
将指定的文件读取到 DRAM 中：
fatload mmc 0:1 00000000 BOOT.BIN
将 DRAM 中的数据写入到 MMC 设备中
tftpboot 00000000 image.ub
fatwrite mmc 0:1 00000000 image.ub 0XB4A564
~~~

EXT 格式文件系统操作命令

~~~
ext4ls mmc 0:2
其余的可以参考fat的形式
ext4load
ext4size
ext4write 

fatload mmc 0:1 0x00000000 image.ub   这里显示的长度为10进制
ext4write mmc 0:2 0x00000000 image.ub 刚写入的长度9876EC
~~~

系统引导命令

~~~
bootm 命令用于启动在内存中的用 mkimage 工具处理过的内核镜像。由于 zynq 使用 image.ub 镜像文件，而 image.ub 镜像文件属于 U-Boot fitImage，里面通常包括 linux 内核和设备树，所以可以将 image.ub 镜像文件写到 DRAM 中，然后使用 bootm 命令来动。
tftpboot 10000000 image.ub 
bootm 10000000

bootz 和 bootm 功能类似，只是 bootz 命令用于启动 zImage 镜像文件，zynq 使用的不多，了解即可。
tftpboot 00000000 zImage 
tftpboot 05000000 system.dtb 
bootz 00000000 - 05000000

bootvx、bootelf等等(vxworks直接通过elf文件加载能否可行?)
~~~

mtest 命令是一个简单的内存读写测试命令，可以用来测试自己开发板上的 DDR

~~~
mtest 00000000 00001000
~~~

run 命令用于运行环境变量中定义的命令

~~~
run bootcmd
~~~

reset即可复位重启

mdio网络phy的调试

~~~
help mdio      以及上网找对应的资料
~~~



## Linux系统启动之后

**配ip地址**

~~~
方法一：
ifconfig eth0 192.168.1.107 netmask 255.255.255.0 up
方法二：
ip addr add 192.168.1.107/24 dev eth0
ip link set eth0 up
~~~



**开机自启动代码**

~~~
开机自启动。开机启动后进入根文件系统的时候会运行/etc/init.d/rcS 这个 shell脚本，Linux 内
核启动以后需要启动一些服务，而 rcS 就是规定启动哪些文件的脚本文件。因此我们可以在这个脚本里面
添加自启动相关内容。添加完成以后的/etc/init.d/rcS
#!/bin/sh 
# 
# rcS Call all S??* scripts in /etc/rcS.d in 
# numerical/alphabetical order. 
# 
# Version: @(#)/etc/init.d/rcS 2.76 19-Apr-1999 miquels@cistron.nl 
# 
PATH=/sbin:/bin:/usr/sbin:/usr/bin 
runlevel=S 
prevlevel=N 
umask 022 
export PATH runlevel prevlevel 

# Make sure proc is mounted 

# 

[ -d "/proc/1" ] || mount /proc 

# 
# Source defaults. 
# 
. /etc/default/rcS 
#开机自启动 (在此处添加开机自启动的代码)

cd /drivers 

./hello & 

cd 
~~~

**启动脚本可能存在的问题(服务启动优先级问题和内存文件系统的问题)**

~~~
即使您已经将 ifconfig eth0 192.168.1.107 netmask 255.255.255.0 up 命令写入 /etc/init.d/rcS 并且确认重启后内容还在，但是IP没有被配置成功，可能的原因和解决方案如下：
网络服务未启动：确保在执行网络配置命令之前，网络服务已经启动。在PetaLinux中，网络服务可能需要在特定的运行级别启动。您可以尝试将网络服务的启动脚本放置在 /etc/rc5.d/（对于运行级别5）或其他适当的运行级别目录下，以确保在系统启动时网络服务已经启动。

***在Linux系统中，/etc/rc5.d/（或其他运行级别目录，如/etc/rc3.d/）包含了系统启动时执行的脚本。这些脚本通常以S或K开头，S代表启动（start），K代表停止（kill）。数字代表执行顺序，数字越小越早执行。

如果您想要在/etc/rc5.d/目录下创建一个新的启动脚本，可以按照以下步骤操作：
创建一个名为start.sh的脚本文件，并写入您想要在启动时执行的命令。
#!/bin/bash
ifconfig eth0 192.168.1.107 netmask 255.255.255.0 up
chmod +x /etc/rc5.d/start.sh
将脚本添加到启动序列
创建符号链接：
为了在启动时自动执行您的脚本，您需要在/etc/rc5.d/目录下创建一个指向您的脚本的符号链接。通常，您会创建一个以S开头的链接，后面跟着一个数字和您的脚本名称。数字越小，脚本越早执行。
ln -s /etc/rc5.d/start.sh /etc/rc5.d/S05start.sh
这里S05start.sh表示在运行级别5启动时，start.sh脚本将作为第五个执行的启动脚本。
列出/etc/rc5.d/目录下的文件，确认您的脚本链接是否正确创建：
ls -l /etc/rc5.d/
~~~

init系统起来之后进行EMMC文件系统的格式化分区，选qspiflash启动，通过IDE下载BOOT.bin到qspiflash启动即可

~~~
init或者nfs启动系统后进入dev分区查看emmc的设备名称(如mmcblk0)然后执行以下命令进行EMCC分区
fdisk /dev/mmcblk0
1、输入 p 可以看到当前的分区表，有两个分区，一个 FAT32的分区和一个 exFAT分区。在开始新分区之前需要将以前的分区删除，键入“d”，然后输入 1，删除 1 分区，再次键入“d”删除第 2 个分区。
2、新建分区。输入“n”创建一个新分区。通过选择“p”使其为主，使用默认分区号 1，First cylinder (1-29512, default 1): Using default value 1，默认即可，设置第一个分区的大小，一般设置 500M 足够了，通过输入“+500M”，为该分区预留 500MB，如果提示分区包含 vfat 签名并询问是否移除该签名，则输入“y”。
3、设置分区类型，输入“t”，然后输入“c”，设置为“W95 FAT32 (LBA)”，输入“a”，设为引导分区。
创建第二个分区，通过键入“n”来创建根文件系统分区。后面一路默认就可以了。
现在输入“p”检查分区表，会看到刚刚创建的 2 个分区。如果没问题，键入“w”以写入到 SD/EMMC并退出。

格式化分区
mkfs.vfat -F 32 -n boot /dev/mmcblk0p1
mkfs.ext4 -L rootfs /dev/mmcblk0p2				//没有这个命令?
petalinux-config -c rootfs
Filesystem Packages
    base
        e2fsprogs
勾选
e2fsprogs
e2fsprogs-mke2fs进行添加这个命令

输入df -T
查看到

Filesystem           Type       1K-blocks      Used Available Use% Mounted on
devtmpfs             devtmpfs      243580         4    243576   0% /dev
tmpfs                tmpfs         255232        76    255156   0% /run
tmpfs                tmpfs         255232        44    255188   0% /var/volatile
/dev/mmcblk0p1       vfat         4873316     10328   4862988   0% /run/media/mmcblk0p1
/dev/mmcblk0p2       ext4        24808148     44992  23479900   0% /run/media/mmcblk0p2



格式化分区之后会自动将分区进行挂载，如果没有则手动挂载
mount /dev/mmcblk0p1 /run/media/mmcblk0p1
如果没有路径则可能需要mkdir /run/media/mmcblk0p1


顺利挂载之后可以通过 cd /run/media/mmcblk0p1
ls
查看到image.ub和BOOT.bin
然后cd /run/media/mmcblk0p2
通过Filezilla软件连接之后，rootfs.tar.gz传过来
然后通过指令进行解压缩
cd /run/media/mmcblk0p2
tar -xzvf rootfs.tar.gz
或者在拥有rootfs.tar.gz的目录下
tar -xzvf rootfs.tar.gz -C /run/media/mmcblk0p2
~~~

遇到的疑似两个根文件系统问题问KIMI给出的解答

~~~
使用petalinux生成的image.ub和BOOT.bin放到sd卡中启动之后，进行格式化分区，boot和rootfs重启之后将image.ub和BOOT.bin放到sd卡的分区1,将rootfs.tar.gz解压到分区2rootfs后，挂载点为/run/media/mmcblk0p1和/run/media/mmcblk0p2，现在的问题是，那这样不就总共有两个linux操作系统了吗?因为我总共看到了两个home目录分别是/home/root和/run/media/mmcblk0p2/home/root，但是已经将initd启动改为sd卡启动了/home/root不能保存文件，/run/media/mmcblk0p2/home/root可以保存文件。为什么有两个linux系统????我使用cd ~ 是去的/home/root，但是/home/root又不能保存文件，这是有bug吗？还是我 的理解错误了？？
~~~

解决方案一：

~~~
问题的核心在于启动过程中出现了两个根文件系统（rootfs）的挂载，导致出现了两个不同的 `/home/root` 目录。这种情况并不是因为存在两个 Linux 操作系统，而是由于启动过程中挂载了两个不同的根文件系统分区。

### 问题分析
1. **默认的 RAMFS 启动**：PetaLinux 默认使用 RAMFS 作为初始根文件系统，这种文件系统在启动时加载到内存中，因此 `/home/root` 目录无法保存文件。
2. **SD 卡上的 EXT4 分区**：你将 `rootfs.tar.gz` 解压到 SD 卡的第二个分区（EXT4 文件系统），并挂载到 `/run/media/mmcblk0p2`，这个分区是可以保存文件的。
3. **挂载点问题**：系统启动后，初始的根文件系统（RAMFS）仍然存在，而你手动挂载的 SD 卡分区（EXT4 文件系统）被挂载到了 `/run/media/mmcblk0p2`，导致出现了两个不同的根文件系统。

### 解决方案
要解决这个问题，需要确保系统启动时直接使用 SD 卡上的 EXT4 分区作为根文件系统，而不是默认的 RAMFS。以下是具体步骤：

#### 1. 修改启动参数
确保启动时直接挂载 SD 卡的 EXT4 分区作为根文件系统。可以通过以下方法修改启动参数：
- **在 U-Boot 中设置环境变量**：
  setenv bootargs “console=ttyPS0,115200 earlyprintk root=/dev/mmcblk0p2 rw rootfstype=ext4 rootwait ”
  这里的 `/dev/mmcblk0p2` 是 SD 卡的第二个分区，具体设备号需要根据实际情况调整。

#### 2. 确保根文件系统类型正确
在 PetaLinux 配置中，确保根文件系统类型设置为 EXT4，并指定正确的设备节点：
- 运行 `petalinux-config`：
  petalinux-config
- 在配置菜单中，选择：
  Image Packaging Configuration --->
      Root filesystem type (SD)
      Device node of SD device (/dev/mmcblk0p2)
#### 3. 重新打包启动文件
重新打包启动文件，确保启动参数生效：
petalinux-package --boot --fsbl --fpga --u-boot --force
将生成的 `BOOT.BIN` 和 `image.ub` 文件复制到 SD 卡的第一个分区。
#### 4. 验证启动
插入 SD 卡并启动开发板，确保系统直接使用 SD 卡的 EXT4 分区作为根文件系统。此时，`/home/root` 目录应该位于 SD 卡的 EXT4 分区中，并且可以保存文件。

### 注意事项
- 如果仍然出现两个根文件系统，可能是启动过程中存在多个挂载点。可以通过 `mount` 命令查看当前挂载的文件系统，并手动卸载不必要的挂载点。
- 确保 `/etc/fstab` 文件中正确配置了根文件系统的挂载点，避免系统启动时自动挂载错误的分区。

通过上述步骤，可以确保系统启动时直接使用 SD 卡上的 EXT4 分区作为根文件系统，避免出现两个根文件系统的问题。


按上述要求修改之后文件系统/dev/root，类型为ext4挂载点为/

root@ALIENTEK-ZYNQ:~# df -T
Filesystem           Type       1K-blocks      Used Available Use% Mounted on
/dev/root            ext4        24808148     61852  23463040   0% /
devtmpfs             devtmpfs      246528         4    246524   0% /dev
tmpfs                tmpfs         255232        76    255156   0% /run
tmpfs                tmpfs         255232        40    255192   0% /var/volatile
/dev/mmcblk0p1       vfat         4873316     10852   4862464   0% /run/media/mmcblk0p1

能不能在系统下修改挂载点呢？
留意这个文件/etc/fstab

cat /etc/fstab
# stock fstab - you probably want to override this with a machine specific one

/dev/root            /                    auto       defaults              1  1
proc                 /proc                proc       defaults              0  0
devpts               /dev/pts             devpts     mode=0620,gid=5       0  0
tmpfs                /run                 tmpfs      mode=0755,nodev,nosuid,strictatime 0  0
tmpfs                /var/volatile        tmpfs      defaults              0  0
# uncomment this if your device has a SD/MMC/Transflash slot
#/dev/mmcblk0p1       /media/card          auto       defaults,sync,noauto  0  0

root@ALIENTEK-ZYNQ:~# mount
/dev/root on / type ext4 (rw,relatime,data=ordered)
proc on /proc type proc (rw,relatime)
sysfs on /sys type sysfs (rw,relatime)
configfs on /sys/kernel/config type configfs (rw,relatime)
devtmpfs on /dev type devtmpfs (rw,relatime,size=246528k,nr_inodes=61632,mode=755)
tmpfs on /run type tmpfs (rw,nosuid,nodev,mode=755)
tmpfs on /var/volatile type tmpfs (rw,relatime)
/dev/mmcblk0p1 on /run/media/mmcblk0p1 type vfat (rw,relatime,gid=6,fmask=0007,dmask=0007,allow_utime=0020,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
devpts on /dev/pts type devpts (rw,relatime,gid=5,mode=620,ptmxmode=000)

使用EMMC&ext4cmd文件夹里面的boot,后续其余系统的花样逐步完善
~~~

解决方案二：

~~~
接上文::
### 注意事项
- 如果仍然出现两个根文件系统，可能是启动过程中存在多个挂载点。可以通过 `mount` 命令查看当前挂载的文件系统，并手动卸载不必要的挂载点。
- 确保 `/etc/fstab` 文件中正确配置了根文件系统的挂载点，避免系统启动时自动挂载错误的分区。


1. 检查当前挂载的文件系统
如果系统启动后出现了多个根文件系统（例如，一个 RAMFS 和一个 EXT4 文件系统），可以通过 mount 命令查看当前挂载的文件系统。这有助于确定哪些挂载点是多余的。
操作步骤：
打开终端。
输入以下命令：
mount
或者更简洁的：
mount | grep -E 'rootfs|/dev/mmcblk0p2'
分析输出：
如果看到类似以下输出：
复制
/dev/mmcblk0p2 on /run/media/mmcblk0p2 type ext4 (rw,relatime)
rootfs on / type tmpfs (rw)
这表明：
/dev/mmcblk0p2 被挂载到了 /run/media/mmcblk0p2，这是正确的挂载点。
rootfs 是一个临时文件系统（如 RAMFS），它被挂载到了 /，这可能是初始启动时的默认行为。
解决方法：
如果 / 挂载的是临时文件系统（如 RAMFS），而 /run/media/mmcblk0p2 是实际的根文件系统，需要将 /dev/mmcblk0p2 重新挂载到 /，并卸载不必要的挂载点。
# 卸载不必要的挂载点
umount /run/media/mmcblk0p2
#卸载完之后可以使用
df -T查看不要在同一目录下卸载，不然会busy，不小心卸载了重新挂载即可
# 重新挂载根文件系统
mount /dev/mmcblk0p2 /
 mounted filesystem with ordered data mode. Opts: (null)说明ok
 使用df -T也ok
2. 配置 /etc/fstab 文件
/etc/fstab 文件用于定义系统启动时自动挂载的文件系统。如果配置不正确，可能会导致系统启动时挂载错误的分区，或者重复挂载分区。
操作步骤：
打开 /etc/fstab 文件：

vi /etc/fstab
确保 /etc/fstab 文件中正确配置了根文件系统的挂载点。例如：

# /etc/fstab: static file system information.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mmcblk0p2  /               ext4    defaults        0       1
关键点：
/dev/mmcblk0p2 是根文件系统所在的分区。
挂载点为 /，表示这是系统的根目录。
文件系统类型为 ext4。
defaults 是默认挂载选项，包括 rw（读写模式）、relatime 等。
0 和 1 分别表示是否需要 dump 和 fsck 检查。
检查其他挂载点：
确保 /etc/fstab 文件中没有重复或冲突的挂载点。例如，避免出现类似以下内容：
/dev/mmcblk0p2  /run/media/mmcblk0p2  ext4    defaults        0       2
如果有类似的重复挂载点，删除或注释掉不必要的行。
3. 总结
检查挂载点：通过 mount 命令查看当前挂载的文件系统，手动卸载不必要的挂载点。
配置 /etc/fstab：确保 /etc/fstab 文件中正确配置了根文件系统的挂载点，避免重复或错误的挂载。
重新启动：完成上述操作后，重新启动系统，确保系统按照预期启动并使用正确的根文件系统。



因为是ramfs(内存文件系统，所以此方法无效)
~~~

bugs:

~~~
出现了一次tar解压文件系统不全导致文件系统起不来的问题，要注意
~~~



