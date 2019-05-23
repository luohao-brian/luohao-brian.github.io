---
layout: post
title: Libvirt虚拟机IP地址静态绑定
tags: [虚拟化]
---

### 背景

使用libvirt默认NAT网络创建虚拟机，发现即使Mac地址不一样的虚拟机，会分配到同样的IP地址，怀疑是dnsmasq服务的range功能有问题：

libvirt默认的default网络：
```
# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     no            yes
 
```

默认的default网络配置, dhcp range从192.168.122.2到192.168.122.254：
```
 # virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>1f700d00-fe92-4d31-963b-f6038d46e3a4</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:65:b0:e5'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
  </ip>
</network>
```

### 配置静态IP绑定

编辑默认网络， 根据虚拟机的mac地址和主机名，在<range>之后加入对应的绑定：

```
# virsh net-edit default

<network>
  <name>default</name>
  <uuid>1f700d00-fe92-4d31-963b-f6038d46e3a4</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:65:b0:e5'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
      <host mac='52:54:00:65:f3:de' name='ubuntu-1804-hluo' ip='192.168.122.76'/>
      <host mac='52:54:00:65:f5:df' name='nodejs' ip='192.168.122.77'/>
    </dhcp>
  </ip>
</network>
```

重启libvirt网络服务

```
# virsh net-destroy default
# virsh net-start default
```

重建虚拟机
```
# virsh destroy nodejs
# virsh create nodejs.xml
```

### 参考

* [KVM libvirt assign static guest IP addresses using DHCP on the virtual machine](https://www.cyberciti.biz/faq/linux-kvm-libvirt-dnsmasq-dhcp-static-ip-address-configuration-for-guest-os/)

