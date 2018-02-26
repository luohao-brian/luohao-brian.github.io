---
layout: post
title: 阿里云CentOS 7服务器配置L2TP+IPSec
tags: [Linux]
---

云谷计算写过一篇 CentOS 7使用pptpd连接内网和外网的文章，如果客户端是Mac OS或者iOS, 最新的版本都已经废弃掉pptp支持，改用L2TP+IPSec, 作为续篇，本文重点介绍如何在阿里云CentOS 7服务器上安装和配置L2TP+IPSec服务。

------


### 云服务器安全组配置
新申请的云服务器默认安全组对入方向仅仅允许ssh, http, https这几个最常用服务的tcp通讯，L2TP服务使用UDP 1701端口，并且在连接过程中可能还要使用到500, 4500等端口，为了简单，我在安全组中入方向上允许所有流量：

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/4032000467c36d6bb74c.jpg)

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/40330004553ab6b33b93.jpg)

### 安装软件

```
yum install strongswan xl2tpd -y
```

### 配置strongswan
先来配置strongSwan，在/etc/strongswan/ipsec.conf中：
```
# vim /etc/strongswan/ipsec.conf
config setup
conn %default
ikelifetime=60m
keylife=20m
rekeymargin=3m
keyingtries=1
conn l2tp

#IKE协议版本
keyexchange=ikev1

#服务器IP，自己修改为你自己的
left=12.34.56.78
leftsubnet=0.0.0.0/0
leftprotoport=17/1701
authby=secret
leftfirewall=no
right=%any
rightprotoport=17/%any
type=transport
auto=add
```

接下来配置PSK密匙，在/etc/strongswan/ipsec.secrets中：
```
# vim /etc/strongswan/ipsec.secrets
# ipsec.secrets - strongSwan IPsec secrets file
: PSK "你的PSK秘钥"
```

启动strongswan服务，并设置为开机自启动：
```
systemctl start strongswan && systemctl enable strongswan
```

### 配置xl2tpd
在/etc/xl2tpd/xl2tpd.conf中：
```
; vim /etc/xl2tpd/xl2tpd.conf
[lns default]
;客户端ip分配地址范围
ip range = 172.16.37.2-172.16.37.254
;L2TP本地ip，也是客户端的网关
local ip = 172.16.37.1
assign ip = yes
require chap = yes
refuse pap = yes
require authentication = yes
name = xl2tpd
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

接下来，编辑/etc/ppp/options.xl2tpd：
```
# vim /etc/ppp/options.xl2tpd
ipcp-accept-local
ipcp-accept-remote
ms-dns 8.8.8.8
ms-dns 8.8.4.4
noccp
auth
#crtscts
idle 1800
mtu 1460
mru 1460
nodefaultroute
debug
#lock
proxyarp
connect-delay 5000
```

最后一步，给L2TP添加用户，在/etc/ppp/chap-secrets中:
```
# client server secret IP addresses
test xl2tpd 123456 *
```

启动xl2tpd, 并设置为开机自启动:
```
systemctl start xl2tpd && systemctl enable xl2tpd
```

### 防火墙配置
使用iptables命令添加如下规则，放开xl2tpd需要的端口，并配置NAT:
```
iptables -I INPUT -p udp --dport 500 -j ACCEPT
iptables -I INPUT -p udp --dport 4500 -j ACCEPT
iptables -I INPUT -p udp -m policy --dir in --pol ipsec -m udp --dport 1701 -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.37.0/24 -o eth0 -j MASQUERADE
iptables -t filter -I FORWARD -s 172.16.37.0/24 -d 172.16.37.0/24 -j DROP
iptables -I FORWARD -s 172.16.37.0/24 -j ACCEPT
iptables -I FORWARD -d 172.16.37.0/24 -j ACCEPT
```

如果需要持久化防火墙配置，可以执行命令：
```
iptables-save > /etc/sysconfig/iptables
systemctl iptables restart
```

### Mac OS配置
首先新建一个ipsec+l2tp的vpn连接:

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/403800022f63f1b9fe5d.jpg)

在服务器地址中输入阿里云服务器的公网地址，用户名和用户坚定密码输入上面chap-secrets中对应的配置，共享秘钥输入ipsec.secrets对应的配置，在高级菜单中勾选通过vpn连接发送所有流量。

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/40370002b87c504f5b83.jpg)