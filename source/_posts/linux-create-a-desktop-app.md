---
title: Linux如何给应用程序创建一个桌面启动图标 
date: 2019-04-29 10:04:29
tags:
- Linux
- Desktop
---

本文主要讲述的是linux中如何给应用程序创建一个快速启动图标，话不多说，我们来看实际的操作步骤：

本文的实例是给celipse创建一个启动图标

---

1. 我们需要通过下列命令，来创建一个启动的脚本：
```
vim /usr/share/applications/eclipse.desktop
```

2. 将下列内容复制到启动脚本中：
```
[Desktop Entry]
Encoding=UTF-8
Name=Eclipse
Comment=Eclipse IDE
Exec=/usr/local/android/eclipse/eclipse      
Icon=/usr/local/android/eclipse/icon.xpm
Terminal=false
StartupNotify=true
Type=Application
Categories=Application;Development;
```
>说明部分：
Exec ：这个是应用程序可执行文件的目录
Icon ：这个是图标的目录

3. 然后在applications -> programming 菜单中就可以找到你所创建的图标了，然后再右键创建一个桌面图标即可




