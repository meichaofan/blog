---
title: lua-resty-auto-ssl
date: 2020-10-21 00:03:44
tags:
---

## 基于lua-resty-auto-ssl插件实现自动部署、更新SSL证书

github地址 `https://github.com/auto-ssl/lua-resty-auto-ssl`

### 1、安装Openresty

```shell
curl -so /etc/yum.repos.d/openresty.repo https://openresty.org/package/centos/openresty.repo

yum install -y openresty gcc make diffutils openssl
```

OpenResty所有的文件以及依赖包都安装在 `/usr/local/openresty`目录下。



### 2、安装配置 lua-resty-auto-ssl

#### 2.1 安装Luarocks

Luarocks是Lua的包管理工具，很多OpenResty的包都可以通过luarocks来安装。

```shell
mkdir /root/package && cd /root/package

wget http://luarocks.github.io/luarocks/releases/luarocks-3.3.1.tar.gz

tar zxvf luarocks-3.3.1.tar.gz

cd luarocks-3.3.1 

./configure --prefix=/usr/local/openresty/luajit/ \
    --with-lua=/usr/local/openresty/luajit/ \
    --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1/ 

make build && make install 

ln -s /usr/local/openresty/luajit/bin/luarocks /usr/local/bin/
```

#### 2.2 安装lua-resty-auto-ssl

```shell
luarocks install lua-resty-auto-ssl
```



### 3 配置lua-resty-auto-ssl

在`/usr/local/openresty/nginx/conf`目录下新建 `mkdir ssl`(存放ssl证书) 和`mkdir vhost-services`(存放server配置文件)

编辑配置文件 `vim /usr/local/openresty/nginx/conf/nginx.conf`，内容如下：

```nginx
user root; 
worker_processes  1;

events {
	worker_connections  1024;
}

http {
	include       mime.types;
	default_type  application/octet-stream;

	#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	#                  '$status $body_bytes_sent "$http_referer" '
	#                  '"$http_user_agent" "$http_x_forwarded_for"';

	#access_log  logs/access.log  main;
	sendfile        on;
    #tcp_nopush     on;
	#keepalive_timeout  0;
	keepalive_timeout  65;

    
	lua_shared_dict auto_ssl 1m;
	lua_shared_dict auto_ssl_settings 64k;
	resolver 8.8.8.8;
	init_by_lua_block {
		auto_ssl = (require "resty.auto-ssl").new()
		auto_ssl:set("allow_domain", function(domain)
			return ngx.re.match(domain, "huany.top$", "ijo")
		end)
		auto_ssl:set("dir", "/usr/local/openresty/nginx/conf/ssl")
		auto_ssl:init()
	}
	init_worker_by_lua_block {
	auto_ssl:init_worker()
	}
	# HTTP server
	#
	server {
		listen      80;
		location / {
			return 301 https://$host$request_uri;
		}
		location /.well-known/acme-challenge/ {
			content_by_lua_block {
				auto_ssl:challenge_server()
			}
		}
	}
	# Inner Request   
	server {
		listen 127.0.0.1:8999;
		client_body_buffer_size 128k;
		client_max_body_size 128k;
		location / {
		content_by_lua_block {
			auto_ssl:hook_server()
			}
		}
	}
	include vhost-services/*.conf;
}
```

**注意:**

#### 3.1 只允许为 huany.top 结尾的域名生成证书

```lua
auto_ssl:set("allow_domain", function(domain)
			return ngx.re.match(domain, "huany.top$", "ijo")
end)
```

#### 3.2 生成的证书存放目录

```lua
auto_ssl:set("dir", "/usr/local/openresty/nginx/conf/ssl")
```

#### 3.3 Let's Encrypt 请求开发的80端口

```nginx
server {
	listen      80;
	# let's encrypt 在验签的过程中，会请求这个地址，所以必须写
    location /.well-known/acme-challenge/ {
		content_by_lua_block {
			auto_ssl:challenge_server()
		}
	}
}
```

