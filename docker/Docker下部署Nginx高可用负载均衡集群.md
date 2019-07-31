# Docker下部署Nginx高可用负载均衡集群

下面以 [人人开源](https://www.renren.io) 前后端项目为例，创建 Nginx 高可用负载均衡集群

- [renren-fast后端项目](https://gitee.com/renrenio/renren-fast)

- [renren-fast-vue前端项目](https://github.com/daxiongYang/renren-fast-vue)

## 打包部署后端项目

- 进入人人开源后端项目，执行打包（修改配置文件，更改端口，打包三次生成三个不同端口的JAR文件）

```bash
mvn clean install -Dmaven.test.skip=true
```

- 安装Java镜像

```bash
docker pull java
```

- 创建3节点Java容器

```bash
#创建数据卷，上传JAR文件
docker volume create j1
#启动容器
docker run -it -d --name j1 -v j1:/home/soft --net=host java
#进入j1容器
docker exec -it j1 bash
#启动Java项目
nohup java -jar /home/soft/renren-fast.jar

#创建数据卷，上传JAR文件
docker volume create j2
#启动容器
docker run -it -d --name j2 -v j2:/home/soft --net=host java
#进入j1容器
docker exec -it j2 bash
#启动Java项目
nohup java -jar /home/soft/renren-fast.jar

#创建数据卷，上传JAR文件
docker volume create j3
#启动容器
docker run -it -d --name j3 -v j3:/home/soft --net=host java
#进入j1容器
docker exec -it j3 bash
#启动Java项目
nohup java -jar /home/soft/renren-fast.jar
```

- 安装Nginx镜像

```bash
docker pull nginx
```

- 创建Nginx容器，配置负载均衡

宿主机上/home/n1/nginx.conf配置文件内容如下：

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
   }

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    upstream tomcat {
        server localhost:6001;
        server localhost:6002;
        server localhost:6003;
    }
    server {
        listen       6101;
        server_name  localhost;
        location / {  
            proxy_pass   http://tomcat;
            index  index.html index.htm;  
        }  
    }
}
```

宿主机上/home/n1/keepalived.conf配置文件内容如下：

```bash
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        192.168.99.151
    }
}
virtual_server 192.168.99.151 6201 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    real_server 192.168.99.104 6101 {
        weight 1
    }
}
```

创建第1个Nginx节点

```bash
docker run -it -d --name n1 -v /home/n1/nginx.conf:/etc/nginx/nginx.conf -v /home/n1/keepalived.conf:/etc/keepalived/keepalived.conf --net=host --privileged nginx
```

宿主机上/home/n2/nginx.conf配置文件内容如下：

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
    }

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    upstream tomcat {
        server 192.168.99.104:6001;
        server 192.168.99.104:6002;
        server 192.168.99.104:6003;
    }
    server {
        listen       6102;
        server_name  192.168.99.104;
        location / {  
            proxy_pass   http://tomcat;
            index  index.html index.htm;  
        }  
    }
}
```

宿主机上/home/n2/keepalived.conf配置文件内容如下：

```bash
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        192.168.99.151
    }
}
virtual_server 192.168.99.151 6201 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    real_server 192.168.99.104 6102 {
        weight 1
    }
}
```

创建第2个Nginx节点

```bash
docker run -it -d --name n2 -v /home/n2/nginx.conf:/etc/nginx/nginx.conf -v /home/n2/keepalived.conf:/etc/keepalived/keepalived.conf --net=host --privileged nginx
```

- 在Nginx容器安装Keepalived

```bash
#进入n1节点
docker exec -it n1 bash
#更新软件包
apt-get update
#安装Keepalived
apt-get install keepalived
#启动Keepalived
service keepalived start

#进入n2节点
docker exec -it n2 bash
#更新软件包
apt-get update
#安装Keepalived
apt-get install keepalived
#启动Keepalived
service keepalived start
```

## 打包部署前端项目

修改前端项目连接后端api接口,执行打包指令

```bash
npm run build
```

build目录的文件拷贝到宿主机的/home/renren-vue-n1/renren-vue、/home/renren-vue-n2/renren-vue、/home/renren-vue-n3/renren-vue的目录下面

创建3节点的Nginx，部署前端项目

