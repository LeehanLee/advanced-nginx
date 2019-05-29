## 目录 
* [proxy_bind](#proxy_bind)
* [proxy_pass](#proxy_pass)

# proxy_bind
```
Syntax:	proxy_bind address [transparent] | off;
Default:	—
Context:	http, server, location
```

# proxy_pass
```
Syntax:	proxy_pass URL;
Default:	—
Context:	location, if in location, limit_except
```
设置一个代理协议和地址

协议可以是http或者https，地址可以带端口和uri
```
  server {
    listen 80;
    location / {
      proxy_pass http://127.0.0.1:81;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abc，将会proxy_pass到81端口，输出/abc

```
  server {
    listen 80;
    location /abc {
      proxy_pass http://127.0.0.1:81/xy;
    }
  }
  server {
    listen 81;
    location / {
      return 200 $request_uri;
    }
  }
```
请求80端口uri为/abcd，将会proxy_pass到81端口，输出/xyd，就是实际请求的uri(/abcd)，与location的/abc做差集，结果为d,将d拼上proxy_pass的uri(/xy)，最后代理过去的uri为/abcd
