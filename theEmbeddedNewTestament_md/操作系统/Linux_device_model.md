---
tags:
  - 操作系统
source: https://github.com/theEmbeddedNewTestament/theEmbeddedNewTestament.github.io/tree/main/Operating_System/Linux/Linux_device_model.md
created: 2026-08-27
---

## Linux 设备模型(Device Model)

### 设备模型的一小部分

![[_assets/small_piece_device_model.png]]

### 架构图

![[_assets/linux_device_model_architecture.jpg]]

### 示例

#### I2C 充电器示例

![[_assets/linux_device_model_example.png]]

![[_assets/linux_device_model_example_2.png]]

![[_assets/linux_device_model_example_3.png]]

#### 谁调用 i2c_new_device?

![[_assets/linux_device_model_example_4.png]]

### 设备树(Device Tree) dtsi 文件

![[_assets/dtsi_example.png]]

![[_assets/dtsi_example_2.png]]

![[_assets/dtsi_parse.png]]

设备节点构建过程:

    of_platform_populate
        -> of_platform_bus_create
            -> of_platform_device_create_pdata
                -> of_device_add
		            ->device_add
                        ->bus_probe_device


### 参考(Reference)

_Linux Device Drivers, 3rd Edition_

http://www.slideshare.net/jserv/linux-discovery

http://m.blog.chinaunix.net/uid-29955651-id-5095220.html

http://www.cs.fsu.edu/~baker/devices/notes/ch14.html#(1)