#### 3.4  Let's Encrypt 要求开发的内部端口 （8999可以更改）

```lua
server {
	listen 127.0.0.1:8999;
	client_body_buffer_size 128k;
	client_max_body_size 128k;
	location / {
		content_by_lua_block {
			auto_ssl:hook_server()
		}
	}
}
```

#### 3.5 强制http 跳转 https

```nginx
server {
    listen	80;
    location / {
        return 301 https://$host$request_uri;
    }
}
```



### 4 配置Server

先看一下`/usr/local/openresty/nginx/conf`下的目录结构 （其中`ssl`和`vhost-services`是我们提前新建好的）

```
[root@rancher conf]# ll
total 76
-rw-r--r-- 1 root root 1077 Jul 14 02:44 fastcgi.conf
-rw-r--r-- 1 root root 1077 Jul 14 02:44 fastcgi.conf.default
-rw-r--r-- 1 root root 1007 Jul 14 02:44 fastcgi_params
-rw-r--r-- 1 root root 1007 Jul 14 02:44 fastcgi_params.default
-rw-r--r-- 1 root root 2837 Jul 14 02:44 koi-utf
-rw-r--r-- 1 root root 2223 Jul 14 02:44 koi-win
-rw-r--r-- 1 root root 5231 Jul 14 02:44 mime.types
-rw-r--r-- 1 root root 5231 Jul 14 02:44 mime.types.default
-rw-r--r-- 1 root root  926 Oct 19 15:12 nginx.conf
-rw-r--r-- 1 root root 2656 Jul 14 02:44 nginx.conf.default
-rw-r--r-- 1 root root  636 Jul 14 02:44 scgi_params
-rw-r--r-- 1 root root  636 Jul 14 02:44 scgi_params.default
drwxr-xr-x 5 root root 4096 Oct 19 15:01 ssl
-rw-r--r-- 1 root root  664 Jul 14 02:44 uwsgi_params
-rw-r--r-- 1 root root  664 Jul 14 02:44 uwsgi_params.default
drwxr-xr-x 2 root root 4096 Oct 19 15:13 vhost-services
-rw-r--r-- 1 root root 3610 Jul 14 02:44 win-utf
```

我们的server配置在`vhost-services`目录中，`www.conf`的内容如下：

```nginx
server {
	listen       443 ssl;
	server_name	 www.huany.top;
	ssl_certificate_by_lua_block {
		auto_ssl:ssl_certificate()
	}

	ssl_certificate ssl/resty-auto-ssl-fallback.crt;
	ssl_certificate_key ssl/resty-auto-ssl-fallback.key;

	ssl_session_cache    shared:SSL:1m;
	ssl_session_timeout  5m;

	ssl_ciphers  HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers  on;


	location / {
		root   /root/web/www;
		index  index.html index.htm;
	}
}
```

`test.conf`内容如下：

```nginx
server {
	listen       443 ssl;
	server_name	 test.huany.top;
	ssl_certificate_by_lua_block {
		auto_ssl:ssl_certificate()
	}
	
	ssl_certificate ssl/resty-auto-ssl-fallback.crt;
	ssl_certificate_key ssl/resty-auto-ssl-fallback.key;

	ssl_session_cache    shared:SSL:1m;
	ssl_session_timeout  5m;

	ssl_ciphers  HIGH:!aNULL:!MD5;
	ssl_prefer_server_ciphers  on;

	location / {
		root   /root/web/test;
		index  index.html index.htm;
	}
}
```

**注意:**

#### 4.1 需要先配置默认证书，让`openrety`正常启动

```shell
openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 \
-subj '/CN=sni-support-required-for-valid-ssl' \
-keyout /usr/local/openresty/nginx/conf/ssl/resty-auto-ssl-fallback.key \
-out /usr/local/openresty/nginx/conf/ssl/resty-auto-ssl-fallback.crt
```

下面两行配置证书

```nginx
ssl_certificate ssl/resty-auto-ssl-fallback.crt;
ssl_certificate_key ssl/resty-auto-ssl-fallback.key;
```
