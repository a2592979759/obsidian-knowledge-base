在project未被创建之前，所有的BSP包均存在于drivers中，依据与寄存器的封装。
F:\vivadoRelationSoft\SDK\2018.3\data\embeddedsw\XilinxProcessorIPLib\drivers

创建工程之后，生成的对应工程名的libsrc，也包含全部的库，囊括之后是全部包含的
C:\Users\Administrator\workspace\test_bsp\ps7_cortexa9_0\libsrc\gpiops_v3_4\src
C:\Users\Administrator\workspace\111_bsp\ps7_cortexa9_0\libsrc

所以，可以通过包含23链接的库然后使用1中的代码示例，直接进行功能验证。看在对应的编译器正常的情况下，是否可以直接可用。

vxworks的路径
E:\lqqSoftwareE\WindRiver3.3\WindRiver3.3\vxworks-6.9\target\src\drv

网络驱动一致，总共32个reg,前16个reg保持通用性，即可保证网络正常。