---
layout: post
title: 华为云ECS服务API使用指南(3) - 使用python cli获取keystone认证信息
tags: [云计算]
---

> 除了使用restful api，openstack还提供了丰富的开发语言binding, 包括python, java, golang等。这里简单介绍一些如何用python binding来实现基本的keystone操作。

------

### OpenStack CLI 软件包

建议安装Mitaka版本的OpenStack CLI，但是最新的CentOS 7.4已经deprecate了OpenStack Mitaka，所以要用7.3的repo.
```sh
#/etc/yum.repos.d/openstack.repo
[openstack]
name=OpenStack Mitaka
baseurl=http://mirrors.163.com/centos/7.3.1611/cloud/x86_64/openstack-mitaka/
gpgcheck=0
```

安装keystone cli
```sh
yum install python2-keystoneauth1
```

### 代码示例
```python
#!/usr/bin/env python

from keystoneauth1.identity import v3
from keystoneauth1 import session

auth = v3.Password(
    username='HEC_USER_NAME',
    password='HEC_USER_PASSWD',
    auth_url='https://iam.cn-north-1.myhwclouds.com/v3',
    user_domain_name='HEC_USER_NAME',
    project_domain_name='HEC_USER_NAME'
)
sess = session.Session(auth=auth, verify=False)
print sess.get_token()
#print sess.get_endpoint(service_type='compute')
```
