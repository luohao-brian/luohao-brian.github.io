---
layout: post
title: 三分钟搞定MacOS挂载NTFS写入问题
tags: [Mac]
---

### [](#背景)背景

MacOS插入U盘或者移动磁盘，如果是Windows NTFS格式，默认是只读模式，只能看不能用的感觉大家懂的。

### [](#Paragon NTFS)Paragon NTFS

最简单有效的解决办法就是Money, [Paragon NTFS](http://www.ntfsformac.cn/)是MacOS上老牌NTFS挂载软件，安装后可以免费试用一周，以后就必须付费了，目前的价格是￥149.00。我在MacOS Sierra版本购买过CleanMyMac，但是到了Mojove居然不能用了，还要重新购买，为了避免同样的悲剧再次发生，所以我不打算做冤大头了。

### [](#FUSE for macOS)FUSE for macOS

安装ntfs-3g

```
brew install ntfs-3g
```

然后插入U盘或者活动硬盘，用diskutil查看磁盘名称和标识符：

```
diskutil list
```

![image](https://raw.githubusercontent.com/luohao-brian/luohao-brian.github.io/master/img/posts-2019/diskutil.png)

用读写模式重新挂载指定磁盘:

```
sudo mkdir /Volumes/NTFS
sudo umount /dev/disk2s1
sudo /usr/local/bin/ntfs-3g /dev/disk2s1 /Volumes/NTFS -olocal -oallow_other
```

`CAUSION`: FUSE是用户态的文件系统，性能非常堪忧，我实测就1M左右的写入速度，小文件拷贝是不错的选择。实际我尝试拷贝几部数G的大文件，悲剧的是还经常出现莫名其妙的写入中断，无奈最后放弃了。


### [](#Experimental NTFS-Writing)Experimental NTFS-Writing

MacOS其实也支持NTFS写入，只不过由于还不是很稳定，一直处于试验(Experimental)阶段，没有正式支持。如果要开启NTFS-Writing, 先编辑`/etc/fstab`, 其中`NAME`是使用`diskutil list`命令获得的磁盘名。

```
LABEL=NAME none ntfs rw,auto,nobrowse
```

然后拔下磁盘并重新插入，指定的磁盘会被挂载到`/Volumes`目录，这个目录默认并不会显示在桌面上，必须通过`前往`->`前往文件夹`指定。

`CAUSION`: MacOS NTFS-Writing毕竟是试验阶段(其实已经试验了很多年了)，稳定性还有些堪忧，比如我尝试把30G的多个电影文件复制到活动硬盘某个很深的文件夹中，结果复制完成后发现文件居然不见了，还好后来在Windows上挂载后用磁盘检查恢复了。

### [](#参考)参考

* [How to Write to NTFS Drives on a Mac](https://www.howtogeek.com/236055/how-to-write-to-ntfs-drives-on-a-mac/)

