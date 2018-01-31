---
layout: post
title: 正确的姿势：三分钟搞定CentOS 7上最新tensorflow-gpu的安装
categories: 
tags: [人工智能]
---
> Tensorflow是现在很火的深度学习软件，如果采用正确的姿势，充分利用CentOS的强大生态，我们可以在几分钟内搞定tensorflow的安装。

------

### python+pip
最新的tensorflow stable版本已经可以在pip源中找到了，也能支持CentOS默认的Python 2.7，因为pip从国外下载的速度很慢，建议配置使用豆瓣的pip源。
```sh
# vim ~/.pip/pip.conf
[global]
index-url = https://pypi.douban.com/simple
[list]
format=columns
```
### CUDA
由于Volta架构GPU的推出，最新的官方CUDA版本已经升级到9.0，Tensorflow 1.3需要至少8.0，从兼容性考虑，我继续选择了8.0。NVIDIA也为CentOS提供了 CUDA官方支持，所以安装配置也超级简单：

首先添加NVIDIA CUDA的repo:
```sh
yum install -y http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-9.0.176-1.x86_64.rpm && yum update
```
然后，用yum安装cuda 8.0，cuda的安装程序会自动搞定nvidia-kmod，xorg的驱动配置，并且将cuda安装到/usr/local目录下:
```sh
yum install cuda-8-0
```
值得注意的是，如果使用了rpmfusion的源，安装nvidia官方cuda版本的时候，需要将rpmfusion禁用掉，因为rpmfusion的non-free频道也包括了cuda相关软件，会和官方的版本造成冲突：
```sh
yum install cuda-8-0 --disablerepo=rpmfusion-*
```
在/etc/bashrc末尾添加CUDA需要的环境变量：
```sh
# vim /etc/bashrc
export CUDA_PATH=/usr/local/cuda
export PATH="$PATH:/usr/local/cuda/bin"
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```
最后，重启主机，让nvidia kmod和xorg-nvidia生效。
### CUDNN
最近NVIDIA官网的Deep Learning软件下载页面发疯，一直不可用，但是tensorflow还需要cudnn v6，我找了好久，终于在baidu网盘上找到了一个可用的分享[](https://pan.baidu.com/s/1hs23HrA), 拿走不谢。
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/1.jpg)

将下载的cudnn软件包解压, 并且将include和lib64下的内容复制到/usr/local/cuda对应的目录下：
```sh
tar -zxvf cudnn-8.0-linux-x64-v6.0.tgz
cp -rf cuda/include/* /usr/local/cuda/include/
cp -rf cuda/lib64/* /usr/local/cuda/lib64/
```
### tensorflow-gpu
pip在安装tensorflow的过程中需要编译安装一些包，所以需要安装gcc和python-devel:
```sh
yum install -y gcc python-devel
```
然后就可以用pip安装tensorflow-gpu了：
```sh
pip install tenflow-gpu
```
### 验证
搞个tf最简单的线性回归例子，用python执行：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2.jpg)

然后可以从结果输出中看到，计算是通过识别到的Geforce GTX 1060完成的：
![](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/3.jpg)