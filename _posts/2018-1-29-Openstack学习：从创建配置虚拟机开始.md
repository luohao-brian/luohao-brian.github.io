---
layout: post
title: Openstack学习：从创建配置虚拟机开始
tags: [OpenStack]
---

对于初学者而言，openstack的架构和内容实在太多太复杂了，我们不妨从创建一个最基本的虚拟机入手，逐步了解和学习其中涉及的各种技术和知识。


# 绕不开的第一课

什么是Qemu？kvm？libvirt？

## Qemu
既然是虚拟机，那总得有虚拟出来的各种硬件吧，Qemu就是提供这一功能的硬件模拟器，让guestOS以为自己在和真的硬件打交道，而其实，这中间隔着一个Qemu来当翻译。也正是因为存在翻译的过程，所以性能上会打折扣，于是我们需要：

## kvm
kvm属于linux内核的模块，可以理解为集成至内核中的Hypervisor，采用intel VT/AMD-V等技术，提供CPU和内存的虚拟化能力，这样guestOS的CPU指令可以不用再经过翻译，性能大增，但它还需要network和周边I/O的支持，所以两者联手：kvm负责cpu和内存，qemu负责其他的，一个内核空间，一个用户空间从而形成qemu-kvm这样一个完整的、更优的虚拟化技术。

## Libvirt
Libvirt是一种常用的虚拟机管理工具，可管理包括kvm在内的众多虚拟化技术，包括一个API库，守护进程libvirtd，命令行工具virsh。

通过对基本概念的了解，我们其实已经可以想到这三者的关系：openstack通过libvirt来控制qemu-kvm进行虚拟机的相关操作，正如下图所示：
![pic_1](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-01-29/2208539948.jpg)

也就是说我们可以通过libvirt或者qemu来操作虚拟机，下面就开始试试吧。

------

# 创建虚拟机
## 1.准备工作
物理主机为centos 7.4，检查CPU是否支持虚拟化

```sh
cat /proc/cpuinfo | egrep '(vmx|svm)' | wc -l
```

返回大于0表示支持
其中，vmx是intel处理器虚拟机技术标志，svm是AMD的
### 2.安装所需工具
首先配置好yum源，然后根据具体需要安装

```bash 
yum install kvm libvirt python-virtinst qemu-kvm virt-viewer bridge-utils virt-install 
```

验证libvirtd是否正常启动

```bash
ps -ef | grep libvirtd
```

验证kvm是否加载成功

```bash
lsmod | grep kvm
```

验证工具安装是否成功

```bash
virsh list --all
```

## 3.创建虚拟机
介绍三种常用方式
### A. qemu方式
#### 创建磁盘镜像
虚拟机磁盘是用主机上的文件来模拟，支持多种镜像格式，如raw、qcow2、vhd等，通常利用qemu-img工具来创建

```bash
qemu-img create -f qcow2 /home/tmp/qemu/centos_vm.qcow2 10G
```

即在指定路径下创建了一个10G大小qcow2格式的镜像
#### 准备操作系统镜像
首次启动虚拟机需要安装操作系统，根据个人需要下载一个iso镜像即可
#### 启动虚拟机

```bash
qemu-system-x86_64 -enable-kvm -vnc:0 -smp 2 -m 2048 -cdrom ./CentOS-7-x86_64.iso -hda ./centos_vm.qcow2 -L /usr/share/qemu -usb -usbdevice tablet
```

参数说明：
-enable-kvm：采用kvm加速，不加的话qemu自身也可搞定，如开篇介绍过的
-vnc:0 :使用vnc连接虚拟机的界面，qemu内置vnc server端，0为监听端口号
-smp 2：给虚拟机配置2个vcpu
-m2048：给虚拟机配置2G内存
-cdrom：给虚拟机配置光驱，挂载之前准备好的系统镜像
-hda：给虚拟机配置磁盘，用我们之前创建好的磁盘镜像
-usb -usbdevice tablet：给虚拟机配置usb设备，这样vnc登录时鼠标好使
#### 安装guestos
使用vncviewer工具，输入监听的主机IP和vnc端口号，打开虚拟机操作系统安装界面，按照正常流程安装即可，装好后reboot，这样一个最基本的虚拟机就创建ok了。
我们接下来介绍下其他的创建方法
### B. virt-install方式
前两步和qemu方式一样，只是启动虚拟机的时候我们采用virt-install工具

```sh
virt-install --name=vm1 --ram=2048 --vcpus=2 --cdrom=./CentOS-7-x86_64.iso --disk path=./centos_vm.qcow2,device=disk,format=qcow2,bus=virtio,size=10 --network network=default --accelerate --graphics vnc,listen=186.100.88.33,port=5900 --force --autostart
```
各参数的含义和qemu方式一样，我们同样用vnc登录虚拟机后正常安装即可
### C. virtsh方式
前两步依旧相同，接着我们采用xml配置文件加virsh命令来创建
#### 编写xml配置文件

```sh
vi vm1.xml
```

具体内容可参考：

