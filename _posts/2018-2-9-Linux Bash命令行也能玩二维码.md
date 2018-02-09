---
layout: post
title: Linux Bash命令行也能玩二维码
tags: [Linux]
---

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-02-09/40270003ca54b2fb661f.jpg)

------

### Linux Bash命令行上的二维码

打开一个Linux终端，输入下面的命令，你会在终端上获得这个网站的二维码，以字符的方式显示在你的终端上，用微信扫一扫这个二维码，就会跳转到这个网站。

```
echo "http://114.115.144.21/" | curl -F-=<- qrenco.de
```

### 浏览器上的二维码

打开一个浏览器，在地址栏输入如下地址，将会在浏览器上获得这个网站的二维码，以字符的方式显示在你的浏览器上:

```
http://qrenco.de/http://114.115.144.21/
```