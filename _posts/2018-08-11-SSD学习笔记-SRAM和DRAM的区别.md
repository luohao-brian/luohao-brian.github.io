---
layout: post
title: SSD学习笔记-SRAM和DRAM的区别
tags: [存储]
---

DRAM(Dynamic RAM)和SRAM(Static RAM)的共同点是一旦掉电，保存的信息会丢失。

DRAM利用电容来保存信息，以即电容端电压的高低来表示1和0。它的集成度较高，读写功耗较低；但是，保存在DRAM中的信息随着电容器的漏电而会逐渐消失，一般信息保存时间为2ms左右。为了保存DRAM中的信息，必须每隔1～2ms对其刷新一次，在我们的PC待机时消耗的电量有很大一部分都来自于对内存的刷新。DRAM一般用作计算机中的主存储器。

SRAM的特点是工作速度快，只要电源不撤除，写入SRAM的信息就不会消失，不需要刷新电路，只要一个纽扣电池就可以保存数年之久；但集成度较低，读写功耗较大。SRAM一般用来作为计算机中的高速缓冲存储器(Cache)。


![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ram-2.jpeg)


SRAM的性能比DRAM高出数个量级，以下是i7 CPU Cache和内存的延时对比：

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ram-1.jpeg)


