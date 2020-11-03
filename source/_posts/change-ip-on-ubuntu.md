---
title: ubuntu系统修改网卡ip
date: 2019-04-22 17:49:25
tags:
- linux
- ubuntu
---

需求：将ubuntu系统`ens33`网卡ip修改为`192.168.244.140`

---

1.编辑`/etc/network/interfaces`文件

```
meichaofan@ubuntu:~$ cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

# 新增内容
auto ens33
iface ens33 inet static
address 192.168.244.140
netmask 255.255.255.0
gateway 192.168.244.2 
```

2.重启网卡

```
sudo /etc/init.d/networking restart
```
