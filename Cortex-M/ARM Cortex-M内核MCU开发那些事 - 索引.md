----
　　大家好，我是痞子衡，是正经搞技术的痞子。本系列痞子衡给大家介绍的是**ARM Cortex-M内核微控制器相关知识**。  

### 资料篇（持续更新中...）
> [史上最强ARM Cortex-M学习资源汇总(持续更新中...)](http://www.cnblogs.com/henjay724/p/8717135.html)  

### 内核篇（持续更新中...）
　　迄今为止（2017），ARM公司发布的最强Cortex-M内核是Cortex-M7，最新的Cortex-M内核是Cortex-M33，这两款内核基本涵盖了Cortex-M内核的所有特性，其功能模块框图如下：  

<table><tbody>
    <tr>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/csdn_blog/%E4%BB%8E%E5%8A%9F%E8%83%BD%E6%A8%A1%E5%9D%97%E8%A7%92%E5%BA%A6%E7%9C%8BCortex-M%E5%90%84%E5%A4%84%E7%90%86%E5%99%A8%E5%8C%BA%E5%88%AB/Cortex-M7-chip-diagram-16.png" style="zoom:100%" /></td>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Cortex-M33-chip-diagram-16.PNG" style="zoom:100%" /></td>
    </tr>
</table>  
　　内核系列文章会逐一介绍这两款内核的所有功能模块，在介绍的同时也会拓展到Cortex-M全系列内核的具体差异，力求一个系列全文将Cortex-M特点全部展现出来。  
> [ARM内核处理器知识概要杂辑（1）- 内核架构编年史](http://www.cnblogs.com/henjay724/p/8408775.html)  
> [ARM内核处理器知识概要杂辑（2）- 第一款Cortex-M微控制器](http://www.cnblogs.com/henjay724/p/8408904.html)  
> [ARM内核处理器知识概要杂辑（3）- Cortex-M功能模块](http://www.cnblogs.com/henjay724/p/8408825.html)  
> [ARM内核处理器知识概要杂辑（4）- Cortex-M性能指标](http://www.cnblogs.com/henjay724/p/8408915.html)  
> [ARM内核处理器知识概要杂辑（5）- Cortex-M指令集](http://www.cnblogs.com/henjay724/p/8763171.html)  

> [ARM内核处理器知识概要杂辑（6）- Cortex-M总线(AHB/APB/AXI)]()  
> [ARM内核处理器知识概要杂辑（7）- Cortex-M内存保护(MPU)]()  
> [ARM内核处理器知识概要杂辑（8）- Cortex-M浮点计算(FPU)]()  
> [ARM内核处理器知识概要杂辑（9）- Cortex-M数字信号处理(DSP)]()  
> [ARM内核处理器知识概要杂辑（10）- Cortex-M安全区域(TrustZone)]()  

### 中断篇（持续更新中...）
> [ARM Cortex-M中断那些事（0）- 索引]()  

### 功耗篇（持续更新中...）
> [ARM Cortex-M低功耗那些事（0）- 索引]()  

### 调试篇（持续更新中...）
　　嵌入式开发离不开调试，调试技巧是嵌入式开发人员必备技能，好的调试工具和方法往往使得嵌入式应用开发事半功倍。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/ARM_debuging.PNG" style="zoom:100%" />
　　调试系列文章会逐一介绍ARM Cortex-M开发中所用到的所有调试相关知识。  
> [ARM Cortex-M调试那些事（1）- 4线协议标准(JTAG)](http://www.cnblogs.com/henjay724/p/8447578.html)  

> [ARM Cortex-M调试那些事（2）- 2线协议标准(SWD)]()  
> [ARM Cortex-M调试那些事（3）- CoreSight架构]()  
> [ARM Cortex-M调试那些事（4）- DAPLink调试器]()  
> [ARM Cortex-M调试那些事（5）- Jlink仿真器]()  
> [ARM Cortex-M调试那些事（6）- IAR内嵌调试C-SPY]()  
> [ARM Cortex-M调试那些事（7）- flashloader]()  
> [ARM Cortex-M调试那些事（8）- 常用技巧]()  

### 文件篇（已完结）
　　文件系列文章以IAR集成开发环境开发ARM Cortex-M微控制器为例，其他环境可触类旁通。  
编译阶段：
> [ARM Cortex-M文件那些事（1）- 源文件(.c/.h/.s)](http://www.cnblogs.com/henjay724/p/8183257.html)  
> [ARM Cortex-M文件那些事（3）- 工程文件(.ewp)](http://www.cnblogs.com/henjay724/p/8232585.html)  
> [ARM Cortex-M文件那些事（4）- 可重定向文件(.o/.a)](http://www.cnblogs.com/henjay724/p/8276595.html)  
![编译](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/IAR%20build%20process%20-%20translation.PNG)
链接阶段：
> [ARM Cortex-M文件那些事（2）- 链接文件(.icf)](http://www.cnblogs.com/henjay724/p/8191908.html)  
> [ARM Cortex-M文件那些事（5）- 映射文件(.map)](http://www.cnblogs.com/henjay724/p/8276648.html)  
> [ARM Cortex-M文件那些事（6）- 可执行文件(.out/.elf)](http://www.cnblogs.com/henjay724/p/8276677.html)  
![链接](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/IAR%20build%20process%20-%20linking.PNG)
下载阶段：
> [ARM Cortex-M文件那些事（7）- 反汇编文件(.s/.lst/.dump)](http://www.cnblogs.com/henjay724/p/8288992.html)  
> [ARM Cortex-M文件那些事（8）- 镜像文件(.bin/.hex/.s19)](http://www.cnblogs.com/henjay724/p/8361693.html)  
![下载](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/IAR%20build%20process%20-%20after%20linking.PNG)
