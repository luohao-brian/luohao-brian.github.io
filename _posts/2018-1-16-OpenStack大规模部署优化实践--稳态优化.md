---
layout: post
title: OpenStack大规模部署优化之二：稳态优化
categories: 稳态优化
tags: [OpenStack]
---

> OpenStack在架构设计上是松耦合解耦架构，天生支持横向扩展；但真正在大规模部署过程中，仍有好多因素决定其部署规模。本文从稳态方面总结分享原生OpenStack支撑大规模(千节点量级)部署的优化思路
OpenStack随着计算节点规模增大，计算节点上的各服务的agent个数会随之增加，比如nova-compute、neutron-agent、ceilometer-agent等。在稳态下，由于个agent存在周期性任务，随着这些agent的增加，主要是周期任务会对系统造成压力，优化点如下。

-----

### 调整MQ连接数，降低MQ压力
**问题描述**：OpenStack各服务内部通信都是通过RPC来交互，各agent都需要去连接MQ；随着各服务agent增多，MQ的连接数会随之增多，最终可能会到达上限，成为瓶颈
**解决思路1**：调整各Agent的RPC连接池大小；连接池默认大小为30，假如有1000个计算节点，nova-compute所需的MQ连接数上限总数就是3W，所以在真正部署过程中，需要对此值进行权衡优化；对应参数如下：
```sh
[default]
# Size of RPC connection pool. (integer value)
# Deprecated group;name - DEFAULT;rpc_conn_pool_size
rpc_conn_pool_size=30
```
**解决思路2**：提升MQ连接数上限；MQ的连接数取决于ulimit文件描述符的限制，一般缺省值为1024，对于MQ服务器来说此值偏小，所以需要重新设置linux系统中文件描述符的最大值，设置方法具体如下。
（1）通过rabbitmqctl命令查看rabbitmq当前支持的最大连接数
```sh
[root@local~]#rabbitmqctl status
Status of node 'rabbit@192-168-0-15'
[{pid,198721},
 ......
 {file_descriptors,
     [{total_limit,924},
      {total_used,10},
      {sockets_limit,829},
      {sockets_used,10}]},
 ......
]
```

（2）通过ulimit查看当前系统中文件描述符的最大值
```sh
[root@local~]# ulimit -n
1024
```

（3）调整最大值
方式1：在rabbitmq启动脚本中增加ulimit -n 10240，修改rabbitmq进程的文件描述符最大值
方式2：修改/etc/security/limits.conf文件中的如下配置
```sh
user    soft    nofile    10240
user    hard    nofile    10240
```

方式3：修改/etc/sysctl.conf文件，增加如下配置项
```sh
fs.file-max=10240
```
### 优化周期任务，减少MQ压力
**问题描述**：稳态下各agent存在一些周期任务周期调用RPC，随着agent增多，RPC的次数会增多，MQ压力会增大
**解决思路1**：优化周期任务，对周期任务周期进行审视，在允许的情况下，可适当增大周期，减少MQ压力；比如心跳上报周期，nova-compute默认10s上报一次心跳；nova在对服务判断是否存活的时间默认60s（取决于service_down_time）;在大规模场景下，可以调整report_interval为20s；在1000节点规模下，nova-compute心跳优化前为100次/s，优化后50次/s；对应参数如下
```sh
[default]
# Seconds between nodes reporting state to datastore (integer value)
report_interval=10
```
**解决思路2**：对MQ进行分库处理，不同服务采用不同的MQ

### 优化周期任务，减少数据库压力
**问题描述**：稳态下各agent存在一些周期任务周期调用数据库操作，nova-compute数据库操作是通过RPC到nova-conductor进行，随着计算节点增多，数据库以及nova-conductor压力会随之增大
**解决思路1**：优化周期任务周期，同上；当然在上报心跳的周期也可以采用memcache存储后端，直接减少不必要的数据库调用，配置项如下
```sh
# This option specifies the driver to be used for the servicegroup service.
# ServiceGroup API in nova enables checking status of a compute node. When a
# compute worker running the nova-compute daemon starts, it calls the join API
# to join the compute group. Services like nova scheduler can query the
# ServiceGroup API to check if a node is alive. Internally, the ServiceGroup
# client driver automatically updates the compute worker status. There are
# multiple backend implementations for this service: Database ServiceGroup driver
# and Memcache ServiceGroup driver.
# Possible Values:
#     * db : Database ServiceGroup driver
#     * mc : Memcache ServiceGroup driver
servicegroup_driver = mc
```
**解决思路2**：调整nova-conductor对应的worker数目，支撑大规模场景，对应参数如下
```sh
[conductor]
# Number of workers for OpenStack Conductor service. The default will be the
# number of CPUs available. (integer value)
workers=1
```
