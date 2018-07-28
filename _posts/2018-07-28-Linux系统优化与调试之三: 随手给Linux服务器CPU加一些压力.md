---
layout: post
title: Linux系统优化与调试之三 - 随手给Linux服务器CPU加一些压力
categories: 系统优化
tags: [系统优化]
---

> 很多服务器上Linux系统是最小化安装，没有额外的压力工具，有时候只是想随手给CPU加一些背景压力，这几天参考了一些很有趣的脚本，非常巧妙的实现了这个，其中的思路值得学习。

------

### CPU满负载

```sh
for i in $(seq 0 16); do taskset -c $i yes > /dev/null & done
```

- `yes > /dev/null`: 本质就是一个while(1)循环
- `taskset -c $i`: 将任务pin在指定的cpu上执行
- `seq 0 16`: 16是逻辑CPU的数量

执行后用top观察，可以看到CPU已经被打满:

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
10217 root      20   0  107964    644    568 R  99.7  0.0   0:17.92 yes
10218 root      20   0  107964    644    568 R  99.7  0.0   0:17.92 yes
```

### 浮点性能测试

```sh
for i in $(seq 0 16); do taskset -c $i echo "scale=20000; 4*a(1)" | bc -l -q & done
```

- `echo "scale=20000; 4*a(1)" | bc -l`: 利用bc命令的数学函数库，该命令计算pi到第20000位
- `taskset -c $i`: 将任务pin在指定的cpu上执行
- `seq 0 16`: 16是逻辑CPU的数量

执行后用top观察，可以看到CPU也已经被打满:

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
10331 root      20   0   13536   2316   1772 R 100.0  0.0   0:14.92 bc
10333 root      20   0   13536   2284   1740 R 100.0  0.0   0:14.91 bc
```
