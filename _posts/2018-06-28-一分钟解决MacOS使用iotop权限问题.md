---
layout: post
title: 一分钟解决MacOS使用iotop权限问题
tags: [Mac]
---

### [](#背景)背景

iotop一般用来在列出系统中top N的正在使用磁盘io的进程，在Linux系统中，一般使用yum, apt命令安装后就可以使用。

Mac OS最新的Sierra系统也集成了iotop, 但是使用的时候会报告如下dtrace错误:

```
$ iotop
dtrace: system integrity protection is on, some features will not be available
dtrace: failed to initialize dtrace: DTrace requires additional privileges
```

### [](#解决办法)解决办法

重启系统，在启动的过程中按住Command+R，让系统进入Recovery模式；

在菜单中选择"打开终端", 执行下面的命令

```
csrutil disable
```

或者：

```
csrutil clear
csrutil enable --without dtrace
```

重启系统, 搞定。
