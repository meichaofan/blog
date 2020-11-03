---
title: Docker 存储驱动之 overlay2 
date: 2019-04-21 01:05:59
tags:
- docker
- overlay2
---

### overlay2中镜像和容器的磁盘结构

docker pull ubuntu:14.04下载了包含4层的镜像，
```
[root@www ~]# docker pull ubuntu:14.04
14.04: Pulling from library/ubuntu
e082d4499130: Pull complete 
371450624c9e: Pull complete 
c8a555b3a57c: Pull complete 
1456d810d42e: Pull complete 
Digest: sha256:6612de24437f6f01d6a2988ed9a36b3603df06e8d2c0493678f3ee696bc4bb2d
Status: Downloaded newer image for ubuntu:14.04
```

可以在/var/lib/docker/overlay2中看到，有5个目录。
```
[root@www overlay2]# ll -h
total 28K
drwx------ 4 root root 4.0K Apr 21 00:32 0734cd8060d194cf3db93162b421e4f245151920f0b9acda5313ec9f671bc5cc
drwx------ 3 root root 4.0K Apr 21 00:32 153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3
drwx------ 4 root root 4.0K Apr 21 00:32 658b3e84761843f58ecaf82e1f987bf32d498b7bd54a9cf40bf6bf635fff8ac3
drwx------ 4 root root 4.0K Apr 21 00:32 9ff7c96e688b25a7353c83904d791eae357c2e16315ecbe602009678965ce761
drwx------ 2 root root 4.0K Apr 21 00:33 l
```

### **l**目录的内容

其中**l**目录中包含了很多软连接，使用短名称指向了其它层。短名称用于避免mount参数时达到页面大小的限制。
```
[root@www overlay2]# ll l/
total 24
lrwxrwxrwx 1 root root 72 Apr 21 00:32 2AQ2F7X67IL4TATXIIYO62CM67 -> ../153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3/diff
lrwxrwxrwx 1 root root 72 Apr 21 00:32 HMOMCX3OAMM4K6KUZORQ7NTIG4 -> ../9ff7c96e688b25a7353c83904d791eae357c2e16315ecbe602009678965ce761/diff
lrwxrwxrwx 1 root root 72 Apr 21 00:32 JZPHZPG7ZLX7GTKCFVHQZDEF43 -> ../658b3e84761843f58ecaf82e1f987bf32d498b7bd54a9cf40bf6bf635fff8ac3/diff
lrwxrwxrwx 1 root root 72 Apr 21 00:32 OJ5PMEVISSBQ4X4N2U66YEE6IH -> ../0734cd8060d194cf3db93162b421e4f245151920f0b9acda5313ec9f671bc5cc/diff
```

### mount 查看容器/镜像的层次关系

**现在起一个容器，它会在rootfs层上，加上init层（环境相关）和容器层（读写）**
```
docker run -it ubuntu:14.04 /bin/bash
```

查看mount相关信息，可以看出层级关系
```
[root@www ~]# mount -l | grep overlay2
/dev/vda1 on /var/lib/docker/overlay2 type ext3 (rw,relatime,data=ordered)
overlay on /var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/merged type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/MXQORLOJXLIRFVZ3R26TNWWURM:/var/lib/docker/overlay2/l/HMOMCX3OAMM4K6KUZORQ7NTIG4:/var/lib/docker/overlay2/l/OJ5PMEVISSBQ4X4N2U66YEE6IH:/var/lib/docker/overlay2/l/JZPHZPG7ZLX7GTKCFVHQZDEF43:/var/lib/docker/overlay2/l/2AQ2F7X67IL4TATXIIYO62CM67,upperdir=/var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/diff,workdir=/var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/work)
```
整理mount信息,如下：
```
merged:
    /var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/merged (联合挂载到此目录下)

workdir:
    /var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/work 

upperdir:
    /var/lib/docker/overlay2/bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15/diff (第六层 rw)

lowerdir:
    /var/lib/docker/overlay2/l/MXQORLOJXLIRFVZ3R26TNWWURM (第5层 init层 ro)
    /var/lib/docker/overlay2/l/HMOMCX3OAMM4K6KUZORQ7NTIG4 (第四层 ro)
    /var/lib/docker/overlay2/l/OJ5PMEVISSBQ4X4N2U66YEE6IH (第三层 ro)
    /var/lib/docker/overlay2/l/JZPHZPG7ZLX7GTKCFVHQZDEF43 (第二层 ro)
    /var/lib/docker/overlay2/l/2AQ2F7X67IL4TATXIIYO62CM67 (rootfs 第一层 ro)
```

### 各个rootfs层文件内容介绍

