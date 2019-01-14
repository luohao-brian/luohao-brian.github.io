---
layout: post
title: 使用Superset可视化Mysql数据
tags: [Linux]
---

### 背景

Superset是一款轻量级的BI工具，由Airbnb的数据部门开源。整个项目基于Python框架，集成了Flask、D3、Pandas、SqlAlchemy等。

Superset本身集成了数据查询功能，支持各类主流数据库，包括MySQL、PostgresSQL、Oracle、Impala、SparkSQL等，深度支持Druid。

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/superset-1.gif)

### 安装

最新的superset已经不支持python2了，需要通过pip3来安装

```
pip3 install superset
```

我主要使用mysql作为数据源，Python3不再支持`MySQL-python`, 可以用下面命令替代:

```
pip3 install mysqlclient
```

安装成功后，需要进行初始化配置，也是在命令行输入:

```
fabmanager create-admin --app superset
```

首先用命令行创建一个admin管理员账户，也是后续的登陆账号。会依次提示输入账户名，账户使用者的first name、last name、邮箱、以及确认密码。fabmanager是flask的权限管理命令，如果大家忘了密码，也能重新设立。

```
superset db upgrade
```

初始化默认的用户角色和权限。
```
superset init
```

最后一步骤，启动Superset服务。

```
superset runserver
```

因为我们是本地环境，所以在浏览器输入 http://localhost:8088 即可。在runserver后面添加`-p XXXX` 可更改为其他端口。

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/superset-login.jpg)

### 使用

首先要连接数据库，添加数据源：

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/superset-datasource.png)

比如mysql, 这里可以按如下格式配置连接，然后点选下面的`测试链接`按钮验证连接是否配置成功。

```
mysql://DB_USER:DB_PASS@DB_HOST:DB_PORT/DB_NAME
```

进入到SQL工具箱，输入SQL语句，直接出来了数据库的数据预览。语法和MySQL一致。

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/superset-runsql.jpg)

选择Visualize/Explore，进入切片绘图模式。

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/superset-explore.png)

保存绘图后可以在绘图菜单中选择，然后可以进一步加入到看板中。

### 参考
 * [一小时建立数据分析平台](https://zhuanlan.zhihu.com/p/28485468)
