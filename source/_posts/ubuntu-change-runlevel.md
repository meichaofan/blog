---
title: Ubuntu 系统修改默认运行级别 
date: 2019-04-29 01:40:12
tags:
- ubuntu
- runlevel
---

## Deb系运行级别（Debian、Ubuntu）
>0 – Halt，关机模式
1 – Single，单用户模式
2 - Full multi-user with display manager (GUI)
3 - Full multi-user with display manager (GUI)
4 - Full multi-user with display manager (GUI)
5 - Full multi-user with display manager (GUI)
6 – Reboot，重启
S - 单用户恢复模式

2~5级是没有任何区别的，他们为多用户模式。

## Rpm系运行级别（Redhat、CentOS）
>0 – Halt，关机模式
1 – Single，单用户模式
2 – 多用户模式，但不能使用NFS（相当于Windows下的网上邻居）
3 – 字符界面的多用户模式
4 – Undefined
5 – Full multi-user with display manager (GUI)
6 – Reboot，重启

## 查看当前运行级别
```
meichaofan@ubuntu:~$ runlevel 
N 2
```

## ubuntu系统下修改运行级别
```
vi /etc/default/grup
将
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
改为
GRUB_CMDLINE_LINUX_DEFAULT="text"
然后执行
update-grub2
```
