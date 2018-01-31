---
layout: post
title: GoTTY：基于go语言的Linux终端Web共享
categories: Gotty
tags: [Golang, Linux]
---
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/5.jpg)

> 现在很多公司的内网防火墙只允许http协议，从这样的网络去访问和控制云上的主机非常麻烦。GoTTY是一个用go语言开发的工具，它启动一个web应用服务，可以将任何指定的终端应用映射到指定的http端口，这样在防火墙内部的客户端就可以通过普通的浏览器Chrome, Firefox来访问。

------

### 安装
首先需要安装go语言>1.9环境, 如果是CentOS 7, 先要添加go-repo：
```sh
rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
```

然后使用yum安装golang：
```sh
yum install golang
```

如果是Mac OS, 执行
```sh
brew install go@1.9
```

设置go语言运行时需要的环境变量, 为了让这些环境变量每次都生效，可以将他们附加到~/.bashrc文件末尾：
```sh
export GOPATH=$HOME/Documents/go
export PATH="$PATH:$GOPATH/bin
```

执行下面的命令,它会从github下载并安装GoTTY
```sh
go get github.com/yudai/gotty
```
### 运行
对于非交互型的命令，比如top, 一般不需要我们在终端上输入，运行如下命令会默认在http://localhost:8080等本地地址上启动一个web应用，然后用浏览器访问这个地址，就可以看到top的持续输出了。
```sh
gotty top
```

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/6.jpg)

如果是交互命令，比如bash, 那么要使用-w参数，告诉gotty这个终端模拟需要允许写入，用浏览器访问http://localhost:8080，就可以像使用xshell一样访问控制这台主机了。
```
gotty -w bash
```

![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-1-16/7.jpg)

最后，[GoTTY官方网站](https://github.com/yudai/gotty)，更多详细的用法，可以参考主页上的文档。
