advanced-nginx
=======================
涉及
- nginx编译与安装
- nginx各模块基本操作与总结
- openssl私钥、CSR、CRT、自签CA的搭建


联系作者
- weixin: 363260961

使用版本
- CentOS Linux release 7.5.1804 (Core)
- nginx/1.12.2版本
- curl 7.61.0 (x86_64-pc-linux-gnu) libcurl/7.61.0 OpenSSL/1.0.2k zlib/1.2.7

## 目录
* [nginx如何处理一个请求](#nginx如何处理一个请求)


# nginx如何处理一个请求

nginx首先选定由那一个虚拟主机来处理请求
```nginx
server {
    listen 80;
    server_name a.com b.com;
    location / {
        return 200 "server_name is a.com b.com";
    }
}
server {
    listen 80;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
在这个配置中，nginx**仅仅**检查请求的host头以决定该请求由那个虚拟主机来处理，如果host头没有匹配任意一个虚拟主机，或者请求中根本没有包含host头，那nginx会将请求分发到定义在此端口上的默认虚拟主机，第一个被列出的虚拟主机就是nginx的默认虚拟主机。也可以显式的某个虚拟主机为默认虚拟主机(default_server从0.8.21开始使用，在以前版本使用default代替)
```curl
    curl 'http://127.0.0.1' -H 'host: a.com'
```
以上请求的host被指定为a.com，所以匹配到server_name a.com b.com，返回http_code 200，消息体server_name is a.com b.com
```curl
    curl 'http://127.0.0.1' -H 'host: c.com'
```
以上请求的host被指定为c.com，所以匹配到server_name c.com d.com，返回http_code 500，消息体server_name is c.com d.com
```curl
    curl 'http://127.0.0.1' -H 'host: xxx.com'
```
以上请求的host被指定为xxx.com，没有server_name与其匹配，所以nginx会将请求分发到定义在此端口上的默认虚拟主机(第一个被列出的)，返回http_code 200，消息体server_name is a.com b.com
如果将nginx配置改为
```nginx
server {
    listen 80;
    server_name a.com b.com;
    location / {
        return 200 "server_name is a.com b.com";
    }
}
server {
    listen 80 default_server;
    server_name c.com d.com;
    location / {
        return 500 "server_name is c.com d.com";
    }
}
```
```curl
    curl 'http://127.0.0.1' -H 'host: xxx.com'
```
以上请求的host被指定为xxx.com，没有server_name与其匹配，所以nginx会将请求分发到定义在此端口上的默认虚拟主机(default_server定义的)，返回http_code 500，消息体server_name is c.com d.com
