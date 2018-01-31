---
layout: post
title: 5分钟让你的CentOS 7字体和Windows 7一样漂亮
tags: [Linux]
---
> CentOS是一个面向工作站和服务器的linux发行版，本来不适合作桌面，最近的CentOS中都自带了中文字体和中文 输入法，相比Windows和Mac OS，CentOS的显示效果实在是惨不忍睹，稍微多看一眼都会觉得眼睛疼:) 其实这主要是两个问题导致的，找到正确的方法5分钟就可以让你的CentOS 7字体和Windows 7一样漂亮。

* 问题1：CentOS默认的字体是文泉驿中文字体，相比MS Windows的字体效果还是有不小的差距；
* 问题2：CentOS自带的Freetype对于中文字体的渲染有bug, 需要用infinality补丁修正一下；

------
### 安装Windows字体
**获得字体**
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/19.jpg)

从`C:/Windows/fonts`下拷贝如下字体
* DejaVuSansMono_0.ttf
* DejaVuSansMono-Bold_0.ttf
* DejaVuSansMono-BoldOblique_0.ttf
* DejaVuSansMono-Oblique_0.ttf
* msyhbd.ttf
* msyh.ttf
* simhei.ttf
* simsun.ttc
* tahomabd.ttf
* tahoma.ttf

**安装字体**
打开一个终端，执行下面的命令：
```sh
yum install -y ttmkdir
mkdir /usr/share/fonts/chinese
cp -rf *.ttf *.ttc /usr/share/fonts/chinese
vim /etc/fonts/fonts.conf
```
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/20.jpg)

```sh
fc-cache
fc-list
```
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/21.jpg)

### Infinality字体补丁
CentOS官方repo中的freetype对中文支持有问题，需要infinality修正，方便的是，nux dextop集成提供了改进后的freetype, 可以添加相关repo后直接安装使用：
```sh
$ yum install http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
$ yum update
$ yum install freetype-infinality
```
可以通过/etc/profile.d/infinality-settings.sh设置字体的各种渲染风格，这里我选的是Windows 7：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/22.jpg)

### 设置字体
我希望我的gnome-termial和windows下常用的xshell显示效果一致，我选择了xshell在Windows下默认使用的Dejavu字体：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/23.jpg)

> Dejavu字体

系统字体可以通过gnome-tweak-tool来设置，可以选择黑体或者Tahoma
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/24.jpg)

> 黑体或者Tahoma 作为系统默认字体

Chrome浏览器使用微软黑体效果如下：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-17/25.jpg)

> 使用infinity黑体渲染的chrome网页显示效果

从此，整个世界变得清爽多了。