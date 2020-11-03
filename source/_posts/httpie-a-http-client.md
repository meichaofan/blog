---
title: HTTPie - 一款http客户端 
date: 2019-04-23 14:22:03
tags:
- http
- linux
---

一款http客户端

---

### 主要特点
* 具表达力和直观语法
* 格式化的及彩色化的终端输出
* 内置JSON支持
* 表单和文件上传
* HTTPS、代理和认证
* 任意请求数据
* 自定义头部
* 持久化会话
* 类似于wget的下载
* 支持Python2.7 和 3.x

### 在Linux下安装Httpie

1. Debian/Ubuntu系统，使用`apt-get`或`apt`
```
$ sudo apt install httpie
```

2. RHEL/CentOs系统，使用`yum`安装
```
$ sudo yum install httpie
```

### 用法

1. 如何使用HTTPie请求URL？
httpie的基本用法是将URL作为参数
```
meichaofan@ubuntu1:~$ http 2daygeek.com
HTTP/1.1 301 Moved Permanently
CF-RAY: 4cbdc9a7b85322a0-LAX
Cache-Control: max-age=3600
Connection: keep-alive
Date: Tue, 23 Apr 2019 06:30:15 GMT
Expires: Tue, 23 Apr 2019 07:30:15 GMT
Location: https://2daygeek.com/
Server: cloudflare
Transfer-Encoding: chunked
Vary: Accept-Encoding

```

2. 如何使用HTTPie下载文件
带 `--download` 参数，用来下载文件，类似于wget
```
meichaofan@ubuntu1:~$ http --download https://www.2daygeek.com/wp-content/uploads/2019/04/Anbox-Easy-Way-To-Run-Android-Apps-On-Linux.png
HTTP/1.1 200 OK
Accept-Ranges: bytes
CF-Cache-Status: HIT
CF-RAY: 4cbdcfbafb53540e-LAX
Cache-Control: public, max-age=7200
Connection: keep-alive
Content-Length: 32066
Content-Type: image/png
Date: Tue, 23 Apr 2019 06:34:23 GMT
Expect-CT: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
Expires: Tue, 23 Apr 2019 08:34:23 GMT
Last-Modified: Mon, 08 Apr 2019 04:54:25 GMT
Server: cloudflare
Set-Cookie: __cfduid=dee8965dcfafc633c5463965e0c40522d1556001263; expires=Wed, 22-Apr-20 06:34:23 GMT; path=/; domain=.2daygeek.com; HttpOnly; Secure
Vary: Accept-Encoding

Downloading 31.31 kB to "Anbox-Easy-Way-To-Run-Android-Apps-On-Linux.png"
Done. 31.31 kB in 0.55747s (56.17 kB/s)
meichaofan@ubuntu1:~$ ls
Anbox-Easy-Way-To-Run-Android-Apps-On-Linux.png  package  peek-a-boo
```
 你还可以使用 `-o` 参数用不同的名称保存输出文件。
```
meichaofan@ubuntu1:~$ http --download https://www.2daygeek.com/wp-content/uploads/2019/04/Anbox-Easy-Way-To-Run-Android-Apps-On-Linux.png -o Anbox-1.png
HTTP/1.1 200 OK
Accept-Ranges: bytes
CF-Cache-Status: HIT
CF-RAY: 4cbdd1493c0e5047-LAX
Cache-Control: public, max-age=7200
Connection: keep-alive
Content-Length: 32066
Content-Type: image/png
Date: Tue, 23 Apr 2019 06:35:27 GMT
Expect-CT: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
Expires: Tue, 23 Apr 2019 08:35:27 GMT
Last-Modified: Mon, 08 Apr 2019 04:54:25 GMT
Server: cloudflare
Set-Cookie: __cfduid=d4ac029dc64db0797db8e607d6a8568341556001327; expires=Wed, 22-Apr-20 06:35:27 GMT; path=/; domain=.2daygeek.com; HttpOnly; Secure
Vary: Accept-Encoding

Downloading 31.31 kB to "Anbox-1.png"
Done. 31.31 kB in 0.28450s (110.07 kB/s)
meichaofan@ubuntu1:~$ ls
Anbox-1.png  package  peek-a-boo
```

3. 如何使用 HTTPie 恢复部分下载？
可以使用 `-c` 参数的HTTPie继续下载
```
http --download --continue https://speed.hetzner.de/100MB.bin -o 100MB.bin
```

4. 如何使用 HTTPie 上传文件？
你可以通过使用带有小于号 `<` 的 HTTPie 命令上传文件
```
$ http https://transfer.sh < Anbox-1.png
```

5. 如何使用带有重定向符号 `>` 下载文件？
你可以使用带有重定向 `>` 符号的 HTTPie 命令下载文件。
```
$ http https://www.2daygeek.com/wp-content/uploads/2019/03/How-To-Install-And-Enable-Flatpak-Support-On-Linux-1.png > Flatpak.png
$ ls -ltrh Flatpak.png
-rw-r--r-- 1 root root 47K Apr  9 01:44 Flatpak.png
```

6. 发送一个 HTTP GET 请求？
您可以在请求中发送 HTTP GET 方法。GET 方法会使用给定的 URI，从给定服务器检索信息。
```
http GET httpie.org
HTTP/1.1 301 Moved Permanently
CF-RAY: 4c4a83a3f90dcbe6-SIN
Cache-Control: max-age=3600
Connection: keep-alive
Date: Tue, 09 Apr 2019 06:44:44 GMT
Expires: Tue, 09 Apr 2019 07:44:44 GMT
Location: https://httpie.org/
Server: cloudflare
Transfer-Encoding: chunked
Vary: Accept-Encoding
```

7. 提交表单
使用以下格式提交表单。POST 请求用于向服务器发送数据，例如客户信息、文件上传等。要使用 HTML 表单。
```
http -f POST Ubuntu18.2daygeek.com hello='World'
HTTP/1.1 200 OK
Accept-Ranges: bytes
Connection: Keep-Alive
Content-Encoding: gzip
Content-Length: 3138
Content-Type: text/html
Date: Tue, 09 Apr 2019 06:48:12 GMT
ETag: "2aa6-5844bf1b047fc-gzip"
Keep-Alive: timeout=5, max=100
Last-Modified: Sun, 17 Mar 2019 15:29:55 GMT
Server: Apache/2.4.29 (Ubuntu)
Vary: Accept-Encoding
```
 运行下面的指令以查看正在发送的请求。
```
http -v Ubuntu18.2daygeek.com
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: ubuntu18.2daygeek.com
User-Agent: HTTPie/0.9.8
hello=World
HTTP/1.1 200 OK
Accept-Ranges: bytes
Connection: Keep-Alive
Content-Encoding: gzip
Content-Length: 3138
Content-Type: text/html
Date: Tue, 09 Apr 2019 06:48:30 GMT
ETag: "2aa6-5844bf1b047fc-gzip"
Keep-Alive: timeout=5, max=100
Last-Modified: Sun, 17 Mar 2019 15:29:55 GMT
Server: Apache/2.4.29 (Ubuntu)
Vary: Accept-Encoding
```
