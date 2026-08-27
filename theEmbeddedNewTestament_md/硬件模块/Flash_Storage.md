---
tags:
  - 硬件模块
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/HW_Module/Flash_Storage.md
created: 2026-08-27
---

# 闪存存储与文件系统(Flash Storage and File System)

## 闪存存储(Flash Storage)

闪存存储，也叫固态存储(solid state)，相比旋转式存储(rotating storage)有多个优势。首先，没有机械和运动部件消除了噪音，提高了可靠性和对冲击与振动的抵抗力，同时也减少了散热和功耗。其次，对数据的随机访问也快得多，因为你不再需要将磁盘磁头移动到介质上的正确位置，这可能需要毫秒级的时间。

当然，闪存也有它的缺点。首先，同样的价格下，你得到的固态存储大约是旋转式存储的十分之一。对于需要数 GB 磁盘空间的操作系统来说，这可能是个问题。幸运的是，Linux 只需要几 MB 的存储。其次，写入闪存存储有特殊的约束。你不能在不擦除整个块的情况下多次写入闪存块上的同一位置，这个块被称为"擦除块"(erase block)。这个约束也可能导致写入速度远低于读取速度。第三，闪存块只能承受相当有限次数的擦除（从如今最密集的 NAND 闪存的几千次，到最佳的百万次）。这要求实现硬件或软件解决方案，称为"磨损均衡"(wear leveling)，以确保没有闪存块被写入得比其他块频繁太多。

## NOR 闪存(Nor Flash)

NOR 闪存是第一种被发明出来的闪存存储类型。NOR 非常方便，因为它允许 CPU 以随机顺序逐个字节地访问每个字节。这样，CPU 可以直接从 NOR 闪存执行代码。这对于 bootloader 非常方便，因为它们在执行代码之前不必被复制到 RAM。

NOR 闪存架构提供了足够的地址线来映射整个内存范围。这带来了随机访问和短读取时间的优势，使其成为代码执行的理想选择。另一个优势是零件寿命内 100% 已知的良好位。缺点包括更大的单元尺寸导致更高的每比特成本，以及更慢的写入和擦除速度。

![525px-NOR_flash_layout.svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/NOR_flash_layout.svg/700px-NOR_flash_layout.svg.png)


在 NOR 门闪存中，每个单元的一端直接连接到地，另一端直接连接到一条位线。这种排列被称为"NOR 闪存"，因为它像一个 NOR 门：当一条字线（连接到单元的 Control Gate）被拉高时，对应的存储晶体管会将输出位线拉低。

## NAND 闪存(Nand Flash)

NAND 闪存是如今最流行的闪存存储类型，因为它以低得多的成本提供更多的存储容量。缺点在于 NAND 存储位于外部设备上，就像旋转式存储一样。你必须使用控制器来访问设备数据，CPU 不能在将代码复制到 RAM 之前从 NAND 执行代码。另一个约束是 NAND 闪存设备出厂时可能带有损坏的块，需要硬件或软件解决方案来识别和丢弃坏块。

相比之下，NAND 闪存的单元尺寸小得多，写入和擦除速度也比 NOR 闪存高得多。缺点包括较慢的读取速度，以及 I/O 映射或间接接口，这更复杂且不允许随机访问。重要的是要注意，从 NAND 闪存执行代码是通过将内容影子复制(shadowing)到 RAM 来实现的，这与直接从 NOR 闪存执行代码不同。另一个主要缺点是坏块的存在。NAND 闪存出厂时通常有 98% 的良好位，零件寿命内还有额外的位故障，因此设备内部需要错误校正码（ECC，Error Correcting Code）功能。

![525px-Nand_flash_structure.svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f5/Nand_flash_structure.svg/525px-Nand_flash_structure.svg.png)

NAND 闪存也使用浮栅晶体管(floating-gate transistors)，但它们的连接方式类似一个 NAND 门：几个晶体管串联，只有当所有字线都被拉高（高于晶体管的 VT）时，位线才被拉低。


