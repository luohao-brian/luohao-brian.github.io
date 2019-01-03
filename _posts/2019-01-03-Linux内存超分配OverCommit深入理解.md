---
layout: post
title: Linux内存超分配Overcommit深入理解
tags: [Linux]
---

### 背景
Memory Overcommit的意思是操作系统承诺给进程的内存大小超过了实际可用的内存。一个保守的操作系统不会允许memory overcommit，有多少就分配多少，再申请就没有了，这其实有些浪费内存，因为进程实际使用到的内存往往比申请的内存要少，比如某个进程malloc()了200MB内存，但实际上只用到了100MB，按照UNIX/Linux的算法，物理内存页的分配发生在使用的瞬间，而不是在申请的瞬间，也就是说未用到的100MB内存根本就没有分配，这100MB内存就闲置了。

下面这个概念很重要，是理解memory overcommit的关键：

> *commit*(或overcommit)针对的是内存申请，内存申请不等于内存分配，内存只在实际用到的时候才分配。

Linux是允许memory overcommit的，只要你来申请内存我就给你，寄希望于进程实际上用不到那么多内存，但万一用到那么多了呢？那就会发生类似“银行挤兑”的危机，现金(内存)不足了。Linux设计了一个OOM killer机制(OOM = out-of-memory)来处理这种危机：挑选一个进程出来杀死，以腾出部分内存，如果还不够就继续杀…也可通过设置内核参数 `vm.panic_on_oom` 使得发生OOM时自动重启系统。

### 内存超分内核参数

内核有几个跟内存超分配相关的参数，可以用sysctl查看和修改：

```
# sysctl -a  2>/dev/null|grep overcommit
vm.nr_overcommit_hugepages = 0
vm.overcommit_kbytes = 0
vm.overcommit_memory = 0
vm.overcommit_ratio = 50
```

或者也可以通过procfs查看和修改：
```
# ls -l /proc/sys/vm/overcommit*
-rw-r--r-- 1 root root 0 Jan  3 17:29 /proc/sys/vm/overcommit_kbytes
-rw-r--r-- 1 root root 0 Dec 24 10:44 /proc/sys/vm/overcommit_memory
-rw-r--r-- 1 root root 0 Jan  3 17:29 /proc/sys/vm/overcommit_ratio
```

### vm.overcommit_memory

* 0 – Heuristic overcommit handling. 这是缺省值，它允许overcommit，但过于明目张胆的overcommit会被拒绝，比如malloc一次性申请的内存大小就超过了系统总内存。Heuristic的意思是“试探式的”，内核利用某种算法（对该算法的详细解释请看文末）猜测你的内存申请是否合理，它认为不合理就会拒绝overcommit。
* 1 – Always overcommit. 允许overcommit，对内存申请来者不拒。
* 2 – Don’t overcommit. 禁止overcommit。

### Memory Commit

当设置vm.overcommit_memory=2就关闭了overcommit ，那么怎样才算是overcommit呢？kernel设有一个阈值，申请的内存总数超过这个阈值就算overcommit，在/proc/meminfo中可以看到这个阈值的大小. `CommitLimit` 就是overcommit的阈值，申请的内存总数超过CommitLimit的话就算是overcommit。


```
# grep -i commit /proc/meminfo
CommitLimit:     5967744 kB
Committed_AS:    5363236 kB
```

这个阈值是如何计算出来的呢？它既不是物理内存的大小，也不是free memory的大小，它是通过内核参数`vm.overcommit_ratio`或`vm.overcommit_kbytes`间接设置的，公式如下：

```
CommitLimit = (Physical RAM * vm.overcommit_ratio / 100) + Swap
```

`vm.overcommit_ratio` 是内核参数，缺省值是50，表示物理内存的50%。

/proc/meminfo中的`Committed_AS` 表示所有进程已经申请的内存总大小，（注意是已经申请的，不是已经分配的），如果 `Committed_AS` 超过 `CommitLimit` 就表示发生了 overcommit，超出越多表示 overcommit 越严重。Committed_AS 的含义换一种说法就是，如果要绝对保证不发生OOM (out of memory) 需要多少物理内存。

### Atop工具

atop是一个我常用的系统状态查看工具，其中的vmcom(Virtual Memory Commit)即反映了`Committed_AS`, 而vmlim(Virtual Memory Limit)反映了`CommitLimit`, 当vmcom大于vmlim的时候，atop会高亮vmcom作为告警，表示这个时候系统会有oom的可能。

```
PRC | sys    0.53s | user   0.65s | #proc    621 | #trun      2 |  #tslpi   807 | #tslpu     0 | #zombie    0 | clones  1242 | #exit   1230 |
CPU | sys       5% | user      9% | irq       0% | idle   4787% |  wait      0% | steal     0% | guest     0% | curf 2.70GHz | curscal   ?% |
CPL | avg1    0.82 | avg5    0.89 | avg15   0.97 |              |  csw    69558 |              | intr   59138 |              | numcpu    48 |
MEM | tot   187.6G | free  161.1G | cache  18.1G | buff  534.0M |  slab    1.0G | shmem 193.6M | vmbal   0.0M | hptot   0.0M | hpuse   0.0M |
SWP | tot     0.0M | free    0.0M |              |              |               |              |              | vmcom  25.2G | vmlim  93.8G |
```

### 验证

Demo程序

```c
#include <stdio.h>

#include <string.h>

#include <stdlib.h>


int main (void) {
    int n = 0;
    char *p;

    while (1) {
        if ((p = malloc(1<<20)) == NULL) {
                printf("malloc failure after %d MiB\n", n);
                return 0;
        }
        memset (p, 0, (1<<20));
        printf ("got %d MiB\n", ++n);
    }
}
```

4 GB RAM, 4 GB Swap, overcommit_memory = 2:

Overcommit Ratio|MemFree (kB)|CommitLimit (kB)|Breaks at (MB)|Expected break (MB)
---|---|---|---|---
10|3803668|4595144|4200|4488
25|3802056|5199488|4793|5078
50|3801852|6206724|5771|6062
75|3802732|7213960|6748|7045
90|3802620|7818300|7340|7636
100|3802888|8221196|7729|8029

4 GB RAM, no Swap, overcommit_memory = 2:

Overcommit Ratio|MemFree (kB)|CommitLimit (kB)|Breaks at (MB)|Expected break (MB)
---|---|---|---|---
10|3803964|402892|243|394
25|3802532|1007236|803|984
50|3799844|2014472|1756|1968
75|3803580|3021708|2708|2951
90|3805424|3626048|3276|3542
100|3804236|4028944|3653|3935

4 GB RAM, 4 GB Swap, overcommit_memory = 1:

Overcommit Ratio|MemFree (kB)|CommitLimit (kB)|Breaks at (MB)
---|---|---|---
10|3803952|4595144|7794
25|3804852|5199488|7804
50|3804564|6206724|7800
75|3805032|7213960|7800
90|3802900|7818300|7803
100|3801392|8221196|7748

### 参考
* [理解LINUX的MEMORY OVERCOMMIT](http://linuxperf.com/?p=102)
* [Virtual memory settings in Linux - The Problem with Overcommit](http://engineering.pivotal.io/post/virtual_memory_settings_in_linux_-_the_problem_with_overcommit/)
