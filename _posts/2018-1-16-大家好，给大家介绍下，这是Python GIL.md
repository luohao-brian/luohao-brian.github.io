---
layout: post
title: 大家好，给大家介绍下，这是Python GIL
categories: pythonGIL
tags: [Python]
---
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/8.jpg)

### Python是一门很古老的语言
相比与C, Fortran这些至今很活跃的编程语言来说，Python的历史算不上久远，但是在Python最初的那些日子里，处理器都是单核心，而现在市场上即使最便宜最低端的处理器也是多核心多线程，正是这样的基础硬件架构变迁，为GIL的问题埋下了伏笔。
- Python 1.0 - 1994.1
- Python 2.0 - 2000/10/16
- Python 2.4 – 2004/11/30
- ...

### 什么是GIL(Global Interpreter Lock)
Python作为一门编程语言，可没有定义GIL, 这玩意其实是Python解析器CPython用来做多线程控制和调度的全局锁，Python还有一些别的解析器，比如JPython, Pypy等，其中JPython就没有GIL。但是CPython现在已经成为Python的实际标准，各种Linux和其它OS默认集成的Python, 以及从官方网站上下载的Python，还有各种相关的工具如pip等都在使用CPython，所以GIL问题自然也就成为了Python语言的 问题了。

GIL的问题其实是由于近十几年来应用程序和操作系统逐步从多任务单核心演进到多任务多核心导致的 , 在一个古老的单核CPU上调度多个线程任务，大家相互共享一个全局锁，谁在CPU执行，谁就占有这把锁，直到因为IO或者Timer Tick到期让出CPU，没有在执行的线程就安静的等待着这把锁（除了等待之外，他们应该也无事可做）：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/9.jpg)

很明显，在一个现代多核心的处理器上，上面的模型就有很大优化空间了，原来只能等待的线程任务，现在可以在其它空闲的核心上调度并发执行。由于古老GIL机制，如果线程2需要在CPU 2上执行，它需要先等待在CPU 1上执行的线程1释放GIL，要命的是，在Python 2.x, 线程A不会动态的调整自身的优先级，所以很大概率下次被选中执行的还是A，在很多个这样的选举周期内，线程2只能安静的看着线程1拿着GIL在CPU 1上欢快的执行。
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/10.jpg)

在稍微极端一点的情况下，比如线程1使用了while 1在CPU 1 上执行，那就真是“一核有难，八核围观”了。
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/11.jpg)

### GIL测试
t1: 2个线程依次执行count down 100,000,000
```
#! /usr/bin/python
from threading import Thread
import time
def my_counter():
	i = 0
	for _ in range(100000000):
		i = i + 1
	return True
def main():
	thread_array = {}
	start_time = time.time()
	for tid in range(2):
		t = Thread(target=my_counter)
		t.start()
		t.join()
	end_time = time.time()
	print("Total time: {}".format(end_time - start_time))
if __name__ == '__main__':
	main()
```

t2: 2个线程并发执行count down 100,000,000
```
#! /usr/bin/python
from threading import Thread
import time
def my_counter():
	i = 0
	for _ in range(100000000):
		i = i + 1
	return True
def main():
	thread_array = {}
	start_time = time.time()
	for tid in range(2):
		t = Thread(target=my_counter)
		t.start()
		thread_array[tid] = t
	for i in range(2):
		thread_array[i].join()
	end_time = time.time()
	print("Total time: {}".format(end_time - start_time))
if __name__ == '__main__':
	main()
```

在我的两核心Intel i5 Macbook上，两个程序用python 2.7解析执行，理论上，在一个双核处理器上，t2应该比t1少用一半的时间，可是实际上，t2居然比t1多用了一倍以上的时间，这是一种多么重的领悟。。。
```
LuodeMacBook-Pro:Documents hluo$ python t1.py

Total time: 11.8666779995

LuodeMacBook-Pro:Documents hluo$ python t2.py

Total time: 23.9497878551
```

### 如何避免受到GIL的影响
`用多进程(multi-process)替代多线程(multi-thread):`我在最近两年的实际项目中多次使用了多线程，踩了很多坑，特别是在程序压力增大的时候，很多任务迟迟不能被调度执行导致超时失败，我因此才开始比较深入的关注GIL等问题，现在也能理解为什么如OpenStack这样的大型Python项目都是使用的多进程，或者是协程的方式。
`使用Python 3.2+:`Python 3.2对GIL做了比较深入的优化，上面的程序如果用python 3.6解析执行，结果会好很多，但是距离理论预期还是有较大距离。
```
LuodeMacBook-Pro:Documents hluo$ python3 t1.py

Total time: 13.256227254867554

LuodeMacBook-Pro:Documents hluo$ python3 t2.py

Total time: 15.18819260597229
```


