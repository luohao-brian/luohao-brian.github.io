---
layout: post
title: Linux系统优化与调试之二：压力测试神器stress-ng
categories: Linux
tags: [Linux]
---

> 工欲成其事，必先善其器，CentOS 7的EPEL源包含了2个压力测试工具，一个是标准的stress, 另外一个是更强大的stress-ng，可以帮助模拟产生各种cpu压力。

------


### 安装

```sh
yum install -y epel-release.noarch && yum -y update
yum install -y stress stress-ng
```

### stress

stress参数和用法都很简单：

*   -c 2 : 生成2个worker循环调用sqrt()产生cpu压力
*   -i 1 : 生成1个worker循环调用sync()产生io压力
*   -m 1 : 生成1个worker循环调用malloc()/free()产生内存压力

比如, 从下面可以看出经过30秒的压力后，系统负载从0.00提升至0.57。

```
[root@hluo ~]# uptime
 00:14:44 up 18 min,  1 user,  load average: 0.00, 0.01, 0.02
[root@hluo ~]# stress -c 2 -t 30
stress: info: [2312] dispatching hogs: 2 cpu, 0 io, 0 vm, 0 hdd
stress: info: [2312] successful run completed in 30s
[root@hluo ~]# uptime
 00:15:40 up 19 min,  1 user,  load average: 0.57, 0.18, 0.07
```

由于stress的压力模型非常简单，所以无法模拟任何复杂的场景，举个例子，在stress压测过程中，如果用top命令去观察，会发现所有的cpu压力都在用户态，内核态没有任何压力：

![top.jpeg](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/3.jpeg)

### stress-ng

stress-ng完全兼容stress, 并且在此基础上通过几百个参数，可以产生各种复杂的压力, 比如：

产生2个worker做圆周率算法压力：

```sh
stress-ng -c 2 --cpu-method pi
```

产生2个worker从迭代使用30多种不同的压力算法，包括pi, crc16, fft等等。

```sh
stress-ng -c 2 --cpu-method all
```

产生2个worker调用socket相关函数产生压力

```sh
stress-ng --sock 2
```

产生2个worker读取tsc产生压力

```sh
stress-ng --tsc 2
```

除了能够产生不同类型的压力，strss-ng还可以将压力指定到特定的cpu上，比如下面的命令将压力指定到cpu 0,2,3,6：

```sh
stress-ng --sock 4 --taskset 0,2-3,6
```