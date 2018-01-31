---
layout: post
title: OpenStack基本概念Domain，Project，User，Role
categories: Domain，Project，User，Role
tags: [OpenStack]
---
### Domain，project，user，role的概念和关系
- Domain - 表示 project 和 user 的集合，在公有云或者私有云中常常表示一个客户
- Group - 一个domain 中的部分用户的集合
- Project - IT基础设施资源的集合，比如虚机，卷，镜像等
- Role - 角色，表示一个 user 对一个 project resource 的权限
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/12.jpg)


**说明**：
说明：
- Domain 可以认为是 project，user，group 的 namespace。 一个 domain 内，这些元素的名称不可以重复，但是在两个不同的domain内，它们的名称可以重复。因此，在确定这些元素时，需要同时使用它们的名称和它们的 domain 的 id 或者 name。
- Group 是一个 domain 部分 user 的集合，其目的是为了方便分配 role。给一个 group 分配 role，结果会给 group 内的所有 users 分配这个 role。
Role 是全局（global）的，因此在一个 keystone 管辖范围内其名称必须唯一。 role 的名称没有意义，其意义在于 policy.json 文件根据 role 的名称所指定的允许进行的操作。

简单地，role 可以只有 admin 和 member 两个，前者表示管理员，后者表示普通用户。但是，结合 domain 和 project 的限定，admin 可以分为 cloud admin，domain admin 和 project admin。
- policy.json 文件中定义了 role 对某种类型的资源所能进行的操作，比如允许 cloud admin 创建 domain，允许所有用户创建卷等
- project 是资源的集合，其中有一类特殊的project 是 admin project。通过指定 admin_project_domain_name 和 admin_project_name 来确定一个 admin project，然后该project 中的 admin 用户即是 cloud admin。

### OpenStack Mitaka默认安装的Domain, Project, User, Group, Role
如果按OpenStack的安装手册默认安装，我们会得到如下的拓扑视图:
Domain:
- default

Project:
- admin
- service
- demo

Role
- admin
- user

User:
- glance
- nova
- neutron
- ...
- user

其中，系统用户glance, nova, neutron在每个服务初始化安装的时候，被如下的命令与service project及admin role关联起来:
```sh
openstack role add --project service --user glance admin
```

而演示用户demo会和demo project及user role关联起来：
```sh
openstack role add --project demo --user demo user
```

