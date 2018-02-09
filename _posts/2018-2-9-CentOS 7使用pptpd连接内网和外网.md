---
layout: post
title: CentOS 7使用pptpd连接内网和外网
tags: [Linux]
---

客户有部分网络服务处于企业内网，由于各种限制，无法迁移到公有云服务器中，，比如处于内网中的基于usb设备的软件狗服务，无法在云服务器中安装，所以希望公有云服务器能访问这些内网服务。

这其实一个典型的vpn场景，目前主要的公有云提供商都会提供vpn服务，但是它需要提供一个公网地址作为网关，而有些大企业客户的内网拓扑非常复杂，这些企业的分支部门无法获取公网出口地址，或者由于采用某些运营商提供的拨号上网的方式，只能获得一个内网地址，这时候就可以通过自己搭建一个pptpd服务将某些内网地址映射到公有云服务器商，双方可以互相访问。

------

### 环境

1. CentOS 7.3
2. pptpd-1.4.0

### 过程

服务器端安装软件pptpd；
```
yum install -y pptpd
```

配置pptpd, vpn网络为10.1.1.0/24，公有云服务器的vpn网络地址为10.1.1.1，从内网拨号上来的主机地址范围从为10.1.1.10到10.1.1.100；

```
vim /etc/pptpd.conf
localip 10.1.1.1
remoteip 10.1.1.10,10.1.1.100
```

内网拨号上来的主机将分配8.8.8.8和8.8.4.4的dns服务地址；

```
vim /etc/ppp/options.pptpd 
ms-dns 8.8.8.8 
ms-dns 8.8.4.4
```

拨号所使用的简单认证信息，用户名test，密码123456；

```
vim /etc/ppp/chap-secrets


Secrets for authentication using CHAP
client server secret IP addressestest 
test pptpd 123456 *
```

重启pptpd服务

```
systemctl start pptpd
systemctl enable pptpd
```

WIndows 7客户端配置如下图所示，然后在弹出的认证中用户名输入test, 密码是123456；

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/40250000251d16487288.jpg)

### 验证
如果有客户端成功拨号到服务器，那么在公有云服务器上将生成一个新的隧道端点，我们可以在/var/log/messages中找到看到对应的地址，并使用这个地址访问这台内网服务器，同时用ifconfig也会观察到这台公有云服务器在vpn网络中被分配了指定的10.1.1.1地址。

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/402400002f3645ebc96f.jpg)