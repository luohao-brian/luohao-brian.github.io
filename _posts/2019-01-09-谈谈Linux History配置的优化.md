---
layout: post
title: 谈谈Linux History配置的优化
tags: [Linux]
---

### Pain #1 - 不带时间戳，不知道命令是什么时候发生的

```
echo 'export HISTTIMEFORMAT="%F %T "' >> ~/.bashrc
```

或者

```
echo 'export HISTTIMEFORMAT="%F %T "' >> /etc/profile
```

生效后的history将会是这个样子：

```
# history
 1 2017-08-16 15:30:15 yum install -y mysql-server mysql-client
 2 2017-08-16 15:30:15 service mysqld start
 3 2017-08-16 15:30:15 sysdig proc.name=sshd
 4 2017-08-16 15:30:15 sysdig -c topprocs_net
 5 2017-08-16 15:30:15 sysdig proc.name=sshd
 6 2017-08-16 15:30:15 sysdig proc.name=sshd | more
```
### Pain #2 - 默认只保留500条记录

```
# 设置历史记录条数
echo 'export HISTFILESIZE=100000' >> >> ~/.bashrc
# 设置显示历史记录条数
echo 'export HISTSIZE=10000' >> ~/.bashrc
```
或者

```
# 设置历史记录条数
echo 'export HISTFILESIZE=100000' >> /etc/profile
# 设置显示历史记录条数
echo 'export HISTSIZE=10000' >> /etc/profile
```

### Pain #3 - 多个终端相互覆盖历史记录

现在通过多个ssh终端连接到同一个服务器是非常常见的操作，但是bash默认多个终端会相互覆盖历史记录，通过下面的配置可以让终端退出的时候采用append方式而不是overwrite保存历史记录:

```
echo 'shopt -s histappend' >> ~/.bashrc
```

或者

```
echo 'shopt -s histappend' >> /etc/profile
```

### 参考
* [让 history 命令显示日期和时间](https://linux.cn/article-9253-1.html)
* [bash 下 history 会因多个终端而覆盖丢失，有好的解决方案吗？](https://www.zhihu.com/question/19863362)

