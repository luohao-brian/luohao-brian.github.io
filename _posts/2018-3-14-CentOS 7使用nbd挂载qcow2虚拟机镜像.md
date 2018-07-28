---
layout: post
title: CentOS 7使用nbd挂载qcow2虚拟机镜像
tags: [虚拟化]
---

### 背景
现在QCOW2格式的虚拟机镜像越来越多，各Linux发行版也都默认发布了QCOW2等格式的云镜像，除了使用虚拟机来挂载启动这些镜像，能不能在主机上直接挂载这些镜像操作镜像中的内容呢？

Linux支持Network Block Drive(NBD)，就可以支持直接挂载QCOW2镜像，不过可惜的是CentOS 7并没有编译集成nbd.ko, 导致qemu-nbd不能使用，所以我们需要自己编译

### 解决办法

我已经把编译好的最新版本nbd.ko放到了云谷计算的服务器上：

`https://ygjs.tech/static/pkgs/drivers/`

如果要挂载一个QCOW2镜像，参考如下命令:

```bash
# 下载nbd.ko
cd /lib/modules/3.10.0-693.21.1.el7.x86_64/extra/ 
wget https://ygjs.tech/static/pkgs/drivers/linux-3.10.0-693.21.1/nbd.ko

# 加载nbd.ko
depmod -a && modprobe nbd max_part=8

# 挂载qcow2镜像
qemu-nbd --connect=/dev/nbd0 image.qcow2

# 将镜像中的第一个分区挂载到本地的/mnt目录
mount /dev/nbd0p1 /mnt
```

### 编译过程
```bash
# 下载对应内核的srpm
wget http://vault.centos.org/7.4.1708/updates/Source/SPackages/kernel-3.10.0-693.21.1.el7.src.rpm

# 安装内核源码
rpm -Uvh kernel-3.10.0-693.21.1.el7.src.rpm

# 安装rpm打包工具rpm-build
yum install -y rpm-build

# 准备内核源代码, rpmbuild命令会提示依赖的包，用yum
# 逐个安装上
cd /root/rpmbuild/SPECS && rpmbuild -bp kernel.spec

# 内核目录
cd ~/rpmbuild/BUILD/kernel-3.10.0-693.21.1.el7/linux-3.10.0-693.21.1.el7.x86_64/

# 编译选项
# Device Driver -> Block devices -> 在“Network block device support”选择M
make menuconfig

# 编译，这个过程可能需要一个小时...
make prepare && make modules_prepare && make
make M=drivers/block -j8

# 检查nbd.ko
modinfo drivers/block/nbd.ko
```
值得注意的是，CentOS 7内核源码中的nbd.c有个bug，会导致编译失败，报告如下错误:
```
/root/rpmbuild/BUILD/kernel-3.10.0-514.26.2.el7/linux-3.10.0-514.26.2.el7.x86_64/drivers/block/nbd.c:619:19: 
error: ‘REQ_TYPE_SPECIAL’ undeclared (first use in this function) 
    sreq.cmd_type = REQ_TYPE_SPECIAL; 
```

这是因为REQ_TYPE_SPECIAL已经被REQ_TYPE_DRV_PRIV替代了，所以把nbd.c 619行做如下修改：

``` c
//sreq.cmd_type = REQ_TYPE_SPECIAL;
sreq.cmd_type = REQ_TYPE_DRV_PRIV;
```

