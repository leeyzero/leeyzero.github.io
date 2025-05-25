---
title: DirectUI的一些思考
date: 2020-10-10 23:11:56
categories:
- 编程匠艺
tags:
- 系统设计
- DirectUI
- Windows
---

对于传统Win32界面编程来讲，微软提供一整套界面标准，比如窗口、按钮、滚动条、列表等。对于每一个窗口（控件也是一个窗口），其能响应的消息和行为都有规范（通过API提供给开发者）。微软这套界面标准是为通用场景下提出的解决方案，能够满足绝大部分需求，但业务场景的多样性，使得开发者们并不满足于这套界面标准。

## DirectUI的发展历史

- *2005年6月，Bjarke Viksoe发布了一篇文章[UI: Become windowless](http://www.viksoe.dk/code/windowless1.htm), 阐述了无窗口句柄（windowless）思想。*
- *2010年12月，金山网络宣布启动[金山卫士开源计划](http://code.ijinshan.com/)，该开源项目以失败告终，但有热心的网友从该项目中分离了金山卫士的界面部分成为一个独立的项目[bkwin](http://code.ijinshan.com/index.html)。*
- *2012年之后，由bkwin项目衍生出各种基于DirectUI思想的开源框架，如[Duilib](https://github.com/duilib/duilib)、[DuiEngine](https://github.com/kevinzhwl/duiplant/tree/master/src/DUIEngine)、[DuiVision](https://github.com/blueantst/DuiVision)、[SOUI](https://github.com/SOUI2/soui)等。*

## 什么是DirectUI
**DirectUI是一种界面开发思想**。其核心思想是指将所有的界面控件都绘制在一个窗口上，这些控件的逻辑和绘制方式都必须自己进行编写和封装，而不是使用Windows的原生控件，所以这些控件都是**无句柄的（Windowsless）**。

那这个名称是怎么来的呢？由于Windows有句柄窗口是一套工业标准，窗口消息和API都是公开的，所有人都知道怎么操作窗口。微软在做MSN的时候为了保护用户隐私，搞了一个DirectUIHWND，后边DirectUI这个名字就被沿用下来，后边说的DirectUI一般都是指**无句柄窗口**。

<!--more-->

## DirectUI需要解决的问题
DirectUI实际是在Windows的原生窗口基础上，更细粒度地进行窗口控制，它需要建立一套自己DirectUI标准，主要需要解决以下问题：

- *窗口子类化，截获窗口消息；*
- *封装自己的控件，并将控件绘制到窗口上；*
- *封装窗口消息，并分发到自己的控件上，让自己的控件根据消息进行相应绘制；*
- *根据不同行为发送自己定义消息给窗口，以便客户程序处理；*
- *界面与逻辑分离，一般使用XML来描述窗口上控件布局；*

## DirectUI的优势

相较于传统Win32界面，DirectUI技术有以下优势：

- **界面逻辑完全分离**：DirectUI将界面布局完全分离出来（通常采用XML进行描述），将逻辑与界面解耦，符合界面设计原则。
- **防止软件被破解**：由于DirectUI是建立在Win32界面标准之上，其DUI窗口（子控件也是一个DUI窗口）的消息转发和消息处理因框架设计的不同而异，这些内部逻辑对外不透明，很难破解。我想这也是很多大厂不想开源其界面框架的原因之一吧。
- **运行效率更高**：这个取决于DirectUI框架的实现。由于是在框架内部进行消息转发和处理，并不需要经过系统，理论上效率会更高。
- **更容易实现绚丽效果和换肤功能**：DirectUI框架提供标准控件的同时也提供良好的扩展性。业务层在框架基本上可以很容易定制自己的控件。

## DirectUI框架
对于互联网桌面端产品，目前大部分还是使用的DirectUI，对于大公司而言，都有一套自己的界面框架，如腾讯、360。但这些公司并没有把这些界面框架开源出来。不过迅雷开源自己的界面引擎[bolt](http://bolt.xunlei.com/)。
另外也有一些个人开发者维护的比较优秀的开源界面库，在众多开源库中，比较活跃的还是[SOUI](https://github.com/SOUI2/soui)，具体请参考SOUI界面库作者的[博客](https://www.cnblogs.com/setoutsoft/category/600691.html)。

## 从另一个角度看DirectUI

UI承载着跟用户交互的职能。从早期二进制打孔到命令行到现在的窗口可视化，其本质是降低使用计算机的门槛，提高使用计算机的效率。而从UI发展来看，DirectUI只是在有句柄的窗口上进行更细粒度的控制；再往上抽象来看，Windows系统制定窗口标准，实际上是对显存更细粒度的控制；**不管是有句柄窗口还是无句柄窗口，其目的无非是提高计算机跟用户的交互能力和开发效率**。从这一点来看，DirectUI在交互能力和开发效率上都有所提高，但其本质没有任何变化。从开发效率来看，它肯定比不上WPF；从交互能力来讲，传统的UI交互有被语音交互取代的趋势。

## 参考资料

[1] http://www.viksoe.dk/code/windowless1.htm
[2] http://www.blueantstudio.net/duivision.html
[3] https://www.cnblogs.com/setoutsoft/p/4996870.html
