# Docker下部署Nginx高可用负载均衡集群

以 [renren-fast](https://gitee.com/renrenio/renren-fast) 后端项目为例，创建 Nginx 高可用负载均衡集群

## 打包部署后端项目

1. 进入人人开源后端项目，执行打包（修改配置文件，更改端口，打包三次生成三个JAR文件）

```bash
mvn clean install -Dmaven.test.skip=true
```

2. 安装Java镜像

```bash
docker pull java
```

3. 创建3节点Java容器

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

4. 安装Nginx镜像

```bash
docker pull nginx
```

5. 创建Nginx容器，配置负载均衡

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

   创建第1个Nginx节点

```bash
docker run -it -d --name n1 -v /home/n1/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx
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

   创建第2个Nginx节点

```bash
docker run -it -d --name n2 -v /home/n2/nginx.conf:/etc/nginx/nginx.conf --net=host --privileged nginx
```

6. 在Nginx容器安装Keepalived

```bash
#进入n1节点
docker exec -it n1 bash
#更新软件包
apt-get update
#安装VIM
apt-get install vim
#安装Keepalived
apt-get install keepalived
#编辑Keepalived配置文件(如下)
vim /etc/keepalived/keepalived.conf
#启动Keepalived
service keepalived start
```

```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens33
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

```bash
#进入n1节点
docker exec -it n2 bash
#更新软件包
apt-get update
#安装VIM
apt-get install vim
#安装Keepalived
apt-get install keepalived
#编辑Keepalived配置文件(如下)
vim /etc/keepalived/keepalived.conf
#启动Keepalived
service keepalived start
```

```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens33
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