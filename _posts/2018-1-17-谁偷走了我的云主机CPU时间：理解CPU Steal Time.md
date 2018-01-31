---
layout: post
title: 谁偷走了我的云主机CPU时间：理解CPU Steal Time
tags: [虚拟化, 云计算]
---
> 虚拟化技术带来的最大直接收益是服务器整合（Server Consolidation）, 通过CPU, 内存，存储，网络的超分配（Overcommitment）技术，最大化服务器的使用率。一般企业会通过采购VMware虚拟化来做资源整合，而AWS, 阿里云，Azure等云提供商也分别采用了Xen， KVM，Hyper-V虚拟化技术用作其基础架构中物理资源整合。

------
### CPU超分配与CPU超卖
虚拟化的技能之一就是随心所欲的操控CPU, 比如，在一台服务器有2个CPU，每CPU 4个计算核心，虚拟化能在这台服务器上创建10台甚至20台8个计算核心的虚拟服务器。想象一下，原来一台服务器给1个系统使用，现在一台服务器可以给10个甚至20个系统使用，这台服务器的使用率不高才怪。如果我们把物理服务器想象成一条公路，以前这条路上只有一辆车，虚拟化其实并没有拓宽公路，只是允许更多的车能在这条公路上同时行进，但是，如果这条公路的车太多了，出现拥堵也就是很正常的事情。

公有云卖通用型云服务器的时候，一般都会超卖CPU，比如一台物理服务器有32个物理核心，在这台服务器上可能会最多创建128个1 VCPU的虚拟机，这时候的CPU超卖比就是1：4。一般来说，同厂商同规格的云服务器越便宜，超卖比越高。
### CPU超分配的影响
如果物理服务器的资源还有大量闲置，CPU超分配一般不会对运行在虚拟机中的业务产生明显的影响，但是如果这些超分配的VCPU大部分都处于高压力状态，物理服务器的PCPU已经接近饱和，那么各虚拟机的VCPU就要相互竞争，相互等待，从而各虚拟机中业务的延时增加，吞吐量下降。

各云服务商肯定不会对外公布他们的的CPU超卖方案，但是对于买家来讲，我们肯定不希望选到一个云服务器，结果它成天都在和其它的云服务器之间成天相互竞争，相互伤害，影响业务。
### CPU Steal Time
Linux的top和iostat命令，提供了Steal Time （st） 指标，用来衡量被Hypervisor偷去给其它虚拟机使用的CPU时间所占的比例，这个值越高，说明这台物理服务器的资源竞争越激烈，购买需谨慎。
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/7.jpg)

> top的 %Cpu(s) 中 st即Steal Time

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/8.jpg)

> iostat -c输出steal 指标

### CPU Steal Time试验
在PCPU 0-5上同时创建两个虚拟机，使用lookbusy软件对VM1的6个CPU分别按20%递增加压，对VM2的6个CPU都加压到50%，可以看出这时候的steal time整体就已经非常高了，对于最后一个cpu，steal time接近50%，说明运行在这个cpu上的任务最少比运行在物理服务器上增加了平均一倍的延时。
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/9.jpg)

> VM1 各CPU按20%递增加压

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/10.jpg)

> VM2所有CPU加压到50%负载

### CPU Steal Time实现
Linux CPU Steal Time的获取能力是Xen，KVM，VMware等社区或者厂商推到Linux社区的，Linux Guest OS通过HYPERCALL获得，相关代码应该10年前就陆续合入Linux主线，所以现在流行的CentOS 6/7, Ubuntu 14/16等都可以支持，但是对于一些很老的OS或者是比较小众的OS（freebsd，netbsd, plan9...)等，对steal time的支持可能会有问题，相关代码可以参看Xen社区提给Linux的补丁：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/11.jpg)