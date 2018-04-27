---
layout: post
title: CentOS-7安装fcitx输入法
tags: [Linux]
---

### EPEL

CentOS 7的EPEL源中包含了fcitx和拼音输入法，所以需要把EPEL源引入。

```bash
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-1.noarch.rpm 
sudo rpm -ivh epel-release-7-1.noarch.rpm
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```

### 安装

```bash
# Remove ibus 
yum remove ibus 
yum remove imsettings imsettings-libs im-chooser

# Install fcitx 
yum install fcitx fcitx-table-chinese 
```

### 配置
在~/.bashrc中添加如下内容

```bash
export GTK_IM_MODULE=fcitx 
export QT_IM_MODULE=fcitx 
export XMODIFIERS=”@im=fcitx” 
```

退出当前session并重新登录。

执行：
```bash
$ gsettings set org.gnome.settings-daemon.plugins.keyboard active false 
$ imsettings-switch fcitx 
```

重启系统，按ctrl+空格可以呼出fcitx

### 诊断

如果你选择fcitx后报错显示：

```
GDBus.Error:org.gtk.GDBus.UnmappedGError.Quark.imsettings2derror_2dquark.Code5: Current desktop isn’t targeted by IMSettings.
```

查看日志，关键一句为：

```
INFO: Attempting to switch IM to FCITX [lang=en_US.utf8, update=true] org.gnome.settings-daemon.plugins.keyboard.active is true. imsettings is going to be disabled.
```
这就需要对gsetting设定：

```bash
gsettings set org.gnome.settings-daemon.plugins.keyboard active false
```
