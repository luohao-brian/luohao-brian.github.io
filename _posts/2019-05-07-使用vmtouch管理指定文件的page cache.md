---
layout: post
title: 使用vmtouch管理指定文件的page cache
tags: [Linux]
---

### 背景

Page Cache对内存的使用是贪婪模式，即尽量把空闲内存都用做文件page cache。如果系统需要比较频繁的分配大内存，会导致陷入slow path回收内存，带来内存分配延时甚至是超时失败。

一个比较极端的案例是写日志，在flush之后，page cache并不会释放掉，但是这样的page cache对系统没有任何正向帮助，我们应该尝试把这些page cache尽快释放掉。

一般来说，我们可以通过drop cache把所有的cache释放掉，比如:

```
# free
             total       used       free     shared    buffers     cached
Mem:       7917304    7603628     313676     288408     590360    1952560
-/+ buffers/cache:    5060708    2856596
Swap:            0          0          0

# echo 3 > /proc/sys/vm/drop_caches

# free
             total       used       free     shared    buffers     cached
Mem:       7917304    4291836    3625468     291852       1760     331644
-/+ buffers/cache:    3958432    3958872
Swap:            0          0          0
```

如果我们知道某个日志文件的page cache使用不合理, 其实还可以使用[vmtouch](https://hoytech.com/vmtouch/)来针对指定的文件管理page cache。

### 快速安装

```
$ git clone https://github.com/hoytech/vmtouch.git
$ cd vmtouch
$ make
$ sudo make install
```

### 管理page cache

查看指定文件占用的page cache:

```
# vmtouch result.log
           Files: 1
     Directories: 0
  Resident Pages: 0/144532  0/564M  0%
         Elapsed: 0.002367 seconds
```

模拟一些page cache占用：

```
# cat result.log > /dev/null
```

再查看result.log的page cache使用:
```
# vmtouch result.log
           Files: 1
     Directories: 0
  Resident Pages: 27776/144532  108M/564M  19.2%
         Elapsed: 0.004451 seconds
```

释放result.log占用的page cache:

```
# vmtouch result.log
           Files: 1
     Directories: 0
  Resident Pages: 27776/144532  108M/564M  19.2%
         Elapsed: 0.004451 seconds
root@n8-119-148^:~# vmtouch -e result.log
           Files: 1
     Directories: 0
   Evicted Pages: 144532 (564M)
         Elapsed: 0.008665 seconds
# vmtouch result.log
           Files: 1
     Directories: 0
  Resident Pages: 0/144532  0/564M  0%
         Elapsed: 0.002522 seconds
```

### 参考

* [vmtouch的github主页](https://github.com/hoytech/vmtouch)