查看rootfs第一层的目录信息
```
[root@www ~]# ll /var/lib/docker/overlay2/l/2AQ2F7X67IL4TATXIIYO62CM67
lrwxrwxrwx 1 root root 72 Apr 21 00:32 /var/lib/docker/overlay2/l/2AQ2F7X67IL4TATXIIYO62CM67 -> ../153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3/diff

[root@www ~]# ll /var/lib/docker/overlay2/153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3/
total 8
drwxr-xr-x 21 root root 4096 Apr 21 00:32 diff
-rw-r--r--  1 root root   26 Apr 21 00:32 link
```
可以看到，最低层只有两个文件，一个是diff，存放当前层的文件和目录，link则和`l`目录的软链接向对应
```
[root@www ~]# ll /var/lib/docker/overlay2/153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3/diff
total 76
drwxr-xr-x  2 root root 4096 Mar  5 12:46 bin
drwxr-xr-x  2 root root 4096 Apr 11  2014 boot
drwxr-xr-x  3 root root 4096 Mar  5 12:45 dev
drwxr-xr-x 61 root root 4096 Mar  5 12:46 etc
drwxr-xr-x  2 root root 4096 Apr 11  2014 home
drwxr-xr-x 12 root root 4096 Mar  5 12:46 lib
drwxr-xr-x  2 root root 4096 Mar  5 12:45 lib64
drwxr-xr-x  2 root root 4096 Mar  5 12:45 media
drwxr-xr-x  2 root root 4096 Apr 11  2014 mnt
drwxr-xr-x  2 root root 4096 Mar  5 12:45 opt
drwxr-xr-x  2 root root 4096 Apr 11  2014 proc
drwx------  2 root root 4096 Mar  5 12:46 root
drwxr-xr-x  7 root root 4096 Mar  5 12:46 run
drwxr-xr-x  2 root root 4096 Mar  5 12:46 sbin
drwxr-xr-x  2 root root 4096 Mar  5 12:45 srv
drwxr-xr-x  2 root root 4096 Mar 13  2014 sys
drwxrwxrwt  2 root root 4096 Mar  5 12:46 tmp
drwxr-xr-x 10 root root 4096 Mar  5 12:45 usr
drwxr-xr-x 11 root root 4096 Mar  5 12:46 var

[root@www ~]# cat /var/lib/docker/overlay2/153f0c95b638414d41bb07b9d45243a0219719e468dd9dcb097b80f837b4d8b3/link 
2AQ2F7X67IL4TATXIIYO62CM67
```

查看rootfs第二层信息
```
[root@www ~]# ll /var/lib/docker/overlay2/l/JZPHZPG7ZLX7GTKCFVHQZDEF43
lrwxrwxrwx 1 root root 72 Apr 21 00:32 /var/lib/docker/overlay2/l/JZPHZPG7ZLX7GTKCFVHQZDEF43 -> ../658b3e84761843f58ecaf82e1f987bf32d498b7bd54a9cf40bf6bf635fff8ac3/diff
[root@www ~]# ll /var/lib/docker/overlay2/658b3e84761843f58ecaf82e1f987bf32d498b7bd54a9cf40bf6bf635fff8ac3
total 16
drwxr-xr-x 6 root root 4096 Apr 21 00:32 diff
-rw-r--r-- 1 root root   26 Apr 21 00:32 link
-rw-r--r-- 1 root root   28 Apr 21 00:32 lower
drwx------ 2 root root 4096 Apr 21 00:32 work
```
第二层有四个文件，diff 和 link如上所序一样。lower文件的内容是当前层下面的rootfs的软连接名
```
# 第二层rootfs的lower内容
[root@www ~]# cat /var/lib/docker/overlay2/658b3e84761843f58ecaf82e1f987bf32d498b7bd54a9cf40bf6bf635fff8ac3/lower 
l/2AQ2F7X67IL4TATXIIYO62CM67

#第三层rootfs的lower内容
[root@www ~]# cat /var/lib/docker/overlay2/0734cd8060d194cf3db93162b421e4f245151920f0b9acda5313ec9f671bc5cc/lower 
l/JZPHZPG7ZLX7GTKCFVHQZDEF43:l/2AQ2F7X67IL4TATXIIYO62CM67

#第四层rootfs的lower内容
[root@www ~]# cat /var/lib/docker/overlay2/9ff7c96e688b25a7353c83904d791eae357c2e16315ecbe602009678965ce761/lower 
l/OJ5PMEVISSBQ4X4N2U66YEE6IH:l/JZPHZPG7ZLX7GTKCFVHQZDEF43:l/2AQ2F7X67IL4TATXIIYO62CM67
```

最顶层，也就是upperdir层，看一下它的文件目录
```
[root@www bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15]# ls
diff  link  lower  merged  work
```
upperdir是容器层，是可读写的。在容器中所有修改文件操作最后都在upperdir的`diff`目录体现，并合并到`merged`目录下。`merged`目录是联合后挂载的目录，也是容器的文件系统。
```
# 假如我docker run -it ubuntu:14.04 /bin/bash 启动了一个容器，然后再其/(根目录)下创建一个Test目录,并在/root目录下新建了一个aa文件，并删除了/bin/ss文件。我们现在看一下upperdir的`diff`目录

[root@www bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15]# tree diff/
diff/
├── bin
│   └── ss
├── root
│   └── aa
└── Test

3 directories, 2 files

# 在看一个merged目录，它是叠加后一个完整的文件系统目录结构
[root@www bda1eaf86f4c4a500aac2418bc45956531e81de6fff829b67452174c891f5f15]# ls merged/
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  Test  tmp  usr  var
```

### OverlayFS constructs 

OverlayFS将单个Linux主机上的两个目录合并成一个目录。这些目录被称为层，统一过程被称为联合挂载。OverlayFS底层目录称为lowerdir， 高层目录称为upperdir。合并统一视图称为merged。当需要修改一个文件时，使用CoW将文件从只读的Lower复制到可写的Upper进行修改，结果也保存在Upper层。在Docker中，底下的只读层就是image，可写层就是Container

下图分层图，镜像层是lowdir，容器层是upperdir，统一的视图层是merged层

<div align="center"><img src="https://github.com/meichaofan/static-file/blob/master/docker/overlay_constructs.jpg?raw=true" width = 50% height = 50% /></div>
<div align="center">OverlayFs constructs</div>

