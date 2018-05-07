---
layout: post
title: CentOS 7通过SCL安装MariaDB 10.2
tags: [Linux]
---

### [](#启用scl)启用SCL

```
yum install centos-release-scl && yum -y update
```

### [](#安装mariadb-102)安装mariadb 10.2

```
yum --enablerepo=centos-sclo-rh -y install rh-mariadb102-mariadb-server
yum install rh-mariadb102
```

### [](#启动时自动设置mariadb102)启动时自动设置mariadb10.2

```
[root@www ~]# vi /etc/profile.d/rh-mariadb102.sh
# create new
 #!/bin/bash

source /opt/rh/rh-mariadb102/enable
export X_SCLS="`scl enable rh-mariadb102 'echo $X_SCLS'`"
```

### [](#配置并启动mariadb102)配置并启动mariadb10.2

```
[root@www ~]# vi /etc/opt/rh/rh-mariadb102/my.cnf.d/mariadb-server.cnf
# add follows into [mysqld] section
[mysqld]
character-set-server=utf8
[root@www ~]# systemctl start rh-mariadb102-mariadb 
[root@www ~]# systemctl enable rh-mariadb102-mariadb
[root@www ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

# Enter
Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

# set root password
Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

# remove anonymous users
Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

# disallow root login remotely
Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

# remove test database
Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

# reload privilege tables
Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

# connect to MariaDB with root
[root@www ~]# mysql -u root -p 
Enter password:     # MariaDB root password you set
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.2.8-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# show user list
MariaDB [(none)]> select user,host,password from mysql.user; 
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | ***************************************** |
| root | 127.0.0.1 | ***************************************** |
| root | ::1       | ***************************************** |
+------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)

# show database list
MariaDB [(none)]> show databases; 
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

MariaDB [(none)]> exit
Bye
```

### [](#参考)参考

*   [https://www.server-world.info/en/note?os=CentOS_7&p=mariadb102&f=1](https://www.server-world.info/en/note?os=CentOS_7&p=mariadb102&f=1)
*   [https://www.softwarecollections.org/en/scls/rhscl/rh-mariadb102/](https://www.softwarecollections.org/en/scls/rhscl/rh-mariadb102/)

