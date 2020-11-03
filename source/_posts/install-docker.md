---
title: 安装Docker CE
date: 2019-04-20 21:41:10
tags:
- docker
---

[官网安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### 卸载老版本
老版本的Docker，以前叫做`docker`,`docker.io`和`docker-engine`，如果系统里安装了它们，就先卸载掉
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 支持的存储系统
Docker CE在Ubuntu上支持`overlay2`，`aufs`,和`btrfs`文件系统。在Linux内核4.0或更高的内核版本上，默认使用`overlay2`文件系统，它的性能能高于`aufs`。如果非要使用`aufs`，请见[配置Docker CE使用aufs文件系统]()。

### 安装Docker CE

#### 1.使用apt repository

**添加仓库**
```
# 更新 `apt`
$ sudo apt-get update

# 安装一些必要的软件
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# 下载Docker官方GPG key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 检验指纹 `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`
$ sudo apt-key fingerprint 0EBFCD88
    
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

# 添加仓库
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

**安装Docker CE**
```
$ sudo apt-get update

# 默认安装最新版本
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

# 可以安装指定版本
# 1.查看可用版本
$ apt-cache madison docker-ce

  docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  ...

# 2.安装指定版本
$ sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

#### 2.使用deb包安装

1.去 https://download.docker.com/linux/ubuntu/dists/ 网站，选择合适的`.deb`文件
2.安装`.deb`包
```
$ sudo dpkg -i /path/to/package.deb
```

#### 3.使用shell脚本安装
1. 脚本在 get.docker.com
2. 不建议在生产环境直接使用脚本安装docker
3. 安装步骤
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh

<output truncated>
```

### 添加非root用户到docker组
```
$ sudo usermod -aG docker your-user
```

### 卸载Docker CE
1.卸载Docker CE
```
$ sudo apt-get purge docker-ce
```
2.镜像、数据卷、容器和一些自定义的配置文件不会自动删除，需要手动删除
```
$ sudo rm -rf /var/lib/docker
```
