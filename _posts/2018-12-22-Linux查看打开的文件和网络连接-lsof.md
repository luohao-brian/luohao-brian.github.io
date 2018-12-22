---
layout: post
title: Linux使用lsof查看打开的文件和网络连接
tags: [Linux]
---

> lsof是List Open Files的缩写。顾名思义，它用来查看系统中进程打开了哪些文件；因为Linux几乎所有的设备都可以看成是文件，所以lsof经常也可以用来查看管道，sockets的使用状态。

### 查看当前所有的活跃连接

```
# lsof -i

COMMAND    PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind   1203     rpc    6u  IPv4  11326      0t0  UDP *:sunrpc
rpcbind   1203     rpc    7u  IPv4  11330      0t0  UDP *:954
rpcbind   1203     rpc   11u  IPv6  11336      0t0  TCP *:sunrpc (LISTEN)
avahi-dae 1241   avahi   13u  IPv4  11579      0t0  UDP *:mdns
avahi-dae 1241   avahi   14u  IPv4  11580      0t0  UDP *:58600
rpc.statd 1277 rpcuser   11u  IPv6  11862      0t0  TCP *:56428 (LISTEN)
cupsd     1346    root    6u  IPv6  12112      0t0  TCP localhost:ipp (LISTEN)
cupsd     1346    root    7u  IPv4  12113      0t0  TCP localhost:ipp (LISTEN)
sshd      1471    root    3u  IPv4  12683      0t0  TCP *:ssh (LISTEN)
master    1551    root   12u  IPv4  12896      0t0  TCP localhost:smtp (LISTEN)
master    1551    root   13u  IPv6  12898      0t0  TCP localhost:smtp (LISTEN)
sshd      1834    root    3r  IPv4  15101      0t0  TCP 192.168.0.2:ssh->192.168.0.1:conclave-cpp (ESTABLISHED)
httpd     1918    root    5u  IPv6  15991      0t0  TCP *:http (LISTEN)
httpd     1918    root    7u  IPv6  15995      0t0  TCP *:https (LISTEN)
clock-app 2362   narad   21u  IPv4  22591      0t0  TCP 192.168.0.2:45284->www.gov.com:http (CLOSE_WAIT)
chrome    2377   narad   61u  IPv4  25862      0t0  TCP 192.168.0.2:33358->maa03s04-in-f3.1e100.net:http (ESTABLISHED)
chrome    2377   narad   80u  IPv4  25866      0t0  TCP 192.168.0.2:36405->bom03s01-in-f15.1e100.net:http (ESTABLISHED)
```

### 查看指定进程打开的文件

```
# lsof -p `pidof systemd`

COMMAND PID USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd   1 root  cwd       DIR                8,2     4096        128 /
systemd   1 root  rtd       DIR                8,2     4096        128 /
systemd   1 root  txt       REG                8,2  1612152  100958350 /usr/lib/systemd/systemd
systemd   1 root  mem       REG                8,2    20112   67195580 /usr/lib64/libuuid.so.1.3.0
systemd   1 root  mem       REG                8,2   261488   67202929 /usr/lib64/libblkid.so.1.1.0
systemd   1 root  mem       REG                8,2    90664   67202184 /usr/lib64/libz.so.1.2.7
systemd   1 root  mem       REG                8,2   157424   67195586 /usr/lib64/liblzma.so.5.2.2
systemd   1 root  mem       REG                8,2    23968   67202345 /usr/lib64/libcap-ng.so.0.0.0
systemd   1 root  mem       REG                8,2    19896   67202240 /usr/lib64/libattr.so.1.1.0
systemd   1 root  mem       REG                8,2    19776   67195455 /usr/lib64/libdl-2.17.so
systemd   1 root  mem       REG                8,2   402384   67195605 /usr/lib64/libpcre.so.1.2.0
systemd   1 root  mem       REG                8,2  2173512   67195449 /usr/lib64/libc-2.17.so
```

### 查看指定端口被哪个进程占用

```
# lsof -i :22

COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1017 root    3u  IPv4  19044      0t0  TCP *:ssh (LISTEN)
sshd    1017 root    4u  IPv6  19053      0t0  TCP *:ssh (LISTEN)
sshd    1542 root    3u  IPv4  28685      0t0  TCP localhost.localdomain:ssh->192.168.1.102:61167 (ESTABLISHED)
```

```
lsof -i TCP:1-1024

COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind 1203     rpc   11u  IPv6  11336      0t0  TCP *:sunrpc (LISTEN)
cupsd   1346    root    7u  IPv4  12113      0t0  TCP localhost:ipp (LISTEN)
sshd    1471    root    4u  IPv6  12685      0t0  TCP *:ssh (LISTEN)
master  1551    root   13u  IPv6  12898      0t0  TCP localhost:smtp (LISTEN)
sshd    1834    root    3r  IPv4  15101      0t0  TCP 192.168.0.2:ssh->192.168.0.1:conclave-cpp (ESTABLISHED)
sshd    1838 tecmint    3u  IPv4  15101      0t0  TCP 192.168.0.2:ssh->192.168.0.1:conclave-cpp (ESTABLISHED)
sshd    1871    root    3r  IPv4  15842      0t0  TCP 192.168.0.2:ssh->192.168.0.1:groove (ESTABLISHED)
httpd   1918    root    5u  IPv6  15991      0t0  TCP *:http (LISTEN)
httpd   1918    root    7u  IPv6  15995      0t0  TCP *:https (LISTEN)
```

### 过滤指定的连接状态

```
# lsof -i :22 -s TCP:LISTEN
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1017 root    3u  IPv4  19044      0t0  TCP *:ssh (LISTEN)
sshd    1017 root    4u  IPv6  19053      0t0  TCP *:ssh (LISTEN)
```

支持的TCP状态包括: CLOSED, IDLE, BOUND, LISTEN, ESTABLISHED, SYN_SENT, SYN_RCDV, ESTABLISHED, CLOSE_WAIT, FIN_WAIT1, CLOSING, LAST_ACK, FIN_WAIT_2, and TIME_WAIT。  

支持的UDP状态包括: Unbound和Idle。

### 如何防止第一列命令行被截断

如果命令的文件名超过9个字符，lsof会默认截断这个命令：

```
# lsof |grep kworker

kworker/0    5             root  cwd       DIR                8,2      4096        128 /
kworker/0    5             root  rtd       DIR                8,2      4096        128 /
kworker/0    5             root  txt   unknown                                         /proc/5/exe
kworker/u    6             root  cwd       DIR                8,2      4096        128 /
kworker/u    6             root  rtd       DIR                8,2      4096        128 /
```
可以使用+c 0防止命令被截断:

```
# lsof +c 0|grep kworker

kworker/0:0H       5             root  cwd       DIR                8,2      4096        128 /
kworker/0:0H       5             root  rtd       DIR                8,2      4096        128 /
kworker/0:0H       5             root  txt   unknown                                         /proc/5/exe
kworker/u256:0     6             root  cwd       DIR                8,2      4096        128 /
kworker/u256:0     6             root  rtd       DIR                8,2      4096        128 /
```
