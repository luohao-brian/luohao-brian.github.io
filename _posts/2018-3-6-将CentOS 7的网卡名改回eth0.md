---
layout: post
title: 将CentOS 7的网卡名改回eth0
tags: [Linux]
---

### 背景

在服务器上安装CentOS 7，用ip或者ifconfig命令查看网卡信息，会发现网卡会被命名为enXXX之类，而不是传统的ethX, 那么该如何继续使用传统的ethX命名方式呢？

### 方法

首先在grub bootloader加上net.ifnames=0 biosdevname=0参数：

```bash
grubby --update-kernel=ALL --args="net.ifnames=0 biosdevname=0"
```

然后重命名网卡配置文件:
```bash
cd /etc/sysconfig/network-scripts && mv ifcfg-enXXX ifcfg-eth0
```

ifcfg-eth0文件中的DEVICE要改成eth0:

```bash
BOOTPROTO=dhcp
DEVICE=eth0
HWADDR=fa:16:3e:53:d2:45
ONBOOT=yes
TYPE=Ethernet
```

最后，别忘记需要重启生效。
