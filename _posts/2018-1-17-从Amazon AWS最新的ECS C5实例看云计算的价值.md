---
layout: post
title: 从Amazon AWS最新的ECS C5实例看云计算的价值
tags: [云计算]
---

> AWS作为云计算的旗帜，于2015.1发布了上一代的计算密集实例C4，就在前几天发布了C4的下一代产品C5, 我们先来看看这两款产品的差别：

-------
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/3.jpeg)

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/4.jpeg))

### 基础设施

两年多来，AWS在计算密集型实例做了以下主要优化:

1. CPU从Intel Xeon V3升级到Xeon Platinum V5, 基础频率从 2.6G提升至3.0G；支持最新的AVX512指令集，较V3的峰值浮点计算能力提升一倍以上;
2. 最大虚拟机规格从36 vCPU提升至72 vCPU;
3. 最大网络带宽从10G提升至25G, 全面采用25G核心交换；
4. 最大存储带宽从4G提升至9G;

但凡有经验的数据中心运维工程师，如果说升级服务器还是一件非常有挑战的任务，那么对于升级核心交换，通常可以认为是一件不可能完成的事情。而在公有云看来，这个节奏竟然可以在两年多完成，而且预计以后这样的节奏会越来越快。基础设施好了，道路修宽了，各种新型的业务自然也就可以发展起来了。这跟我们说的想致富，先修路其实是一个道理。传统的自建数据中心，在这样的迭代节奏面前，还能坚持多久呢？

### 虚拟化和操作系统

从C5开始，AWS全面采用KVM作为其虚拟化引擎，并且通过Elastic Network Adapter (ENA) 只能网卡和NVMe智能卡全面卸载了传统的基于软件实现网络虚拟化和存储虚拟化, 我个人认为，这基本也是数据中心虚拟化的最终形态，目前网络虚拟化普遍使用的ovs+vhost以及存储虚拟化普遍使用的virtio-blk，需要在主机部署业务和管理程序，运行时生成大量进程，比如一个KVM虚拟机，至少需要如下进程和线程:
1. qemu vcpu线程
2. qemu主线程
3. qemu io线程
4. vhost
5. ovs
6. dnsmasq
7. ...

我们需要严格的去隔离这些进程和vcpu进程的运行范围，才能严格的保障业务的QoS, 而采用IO完全卸载的方式，既可以极大的简化安装部署复杂度，又可以提供更精确的QoS保障。

在C5的实例中，我们会看到网卡是一个ENA设备，而通过EBS创建和挂载的卷，会呈现为一个NVME设备。

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/3.png)
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/3.png)

由于需要特殊的驱动支持，目前只有AWS发布的官方镜像能支持C5实例，包括: Microsoft Windows (Windows Server 2012 R2 and Windows Server 2016), Ubuntu, RHEL, CentOS, SLES, Debian, and FreeBSD。