![contenteetimes-images-design-embedded-2018-fl-1-t1.jpg](https://www.embedded.com/wp-content/uploads/contenteetimes-images-design-embedded-2018-fl-1-t1.jpg)

### 1. 闪存转换层(Flash Transition Layer)

第一种 NAND 闪存模拟标准块接口，并包含一个硬件"闪存转换层"(Flash Translation Layer)，负责擦除块、实现磨损均衡和管理坏块。这对应 USB 闪存驱动器、介质卡、嵌入式 MMC（eMMC）和固态磁盘（SSD）。操作系统无法控制闪存扇区的管理方式，因为它只看到一个模拟的块设备。这有助于降低操作系统侧的软件复杂度。然而，硬件厂商通常将他们的闪存转换层算法保密。这使系统开发者无法验证和调整这些算法，我听到自由软件社区中有多个声音怀疑这些商业机密是掩盖糟糕实现的一种方式。例如，有人告诉我一些闪存介质是在 16 MB 扇区上实现磨损均衡，而不是使用整个存储空间。这可能让闪存设备很容易损坏。

### 2. 内存技术设备(Memory Technology Device)

第二种 NAND 闪存是裸闪存(raw flash)。操作系统可以访问闪存控制器，并直接管理闪存块。统计一个块被擦除的次数也是可能的（"块擦除计数"，block erase count）。Linux 内核实现了一个内存技术设备（MTD，Memory Technology Device）子系统，允许用通用接口访问和控制各种类型的闪存设备。这给了我们实现与硬件无关的软件来管理闪存存储的自由，特别是文件系统。自由与独立是我们社区中学会珍视的东西。

![mtd-architecture.png](https://bootlin.com/wp-content/uploads/2012/07/mtd-architecture.png)

## Linux MTD 分区

MTD 设备通常被分区。这对于定义不同用途的区域很有用。裸(raw)意味着不使用文件系统。当你只需存储一个二进制文件而不是多个文件时，这并不是必需的。

![flash-partitions.png](https://bootlin.com/wp-content/uploads/2012/12/flash-partitions.png)


MTD 分区的特殊之处在于没有像块设备那样的分区表。这可能是因为闪存是存储此类关键系统信息的不安全位置，因为闪存块在系统寿命内可能变坏。

相反，分区是在内核中定义的。一个例子在内核源码中的 arch/arm/mach-omap2/board-omap3beagle.c 文件中，为 Beagle 板定义闪存分区：

```C
static struct mtd_partition omap3beagle_nand_partitions[] = {
        /* All the partition sizes are listed in terms of NAND block size */
        {
                .name           = "X-Loader",
                .offset         = 0,
                .size           = 4 * NAND_BLOCK_SIZE,
                .mask_flags     = MTD_WRITEABLE,        /* force read-only */
        },
        {
                .name           = "U-Boot",
                .offset         = MTDPART_OFS_APPEND,   /* Offset = 0x80000 */
                .size           = 15 * NAND_BLOCK_SIZE,
                .mask_flags     = MTD_WRITEABLE,        /* force read-only */
        },
        {
                .name           = "U-Boot Env",
                .offset         = MTDPART_OFS_APPEND,   /* Offset = 0x260000 */
                .size           = 1 * NAND_BLOCK_SIZE,
        },
        {
                .name           = "Kernel",
                .offset         = MTDPART_OFS_APPEND,   /* Offset = 0x280000 */
                .size           = 32 * NAND_BLOCK_SIZE,
        },
        {
                .name           = "File System",
                .offset         = MTDPART_OFS_APPEND,   /* Offset = 0x680000 */
                .size           = MTDPART_SIZ_FULL,
        },
};
```

你可以覆盖这些默认定义而不必修改内核源码。你首先需要找到要分区的 MTD 设备的名称，因为你可能有多个。查看启动时的内核日志。在 Beagle 板示例中，MTD 设备名称是 omap2-nand.0：

```C
omap2-nand driver initializing
ONFI flash detected
NAND device: Manufacturer ID: 0x2c, Chip ID: 0xba (Micron NAND 256MiB 1,8V 16-bit)
Creating 5 MTD partitions on "omap2-nand.0":
0x000000000000-0x000000080000 : "X-Loader"
0x000000080000-0x000000260000 : "U-Boot"
0x000000260000-0x000000280000 : "U-Boot Env"
0x000000280000-0x000000680000 : "Kernel"
0x000000680000-0x000010000000 : "File System"
```
<br>
<br>
Linux 内核提供了一个 mtdpartss 启动参数来定义你自己的分区边界。
我们刚刚在 omap2-nand.0 设备中定义了 6 个分区。

```C
mtdparts=omap2-nand.0:128k(X-Loader)ro,256k(U-Boot)ro,128k(Environment),4m(Kernel)ro,32m(RootFS)ro,-(Data)
```

**注意分区大小必须是擦除块大小的倍数。擦除块大小可以在目标系统的 /sys/class/mtd/mtdx/erasesize 中找到。**

* 第一阶段 bootloader（128 KiB，只读）
* U-Boot（256 KiB，只读）
* U-Boot 环境（128 KiB）
* 内核（4 MiB，只读）
* 根文件系统（16 MiB，只读）
* 数据（剩余空间）


现在分区已定义，你可以通过查看 /proc/mtd 显示对应的 MTD 设备（大小是十六进制）：

```C
dev:    size   erasesize  name
mtd0: 00020000 00020000 "X-Loader"
mtd1: 00040000 00020000 "U-Boot"
mtd2: 00020000 00020000 "Environment"
mtd3: 00400000 00020000 "Kernel"
mtd4: 02000000 00020000 "File System"
mtd5: 0dbc0000 00020000 "Data"
```
## 操作 MTD 设备

你可以通过两种类型的接口访问 MTD 设备编号 X。

### MTD 字符设备

第一个接口是 /dev/mtdX 字符设备，由 mtdchar 驱动管理。特别是，这个字符设备提供 ioctl 命令，通常被 mtd-utils 命令用来操作和擦除 MTD 设备中的块。

### MTD 块设备

第二个接口是 /dev/mtdblockX 块设备，由 mtdblock 驱动处理。这个设备主要用于挂载 MTD 文件系统，如 JFFS2 和 YAFFS2，因为 mount 命令主要与块设备配合工作。


## Linux MTD 命令

这些命令通过 GNU/Linux 发行版中的 mtd-utils 包提供，也可以由嵌入式 Linux 构建系统（如 Buildroot 和 OpenEmbedded）从源码交叉编译。最常用命令的简单实现也可在 BusyBox 中获得，使它们更容易为简单嵌入式系统交叉编译。

操作 MTD 设备的干净方式是通过字符接口，并使用 mtd-utils 命令。以下是最常用的一些：

* mtdinfo：获取关于 MTD 设备的详细信息
* flash_eraseall：完全擦除给定的 MTD 设备
* flashcp：写入 NOR 闪存
* nandwrite：写入 NAND 闪存
* mkfs.jffs2, mkfs.ubifs：闪存文件系统镜像创建工具：
* UBI 实用程序


## JFFS2

日志闪存文件系统版本 2（JFFS2，Journaling Flash File System version 2）于 2001 年加入 Linux 内核，是一种非常流行的闪存存储文件系统。正如闪存文件系统所预期的那样，它实现了坏块检测和管理，以及磨损均衡。它还设计为在突发断电和系统崩溃后保持一致状态。最后但同样重要的是，它还将数据以压缩形式存储。根据哪个更重要——读/写性能还是压缩率——有多种压缩方案可用。例如，zlib 比 lzo 压缩得更好，但也慢得多。

磨损均衡（也写作 wear levelling）是一种延长某些可擦除计算机存储介质（如闪存，用于固态驱动器（SSD）和 USB 闪存驱动器，以及相变存储器）使用寿命的技术。

[Wear leveling - Wikipedia](https://en.wikipedia.org/wiki/Wear_leveling)

### JFFS2 日志结构方法

实现闪存文件系统有特殊约束。当你对特定文件做更改时，你不应该只是走捷径，将对应的块复制到 RAM、擦除它们、然后用新版本刷写这些块。第一个原因是擦除或写入操作期间断电会导致不可恢复的数据丢失。第二个原因是你会因为对同一文件进行多次更新而很快磨损特定块。解决方案是将新数据复制到一个新块，并将对旧块的引用替换为对新块的引用。然而，这意味着在文件系统上进行另一次写入，导致更多引用被修改，直到达到根引用。

JFFS2 使用日志结构(log-structured)方法来解决这个问题。每个文件通过一个"node"来描述，描述文件元数据和数据，每个节点有一个关联的版本号。与其进行就地修改，不如在有空闲空间的擦除块的其他位置写入节点的新版本。虽然这简化了写入操作，但使读取操作复杂化，因为读取文件需要为此文件找到最新版本的节点。

回到节点管理，旧节点必须在某个时候被回收，以保持空间用于更新的写入。一个节点创建时是"有效"(valid)的，当创建新版本时被视为"过期"(obsolete)。JFFS2 管理三种类型的闪存块：

* 干净块(Clean blocks)：只包含有效节点
* 脏块(Dirty blocks)：至少包含一个过期节点
* 空闲块(Free blocks)：尚不包含任何节点

JFFS2 在后台运行垃圾收集器(garbage collector)，将脏块回收为空闲块。它通过收集脏块中的所有有效节点，并将它们复制到（有剩余空间的）干净块或空闲块来实现。旧脏块然后被擦除并标记为空闲。为了让所有擦除块参与磨损均衡，垃圾收集器偶尔也消耗干净块。


![JFFS2.png](https://sourceware.org/jffs2/jffs2-html/img3.png)

https://sourceware.org/jffs2/jffs2-html/node3.html


### JFFS2 CONFIG_JFFS2_SUMMARY

为了优化性能，JFFS2 为每个文件保留最近节点的内存映射。然而，这需要在挂载时扫描所有节点以重建这个映射。这非常昂贵，因为 JFFS2 的挂载时间与节点数量成正比。在大的闪存分区上使用 JFFS2 的嵌入式系统因为这一点遭受很大的启动时间惩罚。幸运的是，添加了 CONFIG_JFFS2_SUMMARY 内核选项，允许将这个映射存储在地闪存设备本身上，显著减少挂载时间。小心，这个选项默认是关闭的。


### JFFS2 命令

有两种在闪存分区上使用 JFFS2 的方式。

第一种方式是擦除分区并格式化为 JFFS2，然后挂载它。注意 flash_eraseall -j 既擦除闪存分区也将其格式化为 JFFS2。然后你可以通过写入数据来填充分区。

```sh
flash_eraseall -j /dev/mtd2
mount -t jffs2 /dev/mtdblock2 /mnt/flash
```

第二种方式，对于编程生产设备更方便，是在开发工作站上准备一个 JFFS2 镜像，并将这个镜像刷写到分区中：

```sh
flash_eraseall /dev/mtd2
nandwrite -p /dev/mtd2 rootfs.jffs2
```

要准备 JFFS2 镜像，你需要使用 mtd-utils 提供的 mkfs.jffs2 命令。不要被它的名字弄糊涂：与某些其他 mkfs 命令不同，它不是创建一个文件系统，而是创建一个文件系统镜像。你首先需要找到擦除块大小（如前所述）。让我们假设它是 256 MiB。然后在你自己的工作站上创建镜像：

```sh
mkfs.jffs2 --pad --no-cleanmarkers --eraseblock=256 -d rootfs/ -o rootfs.jffs2
```
-d 指定包含文件系统内容的目录
--pad 允许创建一个大小是擦除块大小倍数的镜像。
--no-cleanmarkers 只应该用于 NAND 闪存。

## YAFFS2

YAFFS2 是"又一种闪存文件系统"(Yet Another Flash Filesystem)，显然作为 JFFS2 的替代品而创建。它不使用压缩，但具有快得多的挂载时间，以及比 JFFS2 更好的读和写性能。YAFFS2 不如 JFFS2 流行，这可能是因为它不是主线上 Linux 内核的一部分。相反，它以带有用于修补大多数 Linux 内核源码版本的脚本的独立代码形式提供。

在修补内核后使用 YAFFS2，你只需擦除分区：

```sh
flash_eraseall /dev/mtd2
```

文件系统在第一次挂载时自动格式化：
```sh
mount -t yaffs2 /dev/mtdblock2 /mnt/flash
```
也可以用 yaffs-utils 中的 mkyaffs 工具创建 YAFFS2 文件系统镜像。

### UBI 和 UBIFS

JFFS2 和 YAFFS2 有一个主要问题：磨损均衡由文件系统本身实现，意味着磨损均衡只局限于单个分区。在许多系统中，有只读分区，或至少很少更新的分区，例如程序和库，与获得大部分写入的其他读写数据区域相反。这些"热点"(hot)分区比所有闪存区块参与磨损均衡的情况更早磨损。这正是无序块镜像（UBI，Unsorted Block Images）项目所提供的。

UBI 是位于 MTD 之上的一层，负责管理擦除块，在整个设备上实现磨损均衡和坏块管理。这样，上层不再需要自己处理这些任务。UBI 还支持灵活的分区或卷(volumes)，可以动态创建和调整大小，方式类似于块设备的逻辑卷管理器(Logical Volume Manager)。

UBI 通过实现"逻辑擦除块"（LEBs，Logical Erase Blocks）来工作，映射到"物理擦除块"（PEBs，Physical Erase Blocks）。上层只看到 LEB。如果一个 LEB 被写入得过于频繁，UBI 可以决定交换指针，用"冷"PEB 替换"热"PEB。这个机制需要一些空闲 PEB 才能高效工作，这种开销使 UBI 不太适合只有几 MB 空间的小型设备。

![ubi.png](https://bootlin.com/wp-content/uploads/2012/12/ubi.png)

UBIFS 是 UBI 的文件系统。它由 Linux MTD 项目创建，作为 JFFS2 的继任者。它也支持压缩，并有更好的挂载、读和写性能。

## 参考(References)
[Managing flash storage with Linux](https://bootlin.com/blog/managing-flash-storage-with-linux/)

[Flash (SSD) Technology (And Beyond) Fundamentals — So-Cal Engineer](http://socal-engineer.com/computing-blog/2014/4/14/flash-ssd-technology-and-beyond-fundamentals)

[2.1.1 Flash Memory](http://www.iue.tuwien.ac.at/phd/windbacher/node14.html)

[闪存基础(转) \| 陈浩的个人博客](http://cighao.com/2016/02/25/fundamental-of-flash-memory/)

[](https://blog.csdn.net/tigerjibo/article/details/9322035)

[Flash 101: NAND Flash vs NOR Flash - Embedded.com](https://www.embedded.com/flash-101-nand-flash-vs-nor-flash/)
