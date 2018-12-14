----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**关于开源软件协议基本知识**。  

　　牛顿曾说过：“如果我比别人看得更远,那是因为我站在巨人的肩上”。在软件开发中如果说也存在巨人的肩膀让我们站，我想这个巨人应该就是开源软件。一个优秀的软件开发人员应该能够善于学习和利用开源软件来加速自己的开发，而为了正确地使用开源软件，我们必须要了解开源软件协议，今天我们就来聊一聊开源软件协议这个话题。

### 1.开源软件是什么？

<img src="http://henjay724.com/image/csdn_blog/Open_Source_Software_Logo2.jpeg" style="zoom:110%" />

　　所谓“**开源软件**”（open-source software），字面上理解就是开放源代码的软件，即在软件发行的时候，附上软件的源代码，并授权允许用户更改/自由再散布/衍生著作权。

　　开源软件通常是有copyright（著作权）的，它的License（许可证）可能包含这样一些额外的限制： 刻意的保护它的开放源码状态，著者身份的公告，或者开发的控制。

　　提及开源软件，通常会想到两个形容词：免费、自由。开源软件大多是免费的（但也并不排斥商业收费），开源软件的使用往往也是相对自由的（自由度取决于其License）。因此常常会与开源软件造成混淆和误解的有另外两个概念：“**免费软件**”、“**自由软件**”

> * 免费软件：免费提供给用户使用的软件，通常并不包含公开其源码的内容。
> * 自由软件：在反著作权，倡导软件这种知识产品应该免费共享的思想下诞生的软件，当然必须是要开放源代码的。一个专用的名词copyleft（著佐权）就是用来形容这种软件。

　　除了前面介绍的3种类型软件之外，还有一种概念：“**商业软件**”，即被用作商品以达到营利目的收费软件，这种软件一般不会包含源码，并且受各种严苛的版权限制。

　　从上面的介绍可以看出，自由软件与商业软件是完全对立的，而开源软件就是自由软件与商业软件的折中，它既继承了“自由软件”所提倡的知识共享的理念，同时又允许人们以专利的形式从知识产品中谋取利益，从而保护了人们生产、创造知识产品的积极性。

### 2.为什么会存在开源软件？
　　在讲为什么会存在开源软件之前，我们应该先讲讲商业软件（私有软件）到底有哪些弊端，下面是两点可能存在的弊端：

> * 不利于软件社会价值的发挥：如果一个软件已经开发完成，即意味着对软件开发所需要投入的社会资源已经全部付出了，底下就应该是软件回报社会的时候了，但软件私有限制了软件对社会的贡献度，导致一定程度的资源浪费。
> * 不利于对软件的使用和开发：从用户角度来看：软件私有是不利于其被进一步使用的，如果现有软件有不完善的地方，用户无法基于现有软件作少许改动以实现进一步需求；而从开发角度来看：软件私有意味着开发软件的只能是特定的一群人，很难轻易采集到广大使用者和二次开发者对软件发展的意见。

　　既然存在开源软件，说明软件开源肯定能带来好处，那具体能帮我们解决什么问题呢？

> * 从用户使用的角度：使用户能根据自己的需要来使用、定制软件。
> * 从软件本身的角度：开源，让更多的人参与，更有助于软件的完善，开发出更优秀的软件。
> * 从软件行业的角度：极大的提高软件开发的生产力，我们能够自由的复用别人的开发成果，而避免重复劳动。

　　总之，软件开源的目的是为了让软件能得到最大范围的使用。

### 3.一大波开源软件协议

<img src="http://henjay724.com/image/csdn_blog/Open_Source_Software_License_List.jpeg" style="zoom:90%" />

