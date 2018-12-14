----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的文件关联**。  

　　本篇是文件系列第一篇，本系列文章会逐一介绍ARM Cortex-M开发过程中（以IAR集成开发环境为例，其他开发环境可触类旁通）所要接触的8种主要文件类型：源文件、链接文件、工程文件、可重定向文件、映射文件、可执行文件、反汇编文件、镜像文件。  
　　在介绍具体各文件之前有必要先让大家对各文件之间的关联有一个初步了解，下面三张图很好的诠释了8种文件之间的关联：  
#### 编译阶段：
![编译](http://henjay724.com/image/cnblogs/IAR%20build%20process%20-%20translation.PNG)

#### 链接阶段：
![链接](http://henjay724.com/image/cnblogs/IAR%20build%20process%20-%20linking.PNG)

#### 下载阶段：
![下载](http://henjay724.com/image/cnblogs/IAR%20build%20process%20-%20after%20linking.PNG)

　　至此，嵌入式开发里的文件关联痞子衡便介绍完毕了，掌声在哪里~~~ 