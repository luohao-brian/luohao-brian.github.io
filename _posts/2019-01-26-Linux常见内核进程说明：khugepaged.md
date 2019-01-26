---
layout: post
title: Linux常见内核进程说明 - khugepaged
tags: [Linux]
---

开启THP(Transparent Huge Page)系统会产生一个khugepaged进程，他是THP的后台守护进程，主要功能是定时唤醒，根据配置尝试将4k 的普通page转成2M等巨页，减少TLB压力，提高内存使用效率。

关闭该功能： 

1. kernel的启动参数可以通过传入：
```
transparent_hugepage=never
```

2. 系统启动后:
```
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

khugepaged 还有两个重要的配置参数： 
1. `/sys/kernel/mm/transparent_hugepage/khugepaged/pages_to_scan`： 配置khugepaged 后台进程一次处理的页面 


2. `/sys/kernel/mm/transparent_hugepage/khugepaged/scan_sleep_millisecs`：
配置khugepaged 后台进程唤醒的时间间隔 
