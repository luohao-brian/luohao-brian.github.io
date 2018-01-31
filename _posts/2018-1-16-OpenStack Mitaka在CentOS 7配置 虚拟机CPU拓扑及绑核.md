---
layout: post
title: OpenStack Mitaka在CentOS 7配置 虚拟机CPU拓扑及绑核
categories: 虚拟机CPU拓扑及绑核
tags: [OpenStack]
---

> 花了两天时间，终于在CentOS 7.3下搞定了OpenStack Mitaka的虚拟机CPU拓扑和绑核配置，解决了一个一个的坑，记录一下。

-----

### KVM版本
CentOS 7默认的qemu 1.5版本非常老，OpenStack libvirt driver好些功能都因此不能使用，所以一定要使用专门仓库的qemu-kvm，我因此遇到的问题包括：
- compute节点的resource_tracker不能上报numa_topoloty到controller节点，导致NUMATopologyFilter无法调度；
- nova-compute创建虚拟机失败，日志显示: Per-node memory binding is not supported with this QEMU；

### 安装qemu-kvm-ev
CentOS 7有一个virt repo, 提供了很多最新的虚拟化软件包，其中之一就是qemu-kvm-ev, 基于qemu 2.6.0构建, 网易的CentOS镜像也包括这个repo：
```sh
$ sudo rpm -Uvh http://mirrors.163.com/centos/7.3.1611/virt/x86_64/kvm-common/centos-release-qemu-ev-1.0-1.el7.noarch.rpm
 
$ sudo yum install qemu-kvm-ev
```

### 配置OpenStack
以下内容可以参考[Nova Admin手册中CPU topologies章节](https://docs.openstack.org/nova/pike/admin/cpu-topologies.html):
**nova.conf**
添加NUMATopologyFilter调度器，nova默认没有启用这个filter.
```sh
scheduler_default_filters=...,NUMATopologyFilter
```
**设置flavor**
```sh
$ openstack flavor set m1.large \
  --property hw:cpu_policy=dedicated \
  --property hw:cpu_thread_policy=require
```
Valid CPU-POLICY values are:
- shared: (default) The guest vCPUs will be allowed to freely float across host pCPUs, albeit potentially constrained by NUMA policy.
- dedicated: The guest vCPUs will be strictly pinned to a set of host pCPUs. In the absence of an explicit vCPU topology request, the drivers typically expose all vCPUs as sockets with one core and one thread. When strict CPU pinning is in effect the guest CPU topology will be setup to match the topology of the CPUs to which it is pinned. This option implies an overcommit ratio of 1.0. For example, if a two vCPU guest is pinned to a single host core with two threads, then the guest will get a topology of one socket, one core, two threads.

Valid CPU-THREAD-POLICY values are:
- prefer: (default) The host may or may not have an SMT architecture. Where an SMT architecture is present, thread siblings are preferred.
- isolate: The host must not have an SMT architecture or must emulate a non-SMT architecture. If the host does not have an SMT architecture, each vCPU is placed on a different core as expected. If the host does have an SMT architecture - that is, one or more cores have thread siblings - then each vCPU is placed on a different physical core. No vCPUs from other guests are placed on the same core. All but one thread sibling on each utilized core is therefore guaranteed to be unusable.
- require: The host must have an SMT architecture. Each vCPU is allocated on thread siblings. If the host does not have an SMT architecture, then it is not used. If the host has an SMT architecture, but not enough cores with free thread siblings are available, then scheduling fails.

**创建虚拟机**
```sh
openstack server create --image cirros --nic net-id=provider-net --flavor m1.large instance-001
```

**验证**
virsh dumpxml domID输出的xml应该包括类似下面的内容
```sh
<cputune>
<vcpupin vcpu='0' cpuset='2'/>
<vcpupin vcpu='1' cpuset='3'/>
<emulatorpin cpuset='2-3'/>
</cputune>
```

