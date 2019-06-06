---
layout: post
title: Ubuntu和CentOS定制内核驱动模块
tags: [Linux]
---

### 背景

一些特殊设备的驱动程序或者内核模块，并没有被开机默认加载，比如reiserfs的内核模块reiserfs.ko。

内核模块方式的驱动一般在init ramdisk中，在grub中用initrd参数指定:

```
menuentry 'Debian GNU/Linux' {
    ...
    echo	'Loading Linux 4.19.28 ...'
    linux	/boot/vmlinuz-4.19.28 root=UUID=...
    echo	'Loading initial ramdisk ...'
    initrd	/boot/initrd.img-4.19.28
}
```

而Ubuntu/Debian采用initramfs-tools生成initrd, 而CentOS采用drucat。

### Ubuntu & Debian

制作一个新的initrd.img
```
# mkinitramfs -o initrd.img
```

生成的initrd是一个cpio格式文件，但是不能用cpio解开

```
# file initrd.img
initrd.img: ASCII cpio archive (SVR4 with no CRC)
```
使用unmkinitramfs解开到initrd目录下

```
# unmkinitramfs initrd.img initrd
```

如果新增一个驱动到initramfs, 编辑/etc/initramfs-tools/modules，加入需要的module名。
```
# List of modules that you want to include in your initramfs.
# They will be loaded at boot time in the order below.
#
# Syntax:  module_name [args ...]
#
# You must run update-initramfs(8) to effect this change.
#
# Examples:
#
# raid1
# sd_mod

reiserfs
```

然后执行update-initramfs -u或者mkinitramfs重新生成；

默认生成的initramfs会比较大，这是因为`/etc/initramfs-tools/initramfs.conf`配置中`MODULES=most`导致，可以改为`MODULES=dep`;

### CentOS

CentOS在initrd中添加驱动：

```
dracut --add-drivers virtio_blk -f /boot/initramfs-3.10.5-1.el6.elrepo.x86_64.img 3.10.5-1.el6.elrepo.x86_64
```
### 参考
[Use dracut in Ubuntu to generate minimal initramfs](https://www.pcsuggest.com/dracut-linux-ubuntu/)
