# Zabbix 安装

## 环境要求

5.0 版本对基础环境的要求有大的变化，最大的就是对 php 版本的要求，最低要求 7.2.0 版本,对 php 扩展组件版本也有要求，详见[官网文档](https://www.zabbix.com/documentation/current/manual/installation/requirements)。

## YUM 安装

操作系统：CentOS Linux release 7.8.2003 (Core) x86_64

安装方式：最小化安装

### 初始化系统

```bash
curl -O https://www.jangrui.com/centos-init.sh && bash centos-init.sh
```

> 脚本操作说明：
> - 关闭系统防火墙。 
> - 关闭 selinux。 
> - 更换阿里云镜像。 
> - 添加 epel 阿里云镜像。
> - 更新系统。
> - 安装常用包。
> - 同步时间。
> - 更新内核。
> - 开启 journal 日志。
> - 创建 swap。
> - 更新内核。
> - 优化内核参数。
> - 开启 ipvs 转发。
> - 安装 docker。

### 安装 zabbix rpm 源

```bash
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
sed -i 's|repo.zabbix.com|mirrors.aliyun.com/zabbix|g' /etc/yum.repos.d/zabbix.repo
yum makecache
```

### 安装 Zabbix server 和 agent

```bash
yum install zabbix-server-mysql zabbix-agent -y
```

### 安装 Software Collections

> 便于后续安装高版本的 php，默认 yum 安装的 php 版本为 5.4 过低。

```bash
yum install centos-release-scl -y
```

### 启用 Zabbix 前端源

```bash
sed -i "s|enabled=0|enabled=1|" /etc/yum.repos.d/zabbix.repo
```

### 安装 Zabbix 前端和相关环境

```bash
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```

### 安装数据库

> 由于使用 yum 安装 zabbix，不自动依赖安装数据库，因此需要手动安装数据库，这里使用 yum 安装 centos7 默认的 mariadb 数据库。

```bash
yum install mariadb-server -y
systemctl enable --now mariadb
mysql_secure_installation
```

### 建立 zabbix 数据库

```bash
mysql -uroot -proot -e "
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@'localhost' identified by 'zabbix';
grant all privileges on zabbix.* to zabbix@'localhost';
"
```

> 新版 mysql 对密码校验插件，所以可以通过以下命令设置降低密码校验等级。

```bash
mysql -uroot -proot -e "
set global validate_password_policy=0;
set global validate_password_length=4;
"
```

### 导入 zabbix 数据库

```bash
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -pzabbix zabbix
```

### 修改 zabbix 配置文件

```bash
sed -i "s|# DBPassword=|DBPassword=zabbix|" /etc/zabbix/zabbix_server.conf
sed -i "s|;.*|php_value[date.timezone] = Asia/Shanghai|" /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
```

### 启动相关服务

```bash
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```
