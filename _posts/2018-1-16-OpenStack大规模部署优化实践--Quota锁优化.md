---
layout: post
title: OpenStack大规模部署优化实践--Quota锁优化
categories: Quota锁优化
tags: [OpenStack]
---

> 当前OpenStack针对租户的配额（Quota）管理是基于数据库来实现，对配额进行操作过程中都是先加锁再更新，这样在并发操作过程中，会成为平静点；本文针对这一问题以Nova Quota模型进行展开分析（Cinder/Neutron原理一样），并给出优化思路。

-----


### 1、Quota模型
Nova在对Quota管理过程中，在数据库中涉及到三张表，QUOTA_USAGE（当前已使用的配额）、QUOTA（描述组合配额上限）、RESERVATIONS（维护中间态资源配额），其中QUOTA_USAGE核心字段如下（后面两张表不做介绍）：
```
ID    PROJECT_ID    RESOURCE    IN_USE
1    abc            cpu            10
2    abc            memory        4096
3    def            CPU            20
4    def            memory        2048
```
各字段描述如下：
PROJECT_ID：租户ID
RESOUCE：为资源类型，比如CPU、内存等
IN_USE：当前已使用的值
### 2、Quota更新机制
当用户在创建资源过程中，比如创VM，会对CPU、内存等Quota对应的IN_USE字段会更新，其中更新方式是采用悲观锁，先采用selct for update对租户对应的资源Quota多行进行加锁，加锁后再做更新操作，伪代码如下；在并发创建虚拟机过程中，对数据库多行加的锁会成为瓶颈点（python协程会放大此瓶颈点，后续文章展开说明）
```sh
start_transaction:
    with lock(row)  //对应的SQL：select * from QUOTA_USAGE where PROJECT_ID=XXXX for update
        for resource, amount in requested_resource_changes:
            // 获取租户对应的quota
            in_use = db.get_quota(resource,tanent_id)  // 对应SQL: select IN_USE from QUOTA_USAGE where PROJECT_ID=XX
            if in_use == MAX_QUOTA
                //已到上限值，返回失败
                return false
            // 更新quota
            db.update_quota(resource,tanent_id, in_use+amount) //对应SQL：update QUOTA_USAGE set IN_USE=XX WHERE PROJECT_ID=XX
commit_transaction
```
### 3、优化思路
基于数据库的原子操作，采用乐观锁进行实现，先尝试带条件的更新，更新失败说明超出了上限，则返回失败，伪代码如下：
```sh
for resource, amount in requested_resource_changes:
    sql = "UPDATE quota_usage SET used = used + amount
           WHERE resource = $resource
           AND used + amount < MAX_QUOTA
           AND PROJECT_ID=xxx"
    update_rows = execute_sql(sql)
    if update_rows > 0
        return OK
    else
        return REACHED_MAX_QUOTA
```