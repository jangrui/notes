# Nginx模块

## 1. http_stub_status

编译选项:

--with-http_stub_status_modules

作用: Nginx 的客户端状态,用于监控 Nginx 当前的连接信息.

|Syntax|Defaults|Context|
|-|-|-|
|stub_status|-|server, location|

```bash
server {
    listen       80;
    server_name  localhost;
    location /mystatus {
        stub_status;
    }
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 2. http_random_index_module

编译选项:

--with-http_random_index_module

作用: 目录中选择一个随机主页

|Syntax|Defaults|Context|
|-|-|-|
|random_index on|random_index off|location|
|random_index off|||

```bash
mkdir -p /opt/app/code;cd /opt/app/code;touch 1.html 2.html 3.html .4.html
echo 1111 1.html
echo 2222 2.html
echo 3333 3.html
echo 4444 .4.html
vim /etc/nginx/conf.d/defaults.conf


server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        random_index on;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 3. http_sub_module

编译选项:

--with-http_sub_module

作用: HTTP内容替换

|Syntax|Defaults|Context|
|-|-|-|
|sub_filter string replacement|-|http, server, location|
|sub_filter_last_modified (on, off)|sub_filter_last_modified off|http, server, location|
|sub_filter_once (or, off)|sub_filter_once on|http, server, location

```bash
cat /opt/app/code/submodule.html
<html>
<head>
    <meta charset="utf-8">
    <title>submodule</title>
</head>
<body>
    <a>wwww</a>
    <a>wwwww</a>
    <a>aaaaa</a>
</body>
</html>

vim /etc/nginx/conf.d/default.conf

server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        index  index.html index.htm;
        sub_filter '<a>www' '<a>jjj';
        sub_filter_once off;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 4. Nginx 的请求限制

- 连接频率限制 - limit_conn_module
- 请求频率限制 - limit_req_module

> HTTP请求建立在一次TCP连接基础上,一次TCP请求至少产生一次HTTP请求

### 连接限制

|Syntax|Defaults|Context|
|-|-|-|
|limit_conn_zone key zone=name:size|-|http|
|limit_conn zone number|-|http, server, location|

```bash
    limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
    limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        limit_conn conn_zone 1;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 请求限制

|Syntax|Defaults|Context|
|-|-|-|
|limit_req_zone key zone=name:size rate=rate|-|http|
|limit_req zone=name [burst=number] [nodelay]|-|http, server,location|

```bash
    limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        limit_req zone=req_zone burst=3 nodelay;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

> `ab -n 20 -c 20 http://yourdomain.com/index.html`

## 5. Nginx 访问控制

> 基于IP的访问控制 - http_access_module
>
> 基于用户的信任登录 - http_auth_basic_module

### 5.1 http_access_module

|Syntax|Defaults|Context|
|-|-|-|
|allow (address, CIDR, unix:, all)|-|http, server, location, limit_except|

```bash
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        index  index.html index.htm;
    }
    location ~ ^/admin.html {
        root   /opt/app/code;
        deny all;
        allow 113.88.211.101;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

|Syntax|Defaults|Context|
|-|-|-|
|deny (address, CIDR, unix:, all)|-|http, server, location, limit_except|

```bash
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /opt/app/code;
        index  index.html index.htm;
    }
    location ~ ^/admin.html {
        root   /opt/app/code;
        deny all;
        allow 113.88.211.101;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

> http_access_module:

User_IP >> 中间件(Nginx|7lay_LSB|CDN) >> Server_IP

> http_x_forwarded_for:

Client_IP >> remote_addr ( = Client_IP ) >> Server_IP ( = remote_addr )

Client_IP >> x_forwarded_for ( = Client_IP ) >> Server_IP ( = x_forwarded_for)

> http_x_forwarded_for = Client IP ,Proxy(1) IP, Proxy(2) IP, ...
> 解决 http_access_module 局限性

方法一: 采用别的HTTP头信息控制访问,如:HTTP_X_FORWARDED_FOR

方法二: 结合 geo 模块

方法三: 通过HTTP自定义变量传递

### 5.2 http_auth_basic_module

|Syntax|Defaults|Context|
|-|-|-|
|auth_basic (string, off)|auth_basic off|http, server, location, limit_except|
|auth_basic_user_file file|-|http, server, location, limit_except|

局限性:

1. 用户信息依赖文件
2. 操作管理效率低

解决方法:

1. Nginx 结合 LUA 实现高效验证
2. Nginx 和 LDAP ,利用 Nginx + Auth + Ldap模块
