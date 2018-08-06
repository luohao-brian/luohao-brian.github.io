---
layout: post
title: SSD学习笔记-NOR, NAND, FTL, GC基本概念
tags: [存储]
---

### NOR v.s. NAND

两者都是非易失存储介质。即掉电都不会丢失内容, 在写入前都需要擦除。

NOR有点像内存，支持随机访问，这使它也具有支持XIP（eXecute In Place）的特性，可以像普通ROM一样执行程序。现在几乎所有的BIOS和一些机顶盒上都是使用NOR Flash，它的大小一般在1MB到32MB之间，价格昂贵。

NAND Flash广泛应用在各种存储卡，U盘，SSD，eMMC等等大容量设备中。

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ssd-1.jpg)

如果以镁光（Micron）自己的NAND和NOR对比的话，详细速度数据如下：

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ssd-2.jpg)


### NAND Flash

NAND Flash目前的用途更为广泛，它的颗粒根据每个存储单元内存储比特个数的不同，可以分为 SLC（Single-Level Cell）、MLC（Multi-Level Cell） 和 TLC（Triple-Level Cell） 三类。其中，在一个存储单元中，SLC 可以存储 1 个比特，MLC 可以存储 2 个比特，TLC 则可以存储 3 个比特。NAND Flash 的单个存储单元存储的比特位越多，读写性能会越差，寿命也越短，但是成本会更低。现在高端SSD会选取MLC甚至SLC，低端SSD则选取TLC。SD卡一般选取TLC。

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ssd-4.jpg)

### NAND Flash的组成
一个典型的Flash芯片由Package, Die, Plane, Block和Page组成，其中die内部可以通过3D 堆叠技术扩展容量，譬如三星的V-NAND每层容量都有128Gb（16GB），通过3D堆叠技术可以实现最多24层堆叠，这意味着24层堆叠的总容量将达到384GB！ 

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ssd-3.jpg)

### 写放大

Block是擦除操作的最小单位，Page是写入动作的最小单位，一个Block包含若干个Pages。当我们有了块干净的Flash，我们第一个想干的事就是写些东西上去，无论我们是写一个byte还是很多东西，必须以page为单位，即写一个byte上去也要写一个page。要修改一个字节，必须要擦除，擦除的最小单元是Block。

### Flash Translation Layer (FTL)

NAND flash的寿命是由其擦写次数决定的(P/E数 (Program/Erase Count)来衡量的)，频繁的擦除慢慢的会产生坏块。那么我们如何才能平衡整块Flash的整体擦写次数呢？这就要我们的FTL登场了。

![](https://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-08-03/ssd-5.jpg)

FTL简单来说就是系统维护了一个逻辑Block地址（LBA，logical block addresses ）和物理Block地址（PBA, physical block addresses）的对应关系。 有了这层映射关系，我们需要修改时就不需要改动原来的物理块，只需要标记原块为废块，同时找一个没用的新物理块对应到原来的逻辑块上就好了。

垃圾回收（GC，Garbage Collection）机制定期回收这些废块, 和Java，GO等语言的GC机制类似，应用不需要像C/C++那样关注内存释放，GC定期扫描，回收释放内存。目标是让Flash最小化擦除次数，最大化使用寿命。

### 参考

以下两篇知乎文章已经写的很精彩，本文基本是重新编辑版本。

- [杂谈闪存二：NOR和NAND Flash](https://zhuanlan.zhihu.com/p/26745577)
- [杂谈闪存三：FTL](https://zhuanlan.zhihu.com/p/26944064)

