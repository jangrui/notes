# MySQL

## 安装

```bash
yum install -y mariadb-server
systemctl enable mariadb
systemctl start mariadb
```

## 连接数据库

```bash
mysql -uroot -p
```

> -u # 指定用户。
>
> -p # 指定密码。安全起见，可不直接跟密码，然后在交互窗口中输入密码。

## 添加用户

```bash
grant all on *.* to 'admin'@'%' identified by 'adminpassword';
```

## 常用管理

```bash
show databases; # 列出数据库
use mysql; # 操作 mysql 库
select database(); # 查看当前操作的库
show tables; # 列出表信息
show columns from mysql.servers; # 查看指定表的属性
desc mysql.user; # 查看指定表的属性
show index from servers; # 查看指定表的索引信息
show table status from mysql; # 显示 mysql 库中所有表的信息
show table status from mysql like 'innodb%'; # 显示 mysql 库中所有以 innodb 开头的表信息
show variables like '%datadir%'; # 查看数据存放路径
show variables like 'Threads%'; # 查看当前连接数，并发数
show variables like '%max_connections%'; # 查看最大连接数
show grant for 'root'@'localhost'; # 查看用户的权限
select user,host from mysql.user; # 查看所有用户
drop user 'admin'@'%'; # 删除用户
create database jangrui; # 创建数据库
drop database jangrui; # 删除数据库
exit # 退出 mysql 交互窗口
```

