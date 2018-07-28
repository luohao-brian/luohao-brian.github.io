---
layout: post
title: CentOS-7使用pypy让python程序性能提升10倍
tags: [Python]
---

### EPEL REPO

```bash
[epel]
name=epel
baseurl=http://centos.ustc.edu.cn/epel/7/x86_64/
gpgcheck=0
enabled=1
```

### 安装pypy
```bash
yum install pypy pypy-devel
```

安装pypy-pip
```bash
pypy -m ensurepip
```

安装成功后，pip在 /usr/lib64/pypy-5.0.1/bin/，用pypy加速的python程序所有的依赖包，必须通过这个pip来安装。

### 测试

```python
def check(num):
     a = list(str(num))
     b = a[::-1]
     if a == b:
         return True
     return False


def main():
    all = xrange(1,10**7)
    for i in all:
        if check(i):
            if check(i**2):
                print i,i**2


if __name__ == '__main__':
    main()
```

```bash
[root@luohao-vm-static ~]# time python test.py
...
2001002 4004009004004

real	0m11.054s
user	0m11.037s
sys	0m0.007s
```


```bash
[root@luohao-vm-static ~]# time pypy test.py
...

real	0m1.855s
user	0m1.826s
sys	0m0.027s
```
