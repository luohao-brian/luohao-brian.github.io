---
layout: post
title: 使用tcpdump和wireshark分析tcp流
tags: [网络]
---

### Tcpdump抓包
```
tcpdump -w packets.pcap -n -i eth0 tcp port 60 and dst host 10.22.47.66
```

* `-i`: 指定网络接口
* `-n`: 不做域名解析，使用ip
* `-w`: 抓包存储为可供wireshark解析的pcap格式
* `tcp port 60 and dst host 10.22.47.66`: 条件表达式

### Tcpdump条件式

条件表达式语法可以参考: `man pcap-filter`，下面是一些常见的例子：

1. 只抓udp的包

```
tcpdump -i eth0 'udp'
```

2. 只想查看源机器和目的机器的包

```
tcpdump -i eth0 'dst 8.8.8.8'
```

3. 只想查看目标机器端口是53或80的包

```
tcpdump -i eth0 'dst port 53 or dst port 80'
```

4. 抓到那些通过eth0网卡的，且来源是roclinux.cn服务器或者目标是roclinux.cn服务器的网络包

```
tcpdump -i eth0 'host roclinux.cn'
```

5. 抓通过eth0网卡的，且roclinux.cn和baidu.com之间通讯的网络包，或者，roclinux.cn和qiyi.com之间通讯的网络包

```
tcpdump -i eth0 'host roclinux.cn and (baidu.com or qiyi.com)'
```


6. 获取使用ftp端口和ftp数据端口的网络包

```
tcpdump 'port ftp or ftp-data'
```

7. 获取roclinux.cn和baidu.com之间建立TCP三次握手中第一个网络包，即带有SYN标记位的网络包，另外，目的主机不能是 qiyi.com

```
tcpdump 'tcp[tcpflags] & tcp-syn != 0 and not dst host qiyi.com'
```


8. 打印IP包长超过576字节的网络包
```
tcpdump 'ip[2:2] > 576'
```

`proto [ expr : size]`，只要掌握了这个语法格式，相信大家就能看懂上面的三个稀奇古怪的表达式了。

proto就是protocol的缩写，表示这里要指定的是某种协议的名称，比如ip、tcp、ether。其实proto这个位置，总共可以指定的协议类型有15个之多，包括：

* ether – 链路层协议
* fddi – 链路层协议
* tr – 链路层协议
* wlan – 链路层协议
* ppp – 链路层协议
* slip – 链路层协议
* link – 链路层协议
* ip
* arp
* rarp
* tcp
* udp
* icmp
* ip6
* radio

`expr`用来指定数据报偏移量，表示从某个协议的数据报的第多少位开始提取内容，默认的起始位置是0；而`size`表示从偏移量的位置开始提取多少个字节，可以设置为1、2、4。
如果只设置了`expr`，而没有设置`size`，则默认提取1个字节。比如`ip[2:2]`，就表示提取出第3、4个字节；而`ip[0]`则表示提取ip协议头的第一个字节。
在我们提取了特定内容之后，我们就需要设置我们的过滤条件了，我们可用的“比较操作符”包括：>，<，>=，<=，=，!=，总共有6个。
好，掌握了上面内容之后，我可以很负责任的告诉你，你已经掌握了tcpdump过滤表达式的最重要语法了。我们先来小试牛刀，看一个例题：
```
ip[0] & 0xf != 5
```

IP协议的第0-4位，表示IP版本号，可以是IPv4（值为0100）或者IPv6（0110）；第5-8位表示首部长度，单位是“4字节”，如果首部长度为默认的20字节的话，此值应为5。

### 安装Mac版Wireshark

`Wireshark`支持多种OS，包括Windows, Mac和Linux，我在这里是使用的Mac版本。

```
$ brew search wireshark
==> Formulae
wireshark

==> Casks
wireshark  wireshark-chmodbpf

$ brew cask install wireshark
```

### wireshark分析指定的tcp流

在过滤条件中填写`tcp.flags.syn==1`，或者`tcp.flags.fin==1`, 找到tcp连接的首包或者尾包，这样的tcp流会相对比较完整；

![](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/wireshark-1.jpg)

在找到的包上点右键，选择Follow=>TCP Stream, 找到这条完整的tcp流：

![](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/wireshark-2.jpg)

### 参考
* [tcpdump表达式语法参考pcap-filter](https://linux.die.net/man/7/pcap-filter)

