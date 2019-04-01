---
layout: post
title: Linux KVM虚拟机限制core dump大小
tags: [虚拟化]
---

### 背景

Linux KVM虚拟机由qemu加载和引导，虚拟机的运行内存也都映射在qemu进程空间里面，导致qemu的rss内存一般都很大，比如：

```
top - 12:46:02 up 6 days, 13:17,  5 users,  load average: 1.14, 1.22, 1.72
Tasks: 532 total,   1 running, 279 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.7 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 19667006+total, 98877040 free,  5496464 used, 92296560 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 18959481+avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
3357981 root      20   0 17.087g 1.742g  22244 S   1.3  0.9 106:12.59 qemu-system-x86
4157696 root      20   0 9770.2m 1.474g  22676 S   2.0  0.8   6:38.13 qemu-system-x86
```

如果qemu进程由于任何bug发生core dump, 这就导致产生一个非常巨大的core文件。

### 解决办法

编辑`/etc/libvirt/qemu.conf`, 下面的配置将会将qemu进程的core文件限制到2G, 并且不包括guest内存。

```
# Size is a positive integer specifying bytes or the
# string "unlimited"
#
#max_core = "unlimited"
max_core = 2147483648

# Determine if guest RAM is included in QEMU core dumps. By
# default guest RAM will be excluded if a new enough QEMU is
# This setting will be ignored if the guest XML has set the
# dumpcore attribute on the <memory> element.
#
#dump_guest_core = 1
dump_guest_core = 0

```