　　前面讲到，开源软件都是有License的，猜一下，迄今为止，世界上一共有多少种开源软件License？据粗略统计有上百种（[GNU组织整理的开源协议清单](http://www.gnu.org/licenses/license-list.html)），而通过OSI（Open Source Initiative）组织批准的开源软件协议目前也有60多种（[OSI组织批准的开源协议清单](https://opensource.org/licenses/alphabetical)）

　　虽然有这么多开源软件协议，但我们只需要了解其中最常用的几种就足够了，常用的开源软件License有如下6种：

> * **MIT**（The MIT License）：源自麻省理工学院，又称“X条款”（X License）或“X11条款”（X11 License）。
> * **BSD**（Berkly Software Distribution）：源自加州大学柏克利分校，最初是用于该校发表的各个4.4BSD/4.4BSD-Lite版本。
> * **Apache**（Apache License）：著名的非盈利开源组织Apache采用的协议。
> * **LGPL**（ Lesser General Public License）：GNU组织制定的宽松通用公共许可证。
> * **MPL**（The Mozilla Public License）：Netscape的Mozilla小组为其开源软件项目设计的软件许可证。
> * **GPL**（General Public License）：GNU组织制定的通用公共许可证，由斯托曼撰写，最初用于GNU计划。

### 4.看懂常见开源软件协议

　　前面介绍了6种常见开源协议的名称及由来，要去了解每个License具体限制（开放源码状态，著者身份的公告，开发的控制），我们可以去一句一句去读晦涩的License原文，但是还有更简单的方式迅速区分它们，下面是用于迅速区分的的5个特性：

> * 闭源允许：基于开源软件进行二次开发的衍生软件是否可以闭源？
> * 版权声明：修改过的开源软件文件是否必须放置原开源软件版权说明？
> * 品牌推销：衍生软件是否可以用原开源软件的品牌影响力进行推广？
> * 继承机制：基于开源软件开发的新增文件是否也需要采用原开源软件的License？
> * 改动申明：对开源软件文件的修改是否需要提供说明文档？

　　按照以上5个特性，我们可以迅速将6种开源软件协议分类，详见下图：

<img src="http://henjay724.com/6_free_software_licenses.png" style="zoom:80%" />

　　从开源软件的个人使用灵活角度来看：MIT是最自由最没有限制的，MIT协议的开源软件作者只想保留版权，其他方面任你自由发挥；而GPL限制是最严苛的，如果你用了GPL开源软件，那么你的软件也必须同样以GPL协议方式开源。

　　从开源软件的社会传播影响角度来看：GPL最能推动知识共享，任何基于GPL开源软件开发的新成果，都能被大众轻易借鉴和分享；而MIT仅有助于分享开源软件本身的成果，基于MIT开源软件开发的新成果往往被二次开发者私有。

### 5.常见开源软件协议之间兼容性

　　如果你已经了解了前面介绍的内容，那么你现在应该能够轻松搞定基于单一开源软件的二次开发的License问题，但在实际使用中，你的项目可能会存在引用多个开源软件，此时便涉及到开源软件协议之间的兼容性问题，即需要考虑两个核心问题：

> * 多个开源软件是否可以一起组合使用？
> * 多个开源软件的使用最终如何确定License？

　　开源软件协议从使用限制强弱上来看，可以分为三大类：放任型、弱保护型、强保护型；一般来讲强限制协议可以向下兼容弱限制协议（这意味着软件最终License取决于强限制协议），但存在限制条件完全对立的两个协议则无法兼容（这意味着软件开发不能同时引用这两个开源软件）。下图很好地说明了6种常见开源软件协议之间的兼容性情况。

<img src="http://henjay724.com/image/csdn_blog/Typical_License_Relationship.png" style="zoom:90%" />

　　箭头从A框到B框代表，A框和B框中的协议是兼容的（两种开源软件可以组合使用），且最终License取决于B框中协议；而如果两个框之间没有单向的箭头贯通，即意味着两个框中的协议不兼容（两种开源软件不可以组合使用）。

　　举例说明：MIT->BSD->Apache->LGPLv3->GPLv3是一个单向通路，这个通路上的任意两个及以上的开源软件都可以组合使用，软件最终License取决于通路上箭头最末端开源软件协议。MPL<-BSD->Apache是一个双向链路，链路两端的MPL和Apache协议是不兼容的，所以无法组合使用。

### 6.如何选择开源软件协议？

　　介绍到这里，开源软件协议这个话题也就基本结束了，其实你应该知道该如何选择合适的开源软件协议了，底下该是你去各大开源社区尽情淘你所需要的开源项目了，还在等什么？不过记住，如果找到了合适开源项目，请记得浏览一遍其License内容，说不定你会遇到惊喜，比如下面的这个WTFPL 2.0协议：

```text
            DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
                    Version 2, December 2004

Copyright (C) 2004 Sam Hocevar <sam@hocevar.net>

Everyone is permitted to copy and distribute verbatim or modified
copies of this license document, and changing it is allowed as long
as the name is changed.

            DO WHAT THE FUCK YOU WANT TO PUBLIC LICENSE
   TERMS AND CONDITIONS FOR COPYING, DISTRIBUTION AND MODIFICATION
  0. You just DO WHAT THE FUCK YOU WANT TO.
```

　　至此，关于开源软件协议基本知识痞子衡便介绍完毕了，掌声在哪里~~~ 

**参考资料**：

[1]. [开源软件,自由软件,免费软件三者的区别](http://blog.csdn.net/testcs_dn/article/details/37722355)

[2]. [科普:你该认识的四种常见开源许可证](http://mt.sohu.com/20160825/n465851394.shtml)

[3]. [9个主流的开源许可协议(整理)](http://univasity.iteye.com/blog/1292658)

[4]. [如何选择开源许可证？-阮一峰](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html)

[5]. [The Free-Libre / Open Source Software (FLOSS) License Slide](http://www.dwheeler.com/essays/floss-license-slide.html)

[6]. [最牛最暴力的开源协议：WTFPL](http://blog.csdn.net/testcs_dn/article/details/51099415)