```xml
<domain type='kvm'>  <!--如果是Xen，则type=‘xen’-->
  <name>vm1</name> <!--虚拟机名称，同一物理机唯一-->
  <uuid>fd3535db-2558-43e9-b067-314f48211343</uuid>  <!--同一物理机唯一，可用uuidgen生成-->
  <memory>524288</memory>
  <currentMemory>524288</currentMemory>  <!--memory这两个值最好设成一样-->
  <vcpu>2</vcpu>            <!--虚拟机可使用的cpu个数-->
  <os>
   <type arch='x86_64' machine='pc-i440fx-vivid'>hvm</type> <!--arch指出系统架构类型，machine 则是机器类型，查看机器类型：qemu-system-x86_64 -M ?-->
   <boot dev='hd'/>  <!--启动介质，第一次需要装系统可以选择cdrom光盘启动-->
   <bootmenu enable='yes'/>  <!--表示启动按F12进入启动菜单-->
  </os>
  <features>
   <acpi/>  <!--Advanced Configuration and Power Interface,高级配置与电源接口-->
   <apic/>  <!--Advanced Programmable Interrupt Controller,高级可编程中断控制器-->
   <pae/>   <!--Physical Address Extension,物理地址扩展-->
  </features>
  <clock offset='localtime'/>  <!--虚拟机时钟设置，这里表示本地本机时间-->
  <on_poweroff>destroy</on_poweroff>  <!--突发事件动作-->
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>   <!--设备配置-->
   <emulator>/usr/libexec/qemu-kvm</emulator> 
   <disk type='file' device='disk'> <!--硬盘-->
      <driver name='qemu' type='qcow2'/>
      <source file='/home/tmp/qemu/centos_vm.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/> <!--域、总线、槽、功能号，slot值同一虚拟机上唯一-->
   </disk>
   <disk type='file' device='cdrom'><!--光盘-->
      <driver name='qemu' type='raw'/>
      <source file='/home/tmp/qemu/CentOS-7-x86_64.iso.iso'/>
      <target dev='hda' bus='ide'/>
      <readonly/>
   </disk>
   <interface type='network'>   <!--基于虚拟局域网的网络-->
     <mac address='52:54:4a:e1:1c:84'/>  <!--可用命令生成，见下面的补充-->
     <source network='default'/> <!--默认-->
     <target dev='vnet1'/>  <!--同一虚拟局域网的值相同-->
     <alias name='net1'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>  <!--注意slot值-->
   </interface>
  <graphics type='vnc' port='5900' autoport='yes' listen='186.100.88.33' keymap='en-us'/>  <!--配置vnc，windows下可以使用vncviewer登录，获取vnc端口号：virsh vncdisplay vm0-->
   <listen type='address' address='186.100.88.33'/>
  </graphics>
  </devices>
</domain>
```
#### 导入配置文件

```bash
virsh define vm1.xml
```

#### 启动虚拟机

```bash
virsh start vm1
```

如同前两种方法可通过vnc登录虚拟机界面
###配置网络
创建好虚拟机后，我们自然希望将它的网络联通起来，方便后续使用，例如SSH登录
这里介绍一种简单的桥接的方式
## 在主机上增加网桥设备br0

```bash
vi /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE="br0"
ONBOOT="yes"
TYPE="Bridge"
BOOTPROTO=static
IPADDR=186.100.88.32 //主机IP
NETMASK=255.255.255.0
GATEWAY=186.100.88.1
DEFROUTE=yes
```
## 配置主机的网卡，将其加入br0网桥
在/etc/sysconfig/network-scripts/ifcfg-eth0中加入

```bash
BRIDGE="br0"
```
## 重启网络服务

```bash
service network restart
```

检测eth0是否已介入网桥

```bash
brctl show
```

## 虚拟机配置
修改虚拟机的配置文件中devices中的network部分

```xml
<interface type='bridge'>
      <mac address='52:54:00:da:c3:dc'/>
      <source bridge='br0'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

配置虚拟机IP和网关

```bash
vi /etc/sysconfig/network-scripts/ifcfg-eth0
IPADDR=186.100.88.100 //需和br0在同一网段
NETMASK=255.255.255.0
GATEWAY=186.100.88.1
BOOTPROTO=static //dhcp会自动分配地址
```

重启网络服务后，大功告成
***
# 最后的问题
难道我每次创虚拟机都要装一遍系统？
当然不是，首次装好系统后的磁盘镜像就可以用来给后续的虚拟机做模板
## 拷贝磁盘镜像

```bash
cp centos_vm.qcow2 centos_vm2.qcow2
```

##为新的vm准备配置文件

```bash
vi vm2.xml

<domain type='kvm'>  
  <name>vm2</name> 
  <uuid>243535db-2558-43e9-b067-314f48211343</uuid>  
  <memory>524288</memory>
  <currentMemory>524288</currentMemory> 
  <vcpu>2</vcpu>          
  <os>
   <type arch='x86_64' machine='pc-i440fx-vivid'>hvm</type> 
   <boot dev='hd'/>  
  </os>
  <features>
   <acpi/>  
   <apic/>  
   <pae/>   
  </features>
  <clock offset='localtime'/>  
  <on_poweroff>destroy</on_poweroff>  
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>   
   <emulator>/usr/libexec/qemu-kvm</emulator> 
   <disk type='file' device='disk'> 
      <driver name='qemu' type='qcow2'/>
      <source file='/home/tmp/qemu/centos_vm2.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/> 
   </disk>
   <interface type='network'>   
     <mac address='52:54:00:e1:1c:84'/>  
     <source network='default'/> 
     <target dev='vnet1'/> 
     <alias name='net1'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>  <!--注意slot值-->
   </interface>
  <graphics type='vnc' port='5902' autoport='yes' listen='186.100.88.33' keymap='en-us'/>  
   <listen type='address' address='186.100.88.33'/>
  </graphics>
  </devices>
</domain>
```
## 导入配置文件

```bash
virsh define vm2.xml
```
## 启动虚拟机

```bash
virsh start vm2
```

通过vnc上去看看吧，当然你还需要按前述的方法配置网络

