---
title: 利用docker搭建samba目录共享 
date: 2019-04-22 01:20:04
tags:
- docker
- samba
---

1.下载samba镜像
```
docker pull dperson/samba
```

2.启动镜像，具体配置看文档，但重要的配置是一下的注释
```
docker run --name samba \
-it -p 139:139 -p 445:445 \
-v /home/meichaofan:/home/meichaofan \
-v /etc/passwd:/etc/passwd \
-v /etc/group:/etc/group \
-d dperson/samba \
-u "meichaofan;huanhuan0921" \
-s "meichaofan home;/home/meichaofan;yes;no;no;all;none"
```

3.替换samba的启动用户，与权限相关
```
docker exec -it samba sed -i 's/force user = smbuser/force user = meichaofan/g' /etc/samba/smb.conf
```

3.替换samba的启动组，与权限相关
```
docker exec -it samba sed -i 's/force group = users/force group = meichaofan/g' /etc/samba/smb.conf
```

4.重启samba
```
docker restart samba
```
