---
layout: post
title: Nginx部署Let's Encrypt免费SSL证书
tags: [SSL]
---

### 申请Let's Encrypt永久免费SSL证书

#### Let's Encrypt简介

[Let's Encrypt](https://link.jianshu.com?t=https%3A%2F%2Fletsencrypt.org%2F)公共且免费SSL的项目

[Let's Encrypt免费SSL](https://link.jianshu.com?t=https%3A%2F%2Fletsencrypt.org%2F)支持包括FireFox、Chrome在内的主流浏览器的兼容和支持，虽然目前是公测阶段，但是也有不少的用户在自有网站项目中正式使用起来。

#### 步骤如下：

**第一、安装Let's Encrypt前的准备工作**

```
#检查系统是否安装git
git  --version 
#git 安装
yum install git
#查看python版本,尽量保证在2.7以上
python -v 
```

**第二、获取Let's Encrypt免费SSL证书**
//当时配置的时候,碰到好多错误,还好网上都有了解决办法,大家出错的时候可以看看letsencrypt下面的日志,里面报错内容都有,把错误尽量都解决了,再安装,不然错一次申请一次证书,次数用完,就要在等一周才能申请了.
```
#获取letsencrypt
git clone https://github.com/letsencrypt/letsencrypt
#进入letsencrypt目录
cd letsencrypt
#生成证书1
//可以填两个域名，也可以填一个
./letsencrypt-auto certonly --standalone --email your_email@address -d your_url_address -d www.your_url_address

#生成证书2
//我一般用这个
./letsencrypt-auto
1. 填入邮箱地址
2. 选择A
3. 选择Y
4. 填入域名地址    //多域名用空格或者/或者,隔开
```

**第三、Let's Encrypt免费SSL证书获取与应用**

在完成Let's Encrypt证书的生成之后，我们会在"/etc/letsencrypt/live/your_url_address/"域名目录下有4个文件就是生成的密钥证书文件。

cert.pem - Apache服务器端证书
chain.pem - Apache根证书和中继证书
fullchain.pem - Nginx所需要ssl_certificate文件
privkey.pem - 安全证书KEY文件

如果我们使用的Nginx环境，那就需要用到fullchain.pem和privkey.pem两个证书文件，在部署Nginx的时候需要用到。在Nginx环境中，只要将对应的ssl_certificate和ssl_certificate_key路径设置成我们生成的2个文件就可以。

```
#打开linux配置文件，找到HTTPS 443端口配置的server,如果有以下文件就不需要再添加了，没有的话就添加上
 ssl_certificate /etc/letsencrypt/live/your_url_address/fullchain.pem;
 ssl_certificate_key /etc/letsencrypt/live/your_url_address/privkey.pem;
```

**第四、解决Let's Encrypt免费SSL证书有效期问题**

Let's Encrypt证书是有效期90天的，需要我们自己手工更新续期才可以。
手工续期命令如下：_

```
./letsencrypt-auto certonly --renew-by-default --email your_email@address -d your_url_address -d www.your_url_address
```
这样我们就可以续期,当然我们也可以写一个`cron`脚本让他自动续期

### 从证书中删除域名

Let’s Encrypt目前并不提供从证书中删除域名的功能,所以我们只是把证书删除后重新申请.
> 证书每周申请数量有限,不要一直删了申请

删除证书的时候，需要删除archive中的文件和live中的符号链接，同时还需要删除证书更新的配置文件：
```
rm -rf /etc/letsencrypt/live/www.example.com/
rm -rf /etc/letsencrypt/archive/www.example.com/
rm /etc/letsencrypt/renewal/www.example.com.conf
```

删除后，重新申请证书，这里给出的是webroot方式，当然你也可以用上面的方法:
```
letsencrypt certonly --webroot -w /var/www/example -d your_url_address -d www.your_url_address
```

# 自动续期脚本
```
//这个脚本放在和let's encrypt目录同级的目录下
#!/bin/sh
# This script renews all the Let's Encrypt certificates with a validity < 30 days

if ! /opt/letsencrypt/letsencrypt-auto renew > /var/log/letsencrypt/renew.log 2>&1 ; then
    echo Automated renewal failed:
    cat /var/log/letsencrypt/renew.log
    exit 1
fi
```
**权限**
```
chmod +x renewCerts.sh
```
**cron脚本**
```
//每个月凌晨1点运行
0 0 1 * * /bin/sh /opt/renewCerts.sh
```
