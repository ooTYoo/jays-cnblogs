----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的eFUSE**。  

　　在i.MXRT启动系列第二篇文章 [Boot配置(BOOT Pin, eFUSE)](http://www.cnblogs.com/henjay724/p/9034563.html) 里痞子衡提到了eFUSE，部分Boot配置都存储在eFUSE memory里，但是对eFUSE的介绍仅仅浅尝辄止，没有深入，今天痞子衡就为大家再进一步介绍eFUSE。  

　　eFUSE是i.MXRT里一块特殊的存储区域，用于存放全部芯片配置信息，其中有一部分配置信息和Boot相关。这块特殊存储区域并不在ARM的4G system address空间里，需要用特殊的方式去访问（读/写），如何访问eFUSE是本篇文章的重点。  

### 一、eFUSE基本原理
#### 1.1 eFUSE属性（OTP, Lock）
　　eFUSE本质上就是i.MXRT内嵌的一块OTP（One Time Programmable） memory，仅可被烧写一次，但可以被多次读取。<font color="Blue">eFUSE memory的烧写是按bit进行的，初始状态下所有eFUSE bit均为0，通过特殊的烧写时序可以将bit从0改成1，一旦某bit被烧写成1后便再也无法被修改（可理解为硬件熔丝烧断了无法恢复）</font>。  
　　i.MXRT的eFUSE memory总地址空间有1.75KB（地址范围为0x000 - 0x6FF），但可读写操作的空间只有192bytes（位于0x400 - 0x6FF区域），分为6个BANK，每个BANK含8个word（1word = 4bytes）。下图中0x00 - 0x2F是eFUSE的bank word索引地址（也叫index地址），其与eFUSE空间地址对应关系是：  
> fuse_address = fuse_index * 0x10 + 0x400  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_otp_footprint.PNG" style="zoom:100%" />

　　上述可读写的eFUSE memory空间除了OTP特性外，还有Lock控制特性，<font color="Blue">Lock控制是OTP memory的标配，Lock控制有三层：第一层是WP，即写保护，用于保护那些不需要被烧写成1的eFUSE bit；第二层是OP，即覆盖保护，包含WP功能，并且被保护的eFUSE区域对应的shadow register也不能被重写；第三层是RP（WP+OP+RP），即访问保护，被保护的eFUSE区域及其对应的shadow register均不能被读写。</font> Lock控制在eFUSE的BANK0_word0，如下是具体Lock bit定义：  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_otp_lock_area.PNG" style="zoom:100%" />

> 关于可读写eFUSE空间所有bit定义详见Reference Manual里的Table 5-9. Fusemap Descriptions。  

#### 1.2 OCOTP控制器与Shadow Register
　　i.MXRT内部有一个硬件IP模块叫OCOTP_CTRL，即OCOTP控制器，对eFUSE memory的读写控制操作其实都是通过这个OCOTP控制器实现的，下图是OCOTP_CTRL模块图：  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_ocotp_diagram.PNG" style="zoom:100%" />

　　OCOTP_CTRL模块寄存器一共分两类：一类是IP控制寄存器，用于实现对OTP memory的读写操作时序控制；一类是<font color="Blue">Shadow register，用于上电时自动从eFUSE memory获取数据并缓存，这样我们可以直接访问Shadow register而不用访问eFUSE memory也能获取eFUSE内容（注意：当芯片运行中烧写eFUSE，Shadow register的值并不会立刻更新，需要执行IP控制器的reload命令或者将芯片reset才能同步）。</font>  

　　IP控制寄存器偏移地址范围是0x000 - 0x3FF(下图仅截取部分):  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_otp_reg_ctrl.PNG" style="zoom:100%" />

　　Shadow register寄存器偏移地址范围是0x400 - 0x6FF（下图仅截取部分），看到0x400 - 0x6FF的地址范围，有没有感觉很熟悉？是的，这跟上一节讲的可读写操作eFUSE空间偏移地址范围是一致的。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_otp_reg_shadow.PNG" style="zoom:100%" />

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_efuse_otp_reg_shadow_end.PNG" style="zoom:100%" />

### 二、使用blhost烧写eFUSE
　　eFUSE memory的烧写是通过OCOTP_CTRL模块来实现的，我们当然可以在Application中集成OCOTP_CTRL的驱动程序，然后在Application调用OCOTP_CTRL的驱动程序完成eFUSE的烧写，但这种方式并不是痞子衡要介绍的重点，痞子衡要介绍的是通过Flashloader配套的blhost.exe上位机工具实现eFUSE的烧写。  
　　痞子衡在上一篇文章里介绍过如何引导启动Flashloader并且使用blhost与Flashloader通信，此处假设你已经使用blhost与Flashloader建立了通信。让我们再来回顾一下blhost的命令help，可以得知efuse-program-once这个命令就是我们想要的命令。  

```text
PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> .\blhost.exe -?
usage: C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win\blhost.exe
                       [-p|--port <name>[,<speed>]]
                       [-u|--usb [[[<vid>,]<pid>]]]
                       -- command <args...>

Command:
  efuse-program-once <addr> <data>
                               Program one word of OCOTP Field
                               <addr> is ADDR of OTP word, not the shadowed memory address.
                               <data> is hex digits without prefix '0x'
  efuse-read-once <addr>
                               Read one word of OCOTP Field
                               <addr> is ADDR of OTP word, not the shadowed memory address.

```

　　让我们试一下efuse-program-once这个命令，开始试之前要解决2个问题：  
　　addr参数到底是什么地址？帮助里说是OTP word address，其实这个地址就是1.1节里介绍的fuse_index，index范围为0x00 - 0x2F，对应48个可读写操作的eFUSE Word。  
　　data参数到底是什么格式？帮助里说是hex digits without prefix '0x'，但是似乎没有指明长度，我们知道每一个index对应的是4byte，那就应该是8位16进制数据（实测下来必须要填8位，如果是非8位会返回Error: invalid command or arguments）。  
　　弄清了问题，那我们做一个小测试：要求将eFUSE里的SRK_REVOKE word的最低byte烧写成0x5A，然后再将最高byte烧写成0xFE，分两步进行。  
　　翻看OTP Memory Footprint表，找到SRK_REVOKE的index地址是0x2F（对应Shadow register地址是0x401F46F0），命令搞起来：  

> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u -- efuse-program-once 0x2F 0000005A</font>
> ```text
> Inject command 'efuse-program-once'
> Successful generic response to command 'efuse-program-once'
> Response status = 0 (0x0) Success.
> ```
>
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u -- efuse-program-once 0x2F FE000000</font>
> ```text
> Inject command 'efuse-program-once'
> Successful generic response to command 'efuse-program-once'
> Response status = 0 (0x0) Success.
> ```
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u -- efuse-read-once 0x2F</font>
> ```text
> Inject command 'efuse-read-once'
> Response status = 0 (0x0) Success.
> Response word 1 = 4 (0x4)
> Response word 2 = -33554342 (0xfe00005a)
> ```
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u -- read-memory 0x401F46F0 4</font>
> ```text
> Inject command 'read-memory'
> Successful response to command 'read-memory'
> 5a 00 00 fe
> (1/1)100% Completed!
> Successful generic response to command 'read-memory'
> Response status = 0 (0x0) Success.
> Response word 1 = 4 (0x4)
> Read 4 of 4 bytes.
> ```

　　看起来命令执行正常，但你是不是会有几个疑问：
　　为何执行第二条命令将0xFE000000烧写进eFUSE时没有报错？显然第一条命令已经将0x0000005A烧写进eFUSE，而0xFE000000的最低byte是0x00，看起来它跟已经烧写进去的0x5A是冲突的，而前面介绍过eFUSE bit只能从0烧写为1，其实这不是问题，OCOTP controller会自动过滤将eFUSE bit从1烧写为0的操作。  
　　为何eFUSE被烧写后，并没有reset操作，用read-memory去获取Shadow register可以立即看到数据同步更新了？其实blhost里的efuse-program-once命令不仅包含program命令，也自动集成了reload命令。  

　　虽然只有blhost可以实现eFUSE烧写功能，但要获取eFUSE状态并不是只有blhost可以做到，sdphost也可以做到，因为sdphost提供了读写Shadow register的命令。  

> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- read-register 0x401F46F0</font>
> ```text
> 5a 00 00 fe
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> ```
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- write-register 0x401F46F0 32 0x00000000</font>
> ```text
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> Reponse Status = 311069202 (0x128a8a12) Write complete.
> ```
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- read-register 0x401F46F0</font>
> ```text
> 00 00 00 00
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> ```

　　至此，飞思卡尔i.MX RT系列MCU的eFUSE痞子衡便介绍完毕了，掌声在哪里~~~ 

