---
layout: post
title: Intel Uncore
tags: [硬件平台]
---

### 背景

uncore一词，是英特尔用来描述微处理器中，功能上为非处理器核心（Core）所负担，但是对处理器性能的发挥和维持有必不可少的作用的组成部分。处理器核心（Core）包含的处理器组件都涉及处理器指令的运行，包括算术逻辑单元（ALU）、浮点运算单元（FPU）、一级缓存（L1 Cache）、二级缓存（L2 Cache）。Uncore的功能包括QPI控制器、三级缓存（L3 Cache）、内存一致性监测（snoop agent pipeline）、内存控制器，以及Thunderbolt控制器。至于其余的总线控制器，像是PCI-E、SPI等，则是属于芯片组的一部分。

英特尔Uncore设计根源，来自于北桥芯片。Uncroe的设计是，将对于处理器核心有关键作用的功能重新组合编排，从物理上使它们更靠近核心（集成至处理器芯片上，而它们有部分原来是位于北桥上），以降低它们的访问延时。而北桥上余下的和处理器核心并无太大关键作用的功能，像是PCI-E控制器或者是电源控制单元（PCU），并没有集成至Uncore部分，而是继续作为芯片组的一部分。

具体而言，微架构中的uncore是被细分为数个模块单元的。uncore连接至处理器核心是通过一个叫Cache Box（CBox）的界面实现的，CBox也是末级缓存（Last Level Cache，LLC）的连接界面，同时负责管理缓存一致性。复合的内部与外部QPI链接由物理层单元（Physical Layer units）管理，称为PBox。PBox、CBox以及一个或更多的内置存储器控制器（iMC，作MBox）的连接由系统配置控制器（System Config Controller，作UBox）和路由器（Router，作RBox）负责管理。

从uncore部分移出串列总线控制器，可以更好地促进性能的提升，通过允许uncore的时钟频率（UCLK）运作于基准的2.66GHz，提升至超过超频限制值的3.44GHz，实现性能提升。这种时脉提升使得核心访问关键功能部件（像是存储器控制器）时的延时值更低（典型情况下处理器核心访问DRAM的时间可降低10纳秒或更多）。

### 架构

下面是较早的Intel® Xeon® Processor E7-8800/4800/2800系列CPU Uncore架构图以及各组件说明：

![Intel® Xeon® Processor E7-8800/4800/2800 Product Families Block Diagram](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/core-uncore.png)


Name | Function
---|---
Core|Intel Xeon Processor E7-8800/4800/2800 Product Families core architecture processing unit
Bbox|Home Agent
Cbox|Last Level Cache Coherency Engine
Mbox|Integrated Memory Controller
Pbox|Physical Layer (PHY) for the Intel® QPI Interconnect and Intel® SMI Interconnect memory controller
Rbox|Crossbar Router
Sbox|Caching Agent
Ubox|System Configuration Agent
Wbox|Power Controller
Intel® SMI|Intel® Scalable Memory Interconnect (formerly “FBD2” or “Fully Buffered DIMM 2 interface”)
Intel® QPI|Intel® QuickPath Interconnect
LLC|Last Level Cache (Level 3)

这是较新的Intel® Xeon® E5系列Uncore架构图，看起来HA(Home Agent)取代了Bbox, iMC取代了Mbox, PCU取代了WBox。

![Intel® Xeon® E5 series block diagram](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/core-uncore-pcm.png)


### 参考

 * [Intel® Performance Counter Monitor - A better way to measure CPU utilization](https://software.intel.com/en-us/articles/intel-performance-counter-monitor)
 * [Intel Uncore Wiki](https://zh.wikipedia.org/wiki/Uncore)
 * [Intel® Xeon® Processor E7-
8800/4800/2800 Product Families
Datasheet Volume 2 of 2](https://manualsbrain.com/en/manuals/312844/pdf/7d577a522369147a0d54421c1f92ec9d6de9c175e86b6a35ba26ee2dd0476857/cisco-intel-xeon-e7-4860-ucs-cpu-e74860-user-manual.pdf)
