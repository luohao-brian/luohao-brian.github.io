---
layout: post
title: CentOS 7安装最新版本Golang并让vim支持go语法高亮
tags: [VIM]
---

> 这篇文章是我学习Golang的第一篇记录，从编程环境搭建和优化开始。CentOS 7默认对Golang的不是很完善，比如Golang版本很低，而且vim编辑器也不支持go语法高亮。

------

### Keep Golang Updated

go-repo.io维护了golang对RedHat系列各发行版的Golang最新版本，对于CentOS 7，可以通过下面的方式安装:

```bash
rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO
curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
yum install -y golang
```

### vim-go插件

下载安装vim-go插件:

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
git clone https://github.com/fatih/vim-go.git ~/.vim/plugged/vim-go
```

编辑~/.vimrc, 加入以下内容:

```bash
call plug#begin()
Plug 'fatih/vim-go', { 'do': ':GoInstallBinaries' }
call plug#end()
```


That's it!




