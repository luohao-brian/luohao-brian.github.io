---
layout: post
title: OpenStack大规模部署优化之一：并发业务优化
categories: 并发业务优化
tags: [OpenStack]
---

> OpenStack在架构设计上是松耦合解耦架构，天生支持横向扩展；但真正在大规模部署过程中，仍有好多因素决定其部署规模。本文从业务并发方面总结分享原生OpenStack支撑大规模(千节点量级)部署的优化思路
在大规模并发业务过程中，主要是去除红绿灯（数据库行级锁）解决锁抢占问题，以及修多条高速公路(调整各组件进程数)最终提升各组件的处理能力

------
### 调整haproxy进程数，提升Loadbalance能力
**问题描述**：在openstack部署过程中，通常采用haproxy作为前端负载均衡器，在大规模并发过程中，需要观察haproxy的CPU使用率，如果到达了100%，则需要进行优化
**解决思路**：调整haproxy的进程数，支撑大规模并发，参数如下
```sh
global
    nbproc 16  #进程个数
```
### 调整OpenStack各组件进程数，提升组件处理能力
**问题描述**:在大规模业务并发并发过程中，各组件处理能力不足（可以观察进程对应cpu使用率，如果已经到100%，说明处理能力不足）
**解决思路**:可以通过横向扩展组件或调整组件worker数来解决
### 数据库、MQ分库处理，提升性能
问题描述：大规模并发过程中，业务处理会对数据库和MQ，造成比较大的压力，导致业务下发失败
解决思路：对MQ和数据库进行分库处理，不同服务采用不同的库进行优化
### 优化数据库连接数，减少数据库连接总数
问题描述：并发业务处理，需要连接数据库，并发度高的时候，提示数据库连接超过了上限
解决思路：调整各组件的数据库连接数配置，取决于max_pool_size（连接池大小）和max_overflow（最大允许超出的连接数）
```sh
[database]
# Maximum number of SQL connections to keep open in a pool. (integer value)
max_pool_size=10
 
# If set, use this value for max_overflow with SQLAlchemy. (integer value)
max_overflow=10
```
### 采用缓存调度器，提升虚拟机调度时间
**问题描述**：并发调度过程中，调度前都会去数据库读取主机信息，耗时长导致调度时间长
**解决思路**：采用缓存调度器，缓存主机信息，提升调度时间
```sh
#caching_scheduler which aggressively caches the system state for better
scheduler_driver=caching_scheduler
```
### 基于存储内部快速复制能力，缩短镜像创建卷的时间
**问题描述**:单个虚拟机创建耗时长的点主要集中在镜像创建卷，在创建过程中，需要下载镜像，所以创建时间跟镜像大小以及网络带宽强相关
**解决思路**:可以基于存储内部快速复制卷的能力，解决系统卷创建慢的问题，有以下三种方式
**方式1：在cinder上对镜像卷进行缓存**
openstack社区提供了缓存镜像卷的能力，核心思想，第一次创建卷的时候，在存储后端缓存对应的镜像卷，后续创建都是基于这个镜像卷复制一个新的卷。

社区相关链接：[http://docs.openstack.org/admin-guide/blockstorage-image-volume-cache.html](http://docs.openstack.org/admin-guide/blockstorage-image-volume-cache.html)

**方式2：glance后端对接cinder，镜像直接以卷的形式存在cinder上**
这种方式，在镜像上传的过程中，直接以卷的形式存放，在从镜像创建的卷的过程中，直接走卷复制卷的能力。这种方式可以解决首次发放慢的问题。

社区相关链接：[https://specs.openstack.org/openstack/cinder-specs/specs/liberty/clone-image-in-glance-cinder-backend.html](https://specs.openstack.org/openstack/cinder-specs/specs/liberty/clone-image-in-glance-cinder-backend.html)

**方式3：基于存储的差分卷能力实现卷的快速创建**
这一功能需要实现cinder volume中的clone_image方法，在这个方法里面，可以先缓存镜像快照，然后基于快照创建差分卷
### 采用rootwrap daemon方式运行命令，缩短nova/neutron等组件调用系统命令的时间
**问题描述**:rootwrap 主要用来做权限控制。在openstack中，非root用户想执行需要root权限相关的命令时，用rootwrap来控制。 启动虚拟机过程中，会多次调用系统命令；调用命令时，会经过rootwrap命令进行封装，这个命令在每次允许过程中，都会加载命令白名单（允许nova组件执行命令的列表配置文件），最终再调用实际命令运行，额外耗时100ms左右。
**解决思路**:通过rootwrap daemon机制来解决，启动一个rootwrap daemon专门接受执行命令的请求，节省每次加载白名单的时间
nova-compute对应的rootwrap配置项：
```sh
[DEFAULT]
use_rootwrap_daemon=True
```
社区相关链接：[http://docs.openstack.org/admin-guide/compute-root-wrap-reference.html](http://docs.openstack.org/admin-guide/compute-root-wrap-reference.html)
### Qutoa无锁化优化，减少操作Quota时的耗时
**问题描述**：openstack在Quota处理过程中，采用了数据库行级锁来解决并发更新问题，但在并发场景下，这个锁会导致耗时增加
**解决思路**：由于在处理Quota过程中，先select在update，所以需要加锁（悲观锁）。针对这一点，可以通过带有where的update操作来实现更新，然后根据更新行数，判断是否更新成功（乐观锁）

社区相关链接：[http://specs.openstack.org/openstack/nova-specs/specs/kilo/approved/lock-free-quota-management.html](http://docs.openstack.org/admin-guide/compute-root-wrap-reference.html)
### 调整nova-compute并发创建任务上线，提升组件的并发能力
**问题描述**：nova-compute在并发创建虚拟机过程中，有并发任务限制（M版本默认值为10）
**解决思路**：增大并发任务个数上线，对应参数为max_concurrent_builds
### keystone采用PKI机制替换UUID方式，减少keystone压力
**问题描述**：openstack api server在处理请求前会校验token是否合法，如果采用UUID token，则每次都会去keystone做校验
**解决思路**：采用PKI方式，各api在本地通过证书来校验token是否合法
### 适当增大各组件token失效列表的缓存时间，可以减少keystone的压力
**问题描述**：openstack api server在处理请求前会校验token是否合法，除了校验token是否过期，同时还校验token是否在token失效列表里面；这个token失效列表会在本地缓存，如果过期，则会去keystone重新获取，在并发的时候，keystone会成为瓶颈点
**解决思路**：适当增大各组件token失效列表的缓存时间revocation_cache_time