宿主机/home/renren-vue-n1/nginx.conf的配置文件

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    server {
        listen 6501;
        server_name  192.168.99.104;
        location  /  {
            root  /home/renren-vue-n1/renren-vue;
            index  index.html;
        }
    }
}
```

```bash
#启动第renren-vue-n1节点
docker run -it -d --name renren-vue-n1 -v /home/renren-vue-n1/nginx.conf:/etc/nginx/nginx.conf -v /home/renren-vue-n1/renren-vue:/home/renren-vue-n1/renren-vue --privileged --net=host nginx
```

宿主机/home/renren-vue-n2/nginx.conf的配置文件

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    server {
        listen 6502;
        server_name  192.168.99.104;
        location  /  {
            root  /home/renren-vue-n2/renren-vue;
            index  index.html;
        }
    }
}
```

```bash
# 启动第renren-vue-n2节点
docker run -it -d --name renren-vue-n2 -v /home/renren-vue-n2/nginx.conf:/etc/nginx/nginx.conf -v /home/renren-vue-n2:/home/renren-vue-n2/renren-vue --privileged --net=host nginx
```

宿主机/home/renren-vue-n3/nginx.conf的配置文件

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    server {
        listen 6503;
        server_name  192.168.99.104;
        location  /  {
            root  /home/renren-vue-n3/renren-vue;
            index  index.html;
        }
    }
}
```

### 启动renren-vue-n3节点

```bash
#启动第renren-vue-n3节点
docker run -it -d --name renren-vue-n3 -v /home/renren-vue-n3/nginx.conf:/etc/nginx/nginx.conf -v /home/renren-vue-n3/renren-vue:/home/renren-vue-n3/renren-vue --privileged --net=host nginx
```

### 配置负载均衡

宿主机/home/renren-vue-fz1/nginx.conf配置文件

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
   worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

    upstream renren-vue-node {
        server 192.168.99.104:6501;
        server 192.168.99.104:6502;
        server 192.168.99.104:6503;
    }
    server {
        listen       6601;
        server_name  192.168.99.104;
        location / {  
            proxy_pass   http://renren-vue-node;
            index  index.html index.htm;  
        }  
    }
}
```

宿主机/home/renren-vue-fz1/keepalived.conf配置文件

```bash
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        192.168.99.152
    }
}
virtual_server 192.168.99.152 6701 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    real_server 192.168.99.104 6601 {
        weight 1
    }
}
```

```bash
#启动renren-vue-fz1节点
docker run -it -d --name renren-vue-fz1 -v /home/renren-vue-fz1/nginx.conf:/etc/nginx/nginx.conf -v /home/renren-vue-fz1/keepalived.conf:/etc/keepalived/keepalived.conf --net=host --privileged nginx
```

宿主机/home/renren-vue-fz2/nginx.conf配置文件

```bash
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    proxy_redirect          off;
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    client_max_body_size    10m;
    client_body_buffer_size   128k;
    proxy_connect_timeout   5s;
    proxy_send_timeout      5s;
    proxy_read_timeout      5s;
    proxy_buffer_size        4k;
    proxy_buffers           4 32k;
    proxy_busy_buffers_size  64k;
    proxy_temp_file_write_size 64k;

 upstream renren-vue-node {
     server 192.168.99.104:6501;
     server 192.168.99.104:6502;
     server 192.168.99.104:6503;
 }
 server {
        listen       6602;
        server_name  192.168.99.104;
        location / {  
            proxy_pass   http://renren-vue-node;
            index  index.html index.htm;  
        }  
    }
}
```

宿主机/home/renren-vue-fz2/keepalived.conf配置文件

```bash
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }
    virtual_ipaddress {
        192.168.99.152
    }
}
virtual_server 192.168.99.152 6701 {
    delay_loop 3
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
    real_server 192.168.99.104 6602 {
        weight 1
    }
}
```

```bash
#启动renren-vue-fz2节点
docker run -it -d --name renren-vue-fz2 -v /home/renren-vue-fz2/nginx.conf:/etc/nginx/nginx.conf -v /home/renren-vue-fz2/keepalived.conf:/etc/keepalived/keepalived.conf --net=host --privileged nginx
```

### 配置双机热备

分别进入renren-vue-fz1和renren-vue-fz2容器,安装并启动keepalived

```bash
#进入ff1节点
docker exec -it renren-vue-fz1 bash

#更新软件包
apt-get update

#安装启动Keepalived
apt-get install keepalived

#启动Keepalived
service keepalived start\

#进入renren-vue-fz1节点
docker exec -it renren-vue-fz2 bash

#更新软件包
apt-get update

#安装Keepalived
apt-get install keepalived

#启动Keepalived
service keepalived start
```
