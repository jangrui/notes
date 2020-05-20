# MySQL

## 安装

```bash
yum install -y mariadb-server
systemctl enable mariadb
systemctl start mariadb
```

## 连接数据库

```bash
mysql -uroot
```

> -u # 指定用户。
>
> -p # 指定密码。安全起见，可不直接跟密码，然后在交互窗口中输入密码。
>
> exit # 退出交互窗口

## 添加用户

```bash
MariaDB [(none)]> grant all on *.* to admin@'%' identified by 'admin.com';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all on *.* to admin@'localhost' identified by 'admin.com';
Query OK, 0 rows affected (0.00 sec)
```

## 创建数据库

```bash
mysql -uadmin -padmin.com

# 创建数据库时不指定字符集，将使用 mysql server 默认字符集
MariaDB [(none)]> create database jangrui;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show create database jangrui ;
+----------+--------------------------------------------------------------------+
| Database | Create Database                                                    |
+----------+--------------------------------------------------------------------+
| jangrui  | CREATE DATABASE `jangrui` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+--------------------------------------------------------------------+
1 row in set (0.00 sec)

# 创建数据库时指定字符集
MariaDB [(none)]> create database if not exists nuonuo default charset utf8mb4 collate utf8mb4_general_ci;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show create database nuonuo;
+----------+--------------------------------------------------------------------+
| Database | Create Database                                                    |
+----------+--------------------------------------------------------------------+
| nuonuo   | CREATE DATABASE `nuonuo` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+--------------------------------------------------------------------+
1 row in set (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jangrui            |
| mysql              |
| nuonuo             |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)
```

## 修改字符集

```bash
MariaDB [(none)]> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

# 修改数据库字符集
MariaDB [(none)]> show create database jangrui;
+----------+---------------------------------------------------------------------+
| Database | Create Database                                                     |
+----------+---------------------------------------------------------------------+
| jangrui  | CREATE DATABASE `jangrui` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+---------------------------------------------------------------------+
1 row in set (0.00 sec)

# 临时修改 mysql 默认字符集
MariaDB [(none)]> create database dbtest;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

MariaDB [(none)]> show create database dbtest;
+----------+--------------------------------------------------------------------+
| Database | Create Database                                                    |
+----------+--------------------------------------------------------------------+
| dbtest   | CREATE DATABASE `dbtest` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+--------------------------------------------------------------------+
1 row in set (0.00 sec)
```

> 字符集临时设置：`set character_set_server = utf8mb4`，重启 mysql 字符集会回到默认值，但之前创建的数据库字符集保持不变。
> 
> 字符集永久设置：`sed -i '/\[mysqld\]/a\character_set_server=utf8mb4' /etc/my.cnf`，重启 mysql 永久生效。

## 删除数据库

```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| dbtest             |
| jangrui            |
| mysql              |
| nuonuo             |
| performance_schema |
| test               |
+--------------------+
7 rows in set (0.00 sec)

MariaDB [(none)]> drop database dbtest;
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jangrui            |
| mysql              |
| nuonuo             |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.01 sec)
```

## 选择操作指定数据库

连接到 MySQL 数据库后，可能有多个可以操作的数据库，所以需要选择要操作的数据库。

```bash
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| jangrui            |
| mysql              |
| nuonuo             |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.01 sec)

MariaDB [(none)]> use jangrui;
Database changed

MariaDB [jangrui]> select database();
+------------+
| database() |
+------------+
| jangrui    |
+------------+
1 row in set (0.00 sec)
```

## 创建数据表

数据表的创建需要指定 `表名`、`列名（字段名）`、`定义列属性`，还需要定义`主键列`，为所有的表添加主键，是一个使用数据库的好习惯，主键将帮助用户在迁移数据时，不会引入重复数据问题。

```bash
MariaDB [jangrui]> create table if not exists `article`(
    -> `id` int unsigned auto_increment,
    -> `title` varchar(100) not null,
    -> `author` varchar(100) not null,
    -> `date` date,
    -> primary key(id)
    -> )engine=innodb default charset=utf8mb4;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> desc article;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(100)     | NO   |     | NULL    |                |
| author | varchar(100)     | NO   |     | NULL    |                |
| date   | date             | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

> - `auto_increment`: 设置字段为自增，一般用于主键，数值会自动加 1。
> - `not null`: 设置字段不能为空。
> - `primary key`: 设置主键列，可以定义多个列为主键，以逗号分隔。
> - `engine`: 设置储存引擎，默认为 InnoDB。
> - `charset`: 设置表的字符集。

## 删除数据表

```bash
MariaDB [jangrui]> drop table article;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> show tables;
Empty set (0.00 sec)
```

## INSERT 插入数据

```bash
MariaDB [jangrui]> insert into
    -> article(title, author, date)
    -> values("MySQL Insert", "jangrui", now());
Query OK, 1 row affected, 1 warning (0.00 sec)
```

## SELECT 数据查询

```bash
MariaDB [jangrui]> select * from article;
+----+--------------+---------+------------+
| id | title        | author  | date       |
+----+--------------+---------+------------+
|  1 | MySQL Insert | jangrui | 2020-03-03 |
+----+--------------+---------+------------+
1 row in set (0.00 sec)
```

> - 查询语句中你可以使用一个或者多个表，表之间使用逗号 `,` 分割，并使用 WHERE 语句来设定查询条件。
> - SELECT 命令可以读取一条或者多条记录。
> - 可以使用星号 `*` 来代替其他字段，SELECT 语句会返回表的所有字段数据
> - 可以使用 WHERE 语句来包含任何条件。
> - 可以使用 LIMIT 属性来设定返回的记录数。
> - 可以通过 OFFSET 指定 SELECT 语句开始查询的数据偏移量。默认情况下偏移量为0。

## WHERE 子句

```bash
MariaDB [jangrui]> select * from article where id='1';
+----+--------------+---------+------------+
| id | title        | author  | date       |
+----+--------------+---------+------------+
|  1 | MySQL Insert | jangrui | 2020-03-03 |
+----+--------------+---------+------------+
1 row in set (0.00 sec)

MariaDB [jangrui]> select * from article where id='2' and author='jangrui';
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  2 | MySQL Data Types | jangrui | 2020-03-03 |
+----+------------------+---------+------------+
1 row in set (0.00 sec)
```

> - 可以使用 AND 或者 OR 指定一个或多个条件。
> - WHERE 子句可以运用于 SQL 的 INSERT、DELETE 或者 UPDATE 命令。

## UPDATE 更新数据

```bash
MariaDB [jangrui]> update article set author='nuonuo' where id='1';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [jangrui]> select * from article where id=1;
+----+--------------+--------+------------+
| id | title        | author | date       |
+----+--------------+--------+------------+
|  1 | MySQL Insert | nuonuo | 2020-03-03 |
+----+--------------+--------+------------+
1 row in set (0.00 sec)
MariaDB [jangrui]> update article set author="jangrui", title="mysql update" where date="2020-03-03";
Query OK, 2 rows affected (0.01 sec)
Rows matched: 2  Changed: 2  Warnings: 0

MariaDB [jangrui]> select * from article;
+----+--------------+---------+------------+
| id | title        | author  | date       |
+----+--------------+---------+------------+
|  1 | mysql update | jangrui | 2020-03-03 |
|  2 | mysql update | jangrui | 2020-03-03 |
+----+--------------+---------+------------+
2 rows in set (0.00 sec)
```

> - 可以同时更新一个或多个字段，每个字段以 `,` 分隔。

## DELETE 语句

DELETE FROM 命令用来删除 MySQL 数据表中的记录。

```bash
MariaDB [jangrui]> delete from article where id=2;
Query OK, 1 row affected (0.00 sec)

MariaDB [jangrui]> select * from article;
+----+--------------+---------+------------+
| id | title        | author  | date       |
+----+--------------+---------+------------+
|  1 | mysql update | jangrui | 2020-03-03 |
+----+--------------+---------+------------+
1 row in set (0.00 sec)

MariaDB [jangrui]> insert into article(title, author, date) values("MySQL Data Types", "jangrui", now());
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [jangrui]> select * from article;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  1 | mysql update     | jangrui | 2020-03-03 |
|  3 | MySQL Data Types | jangrui | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)

MariaDB [jangrui]> delete from article where id=1 or id=3;
Query OK, 2 rows affected (0.00 sec)

MariaDB [jangrui]> select * from article;
Empty set (0.00 sec)
```

## LIKE 子句

```bash
MariaDB [jangrui]> insert into article(title, author, date) values("MySQL Data Types", "jangrui", now());
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [jangrui]> insert into article(title, author, date) values("MySQL", "nuonuo", now());
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [jangrui]> select * from article;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  4 | MySQL Data Types | jangrui | 2020-03-03 |
|  5 | MySQL            | nuonuo  | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)

MariaDB [jangrui]> select * from article where author like "%nuo%";
+----+-------+--------+------------+
| id | title | author | date       |
+----+-------+--------+------------+
|  5 | MySQL | nuonuo | 2020-03-03 |
+----+-------+--------+------------+
1 row in set (0.01 sec)
```

> - 可以在 WHERE 子句中使用 LIKE 子句。
> - 可以使用 LIKE 子句代替等号 `=`。
> - LIKE 通常与 `%` 一同使用，类似于 UNIX 或正则表达式中的星号 `*`。

## UNION 操作符

UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。

```bash
SELECT 列名称 FROM 表名称 UNION SELECT 列名称 FROM 表名称 ORDER BY 列名称；
SELECT 列名称 FROM 表名称 UNION ALL SELECT 列名称 FROM 表名称 ORDER BY 列名称；
```

> - `UNION`: 用于将不同表中相同列中查询的数据展示出来；（不包括重复数据）
> - `UNION ALL`: 用于将不同表中相同列中查询的数据展示出来；（包括重复数据）

```bash
MariaDB [jangrui]> select * from article;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  4 | MySQL Data Types | jangrui | 2020-03-03 |
|  5 | MySQL            | nuonuo  | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)

# UNION 不能列出重复的值
MariaDB [jangrui]> select date from article union select title from article;
+------------------+
| date             |
+------------------+
| 2020-03-03       |
| MySQL Data Types |
| MySQL            |
+------------------+
3 rows in set (0.00 sec)

# UNION ALL 可以列出重复的值
MariaDB [jangrui]> select date from article union all select title from article;
+------------------+
| date             |
+------------------+
| 2020-03-03       |
| 2020-03-03       |
| MySQL Data Types |
| MySQL            |
+------------------+
4 rows in set (0.00 sec)

# 带有 WHERE 的 UNION ALL
MariaDB [jangrui]> select date,author from article where date="2020-03-03" union all select title,author from article where date="2020-03-03";
+------------------+---------+
| date             | author  |
+------------------+---------+
| 2020-03-03       | jangrui |
| 2020-03-03       | nuonuo  |
| MySQL Data Types | jangrui |
| MySQL            | nuonuo  |
+------------------+---------+
4 rows in set (0.00 sec)
```

## ORDER 排序

ORDER BY 子句来设定指定字段以哪种方式来进行排序，再返回搜索结果。

```bash
MariaDB [jangrui]> select * from article order by title;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  5 | MySQL            | nuonuo  | 2020-03-03 |
|  4 | MySQL Data Types | jangrui | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)

# 根据 title 字段，以升序排列
MariaDB [jangrui]> select * from article order by id asc;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  4 | MySQL Data Types | jangrui | 2020-03-03 |
|  5 | MySQL            | nuonuo  | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)

# 根据 title 字段，以降序排列
MariaDB [jangrui]> select * from article order by id desc;
+----+------------------+---------+------------+
| id | title            | author  | date       |
+----+------------------+---------+------------+
|  5 | MySQL            | nuonuo  | 2020-03-03 |
|  4 | MySQL Data Types | jangrui | 2020-03-03 |
+----+------------------+---------+------------+
2 rows in set (0.00 sec)
```

## GROUP 分组

GROUP BY 语句根据一个或多个列对结果集进行分组。

```bash
MariaDB [jangrui]> create table login(
    -> id int unsigned auto_increment,
    -> name varchar(20) not null default '',
    -> date datetime not null,
    -> logins int unsigned not null default '0' comment '登录次数',
    -> primary key(id)
    -> )engine=innodb default charset=utf8mb4;
Query OK, 0 rows affected (0.01 sec)

MariaDB [jangrui]> desc login;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| name   | varchar(20)      | NO   |     |         |                |
| date   | datetime         | NO   |     | NULL    |                |
| logins | int(10) unsigned | NO   |     | 0       |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

MariaDB [jangrui]> insert into login(name, date, logins)
    -> values
    -> ("jangrui", now(), 3),
    -> ("nuonuo", now(), 4),
    -> ("yangjie", now(), 5);
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> insert into login(name, date, logins) 
    -> values
    -> ("jangrui", date_add(now(), interval 1 day), 3),
    -> ("nuonuo", date_add(now(), interval 1 day), 4);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from login;
+----+---------+---------------------+--------+
| id | name    | date                | logins |
+----+---------+---------------------+--------+
|  1 | jangrui | 2020-03-03 23:24:30 |      3 |
|  2 | nuonuo  | 2020-03-03 23:24:30 |      4 |
|  3 | yangjie | 2020-03-03 23:24:30 |      5 |
|  4 | jangrui | 2020-03-04 23:24:42 |      3 |
|  5 | nuonuo  | 2020-03-04 23:24:42 |      4 |
+----+---------+---------------------+--------+
5 rows in set (0.00 sec)

# 使用 GROUP BY 语句将数据表按 `name` 字段进行分组，并统计每个人有多少条记录：
MariaDB [jangrui]> select name, count(*) from login group by name;
+---------+----------+
| name    | count(*) |
+---------+----------+
| jangrui |        2 |
| nuonuo  |        2 |
| yangjie |        1 |
+---------+----------+
3 rows in set (0.00 sec)

# 使用 WITH ROLLUP 按 `name` 字段进行分组，统计每个人登录的次数
MariaDB [jangrui]> select name, sum(logins) as logins from login group by name with rollup;
+---------+--------+
| name    | logins |
+---------+--------+
| jangrui |      6 |
| nuonuo  |      8 |
| yangjie |      5 |
| NULL    |     19 |
+---------+--------+
4 rows in set (0.00 sec)

MariaDB [jangrui]> select coalesce(name, 'all_logins'), sum(logins) as logins from login group by name with rollup;
+------------------------------+--------+
| coalesce(name, 'all_logins') | logins |
+------------------------------+--------+
| jangrui                      |      6 |
| nuonuo                       |      8 |
| yangjie                      |      5 |
| all_logins                   |     19 |
+------------------------------+--------+
4 rows in set (0.00 sec)
```

> - 可以使用 COUNT, SUM, AVG,等函数。
> - `WITH ROLLUP`: 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。
> - `coalesce`: 用来设置一个可以取代 null 的名称。
> - `coalesce` 语法: `select coalesce(a,b,c);`，如果 a=null，则选 b；如果 b=null，则选 c；如果 c=null，则返回 null。

## JOIN 连接的使用

JOIN 在多个表中查询数据。

- `INNER JOIN`（内连接,或等值连接）：获取两个表中字段匹配关系的记录，相当于交集。

```bash
# notes 数据表
MariaDB [jangrui]> select * from notes;
+----+-----------+---------+---------------------+
| id | title     | author  | date                |
+----+-----------+---------+---------------------+
|  1 | 第一篇    | jangrui | 2020-03-04 19:38:11 |
|  2 | 第二篇    | nuonuo  | 2020-03-05 19:38:11 |
|  3 | 第三篇    | jangrui | 2020-03-04 21:38:11 |
|  4 | 第四篇    | jangrui | 2020-03-04 22:38:11 |
|  5 | 第五篇    | nuonuo  | 2020-03-04 23:38:11 |
+----+-----------+---------+---------------------+
5 rows in set (0.00 sec)

# author 数据表
MariaDB [jangrui]> select * from author;
+----------+-----+
| name     | age |
+----------+-----+
| jangrui  |  25 |
| xiaohong |   1 |
| xiaoming |  26 |
+----------+-----+
3 rows in set (0.00 sec)

# 使用 INNER JOIN(可以省略 INNER 使用 JOIN，效果一样)来连接 author 表中所有 name 字段在 notes 表对应的 author 字段
MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a join author b on a.author = b.name;
+----+-----------+---------+-----+---------------------+
| id | title     | name    | age | date                |
+----+-----------+---------+-----+---------------------+
|  1 | 第一篇    | jangrui |  25 | 2020-03-04 20:12:23 |
|  3 | 第三篇    | jangrui |  25 | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui |  25 | 2020-03-04 23:12:23 |
+----+-----------+---------+-----+---------------------+
3 rows in set (0.00 sec)

# 使用 where 语句替换 join（等价）
MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a, author b where a.author = b.name;
+----+-----------+---------+-----+---------------------+
| id | title     | name    | age | date                |
+----+-----------+---------+-----+---------------------+
|  1 | 第一篇    | jangrui |  25 | 2020-03-04 20:12:23 |
|  3 | 第三篇    | jangrui |  25 | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui |  25 | 2020-03-04 23:12:23 |
+----+-----------+---------+-----+---------------------+
3 rows in set (0.00 sec)
```

- `LEFT JOIN`（左连接）：获取左表所有记录，即使右表没有对应匹配的记录。

```bash
MariaDB [jangrui]> select a.id,a.title,a.author,b.age,a.date from notes a left join author b on a.author = b.name;
+----+-----------+---------+------+---------------------+
| id | title     | author  | age  | date                |
+----+-----------+---------+------+---------------------+
|  1 | 第一篇    | jangrui |   25 | 2020-03-04 20:12:23 |
|  2 | 第二篇    | nuonuo  | NULL | 2020-03-05 20:12:23 |
|  3 | 第三篇    | jangrui |   25 | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui |   25 | 2020-03-04 23:12:23 |
|  5 | 第五篇    | nuonuo  | NULL | 2020-03-05 00:12:23 |
+----+-----------+---------+------+---------------------+
5 rows in set (0.00 sec)

MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a left join author b on a.author = b.name;
+----+-----------+---------+------+---------------------+
| id | title     | name    | age  | date                |
+----+-----------+---------+------+---------------------+
|  1 | 第一篇    | jangrui |   25 | 2020-03-04 20:12:23 |
|  2 | 第二篇    | NULL    | NULL | 2020-03-05 20:12:23 |
|  3 | 第三篇    | jangrui |   25 | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui |   25 | 2020-03-04 23:12:23 |
|  5 | 第五篇    | NULL    | NULL | 2020-03-05 00:12:23 |
+----+-----------+---------+------+---------------------+
5 rows in set (0.00 sec)
```

- `RIGHT JOIN`（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

```bash
MariaDB [jangrui]> select a.id,a.title,a.author,b.age,a.date from notes a right join author b on a.author = b.name;
+------+-----------+---------+-----+---------------------+
| id   | title     | author  | age | date                |
+------+-----------+---------+-----+---------------------+
|    1 | 第一篇    | jangrui |  25 | 2020-03-04 20:12:23 |
|    3 | 第三篇    | jangrui |  25 | 2020-03-04 22:12:23 |
|    4 | 第四篇    | jangrui |  25 | 2020-03-04 23:12:23 |
| NULL | NULL      | NULL    |   1 | NULL                |
| NULL | NULL      | NULL    |  26 | NULL                |
+------+-----------+---------+-----+---------------------+
5 rows in set (0.00 sec)

MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name;
+------+-----------+----------+-----+---------------------+
| id   | title     | name     | age | date                |
+------+-----------+----------+-----+---------------------+
|    1 | 第一篇    | jangrui  |  25 | 2020-03-04 20:12:23 |
|    3 | 第三篇    | jangrui  |  25 | 2020-03-04 22:12:23 |
|    4 | 第四篇    | jangrui  |  25 | 2020-03-04 23:12:23 |
| NULL | NULL      | xiaohong |   1 | NULL                |
| NULL | NULL      | xiaoming |  26 | NULL                |
+------+-----------+----------+-----+---------------------+
5 rows in set (0.00 sec)
```

## NULL值处理

关于 NULL 的条件比较运算是比较特殊的。你不能使用 = NULL 或 != NULL 在列中查找 NULL 值 。

在 MySQL 中，NULL 值与任何其它值的比较（即使是 NULL）永远返回 NULL，即 NULL = NULL 返回 NULL 。

MySQL 中处理 NULL 使用 IS NULL 和 IS NOT NULL 运算符。

- `IS NULL`: 当列的值是 NULL,此运算符返回 true。
- `IS NOT NULL`: 当列的值不为 NULL, 运算符返回 true。
- `<=>`: 比较操作符（不同于 = 运算符），当比较的的两个值相等或者都为 NULL 时返回 true。

```bash
MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name where id=3;
+----+-----------+---------+-----+---------------------+
| id | title     | name    | age | date                |
+----+-----------+---------+-----+---------------------+
|  3 | 第三篇    | jangrui |  25 | 2020-03-04 22:12:23 |
+----+-----------+---------+-----+---------------------+
1 row in set (0.00 sec)

MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name where id=null;
Empty set (0.00 sec)

MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name where id=!null;
Empty set (0.00 sec)

# is null 处理 null 值
MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name where id is null;
+------+-------+----------+-----+------+
| id   | title | name     | age | date |
+------+-------+----------+-----+------+
| NULL | NULL  | xiaohong |   1 | NULL |
| NULL | NULL  | xiaoming |  26 | NULL |
+------+-------+----------+-----+------+
2 rows in set (0.00 sec)

# is not null 处理 null 值
MariaDB [jangrui]> select a.id,a.title,b.name,b.age,a.date from notes a right join author b on a.author = b.name where id is not null;
+----+-----------+---------+-----+---------------------+
| id | title     | name    | age | date                |
+----+-----------+---------+-----+---------------------+
|  1 | 第一篇    | jangrui |  25 | 2020-03-04 20:12:23 |
|  3 | 第三篇    | jangrui |  25 | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui |  25 | 2020-03-04 23:12:23 |
+----+-----------+---------+-----+---------------------+
3 rows in set (0.00 sec)
```

## 正则表达式

MySQL 支持 REGEXP 操作符来进行正则表达式匹配。

```bash
# 查找 name 字段中以 `j` 为开头的所有数据：
MariaDB [jangrui]> select name from author where name regexp '^j';
+---------+
| name    |
+---------+
| jangrui |
+---------+
1 row in set (0.00 sec)

# 查找 name 字段中以 `g` 为结尾的所有数据：
MariaDB [jangrui]> select name from author where name regexp 'g$';
+----------+
| name     |
+----------+
| xiaohong |
| xiaoming |
+----------+
2 rows in set (0.00 sec)

# 查找 name 字段中包含 `min` 字符串的所有数据：
MariaDB [jangrui]> select name from author where name regexp 'min';
+----------+
| name     |
+----------+
| xiaoming |
+----------+
1 row in set (0.00 sec)

# 查找 name 字段中以 `a-m` 之间的所有小写字母开头的所有数据：
MariaDB [jangrui]> select name from author where name regexp '^[a-m]';
+---------+
| name    |
+---------+
| jangrui |
+---------+
1 row in set (0.00 sec)

# 查找 name 字段中以 `a-m` 之间的所有小写字母开头或以 `g` 字母结尾的所有数据：
MariaDB [jangrui]> select name from author where name regexp '^[a-m]|g$';
+----------+
| name     |
+----------+
| jangrui  |
| xiaohong |
| xiaoming |
+----------+
3 rows in set (0.00 sec)
```

## ALTER 命令

alter 命令用来修改数据表名或者修改数据表字段。

```bash
MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     |         |                |
| date   | datetime         | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

MariaDB [jangrui]> select * from notes;
+----+-----------+---------+---------------------+
| id | title     | author  | date                |
+----+-----------+---------+---------------------+
|  1 | 第一篇    | jangrui | 2020-03-04 20:12:23 |
|  2 | 第二篇    | nuonuo  | 2020-03-05 20:12:23 |
|  3 | 第三篇    | jangrui | 2020-03-04 22:12:23 |
|  4 | 第四篇    | jangrui | 2020-03-04 23:12:23 |
|  5 | 第五篇    | nuonuo  | 2020-03-05 00:12:23 |
+----+-----------+---------+---------------------+
5 rows in set (0.00 sec)

# 删除字段
MariaDB [jangrui]> alter table notes drop date;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     |         |                |
+--------+------------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

# 添加字段
MariaDB [jangrui]> alter table notes add time datetime;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     |         |                |
| time   | datetime         | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

# 修改字段名
MariaDB [jangrui]> alter table notes change time date date;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     |         |                |
| date   | date             | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)

# 修改字段类型，指定字段 author 为 NOT NULL 且默认值为 jangrui。
MariaDB [jangrui]> alter table notes modify author varchar(10) not null default 'jangrui';
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     | jangrui |                |
| date   | date             | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

# 也可以用 alter 命令来修改字段的默认值
MariaDB [jangrui]> alter table notes alter author set default 'nuonuo';
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     | nuonuo  |                |
| date   | date             | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

# 也可以用 alter 命令和 drop 子句删除字段默认值
MariaDB [jangrui]> alter table notes alter author drop default;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   |     |         |                |
| author | varchar(10)      | NO   |     | NULL    |                |
| date   | date             | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

# 还可以用 alter 命令和 type 子句修改表类型
MariaDB [jangrui]> alter table notes engine = myisam;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show table status from jangrui like 'notes'\G;
*************************** 1. row ***************************
           Name: notes
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 28
    Data_length: 140
Max_data_length: 281474976710655
   Index_length: 2048
      Data_free: 0
 Auto_increment: 6
    Create_time: 2020-03-05 10:10:59
    Update_time: 2020-03-05 10:10:59
     Check_time: NULL
      Collation: utf8mb4_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

ERROR: No query specified

# 修改字段的相对位置
MariaDB [jangrui]> alter table notes modify date datetime after title;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from notes;
+----+-----------+------+---------+
| id | title     | date | author  |
+----+-----------+------+---------+
|  1 | 第一篇    | NULL | jangrui |
|  2 | 第二篇    | NULL | nuonuo  |
|  3 | 第三篇    | NULL | jangrui |
|  4 | 第四篇    | NULL | jangrui |
|  5 | 第五篇    | NULL | nuonuo  |
+----+-----------+------+---------+
5 rows in set (0.00 sec)

# 修改表名
MariaDB [jangrui]> alter table notes rename to mynotes;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> show table status from jangrui like 'mynotes' \G;
*************************** 1. row ***************************
           Name: mynotes
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 5
 Avg_row_length: 28
    Data_length: 140
Max_data_length: 281474976710655
   Index_length: 2048
      Data_free: 0
 Auto_increment: 6
    Create_time: 2020-03-05 10:10:59
    Update_time: 2020-03-05 10:10:59
     Check_time: NULL
      Collation: utf8mb4_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

ERROR: No query specified
```

> - 如果数据表中只剩余一个字段则无法使用 DROP 来删除字段。

## 索引

索引可以大大提高 MySQL 的检索速度。

索引分单列索引和组合索引。单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。组合索引，即一个索引包含多个列。

索引是应用在 SQL 查询语句的条件(一般作为 WHERE 子句的条件)。

索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。

索引的缺点：虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行 INSERT、UPDATE 和 DELETE。因为更新表时，不仅要保存数据，还要保存索引文件。建立索引会占用磁盘空间的索引文件。

```bash
# 查看索引
MariaDB [jangrui]> show index from notes;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| notes |          0 | PRIMARY  |            1 | id          | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.00 sec)
```

### 普通索引 

```bash
# create 创建索引
MariaDB [jangrui]> create index title on notes(title);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show index from notes;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| notes |          0 | PRIMARY  |            1 | id          | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
| notes |          1 | title    |            1 | title       | A         |        NULL |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

## alter 创建索引
MariaDB [jangrui]> alter table notes add index author(author);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show index from notes;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| notes |          0 | PRIMARY  |            1 | id          | A         |           5 |     NULL | NULL   |      | BTREE      |         |               |
| notes |          1 | title    |            1 | title       | A         |        NULL |     NULL | NULL   |      | BTREE      |         |               |
| notes |          1 | author   |            1 | author      | A         |        NULL |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)

# 创建表时添加索引
MariaDB [jangrui]> create table city(id int unsigned auto_increment primary key, name varchar(20) not null, index city(name(20)));
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> show index from city;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| city  |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| city  |          1 | city     |            1 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

# 删除索引
MariaDB [jangrui]> drop index city on city;
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show index from city;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| city  |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.00 sec)
```

### 唯一索引

索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

```bash
# create 创建
MariaDB [jangrui]> create unique index city on city(name);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show index from city;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| city  |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| city  |          0 | city     |            1 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

# alter 创建
MariaDB [jangrui]> alter table city add unique name(name);
Query OK, 0 rows affected (0.00 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> show index from city;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| city  |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| city  |          0 | city     |            1 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| city  |          0 | name     |            1 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)

# 创建表时直接指定
MariaDB [jangrui]> create table student(id int auto_increment primary key, name varchar(20) not null, age int not null, grade int not null, class int not null, unique name(name));
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> show index from student;
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table   | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| student |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
| student |          0 | name     |            1 | name        | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+---------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)
```

## 临时表

临时表用于保存一些临时数据时是非常有用的。临时表只在当前连接可见，当关闭连接时，Mysql会自动删除表并释放所有空间。

```bash
MariaDB [jangrui]> create temporary table tmp(
    -> id int auto_increment primary key,
    -> date date,
    -> datetime datetime,
    -> timestamp timestamp);
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> insert into tmp(date, datetime, timestamp) values
    -> (now(), now(), now()),
    -> (date_add(now(), interval 1 hour), date_add(now(), interval 1 hour), date_add(now(), interval 1 hour)),
    -> (date_add(now(), interval 1 day), date_add(now(), interval 1 day), date_add(now(), interval 1 day));
Query OK, 3 rows affected, 3 warnings (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 3

MariaDB [jangrui]> select * from tmp;
+----+------------+---------------------+---------------------+
| id | date       | datetime            | timestamp           |
+----+------------+---------------------+---------------------+
|  1 | 2020-03-05 | 2020-03-05 11:52:14 | 2020-03-05 11:52:14 |
|  2 | 2020-03-05 | 2020-03-05 12:52:14 | 2020-03-05 12:52:14 |
|  3 | 2020-03-06 | 2020-03-06 11:52:14 | 2020-03-06 11:52:14 |
+----+------------+---------------------+---------------------+
3 rows in set (0.00 sec)

# 当使用 SHOW TABLES 命令查看数据列表时，无法看到 tmp 表
MariaDB [jangrui]> show tables like 'tmp';
Empty set (0.00 sec)

# 删除 tmp 临时表（无意义，关闭连接时自动删除）
MariaDB [jangrui]> drop table tmp;
Query OK, 0 rows affected (0.00 sec)
```

## 复制表

```bash
# 复制表结构
MariaDB [jangrui]> create table clone_notes like notes;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> select * from clone_notes;
Empty set (0.00 sec)

# 复制内容
MariaDB [jangrui]> insert into clone_notes select * from notes;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from clone_notes;
+----+-----------+---------+------+
| id | title     | author  | date |
+----+-----------+---------+------+
|  1 | 第一篇    | jangrui | NULL |
|  2 | 第二篇    | nuonuo  | NULL |
|  3 | 第三篇    | jangrui | NULL |
|  4 | 第四篇    | jangrui | NULL |
|  5 | 第五篇    | nuonuo  | NULL |
+----+-----------+---------+------+
5 rows in set (0.00 sec)

# 直接复制表和数据
MariaDB [jangrui]> create table newdocs select * from notes;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from newdocs;
+----+-----------+---------+------+
| id | title     | author  | date |
+----+-----------+---------+------+
|  1 | 第一篇    | jangrui | NULL |
|  2 | 第二篇    | nuonuo  | NULL |
|  3 | 第三篇    | jangrui | NULL |
|  4 | 第四篇    | jangrui | NULL |
|  5 | 第五篇    | nuonuo  | NULL |
+----+-----------+---------+------+
5 rows in set (0.00 sec)

# 复制一部分字段和数据
MariaDB [jangrui]> create table clone_docs select title,author from notes;
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from clone_docs;
+-----------+---------+
| title     | author  |
+-----------+---------+
| 第一篇    | jangrui |
| 第二篇    | nuonuo  |
| 第三篇    | jangrui |
| 第四篇    | jangrui |
| 第五篇    | nuonuo  |
+-----------+---------+
5 rows in set (0.00 sec)

# 复制的同时修改字段信息
MariaDB [jangrui]> create table docs select title as article,author as name from notes;
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from docs;
+-----------+---------+
| article   | name    |
+-----------+---------+
| 第一篇    | jangrui |
| 第二篇    | nuonuo  |
| 第三篇    | jangrui |
| 第四篇    | jangrui |
| 第五篇    | nuonuo  |
+-----------+---------+
5 rows in set (0.00 sec)

# 复制的同时重新定义字段信息
MariaDB [jangrui]> create table new(id int auto_increment,title varchar(50) not null,author varchar(10) not null,date timestamp,primary key(id)) as (select * from notes);
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> desc new;
+--------+-------------+------+-----+-------------------+-----------------------------+
| Field  | Type        | Null | Key | Default           | Extra                       |
+--------+-------------+------+-----+-------------------+-----------------------------+
| id     | int(11)     | NO   | PRI | NULL              | auto_increment              |
| title  | varchar(50) | NO   |     | NULL              |                             |
| author | varchar(10) | NO   |     | NULL              |                             |
| date   | timestamp   | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+--------+-------------+------+-----+-------------------+-----------------------------+
4 rows in set (0.00 sec)

MariaDB [jangrui]> desc notes;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| title  | varchar(20)      | NO   | MUL |         |                |
| author | varchar(10)      | NO   | MUL | NULL    |                |
| date   | datetime         | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

## 序列

序列是一组整数：1, 2, 3, ...，由于一张数据表只能有一个字段自增主键， 如果想实现其他字段也实现自动增加，就可以使用序列来实现。

```bash
# auto_increment 函数实现自增列
MariaDB [jangrui]> create table auto (
    -> id int unsigned auto_increment primary key,
    -> name varchar(50) not null,
    -> date date not null);
Query OK, 0 rows affected (0.01 sec)

MariaDB [jangrui]> select * from auto;
Empty set (0.00 sec)

MariaDB [jangrui]> insert into auto values
    -> (null, "jangrui", now()),
    -> (null, "nuonuo", now()),
    -> (null, "yangjie", now());
Query OK, 3 rows affected, 3 warnings (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 3

MariaDB [jangrui]> select * from auto;
+----+---------+------------+
| id | name    | date       |
+----+---------+------------+
|  1 | jangrui | 2020-03-05 |
|  2 | nuonuo  | 2020-03-05 |
|  3 | yangjie | 2020-03-05 |
+----+---------+------------+
3 rows in set (0.00 sec)

# 设置序列的开始值
MariaDB [jangrui]> create table begin(
    -> id int auto_increment,
    -> name varchar(50) not null,
    -> primary key(id)
    -> ) auto_increment=10;
Query OK, 0 rows affected (0.02 sec)

MariaDB [jangrui]> insert into begin values
    -> (null, "jangrui"),
    -> (null, "nuonuo");
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from begin;
+----+---------+
| id | name    |
+----+---------+
| 10 | jangrui |
| 11 | nuonuo  |
+----+---------+
2 rows in set (0.01 sec)

# 重置序列
MariaDB [jangrui]> alter table begin drop id;
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> alter table begin add id int auto_increment first, add primary key(id), auto_increment=20;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from begin;
+----+---------+
| id | name    |
+----+---------+
| 20 | jangrui |
| 21 | nuonuo  |
+----+---------+
2 rows in set (0.00 sec)
```

## 处理重复数据

有些情况需要重复数据的存在，但有时候也需要删除这些重复的数据。

MySQL 数据表中设置指定的字段为 PRIMARY KEY（主键） 或者 UNIQUE（唯一） 索引来保证数据的唯一性。

### 避免重复数据

```bash
# 创建一个无主键表
MariaDB [jangrui]> create table t( id int null, name varchar(255), age int unsigned null);
Query OK, 0 rows affected (0.00 sec)

# 无索引内容
MariaDB [jangrui]> show index from t;
Empty set (0.00 sec)

# 可以添加重复数据
MariaDB [jangrui]> insert into t values(1,"jangrui",26), (1,"jangrui",26);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from t;
+------+---------+------+
| id   | name    | age  |
+------+---------+------+
|    1 | jangrui |   26 |
|    1 | jangrui |   26 |
+------+---------+------+
2 rows in set (0.00 sec)

# 创建一个有主键表
MariaDB [jangrui]> create table t1(id int null primary key, name varchar(255), age int unsigned null);
Query OK, 0 rows affected (0.00 sec)

# 有索引内容
MariaDB [jangrui]> show index from t1;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| t1    |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.00 sec)

# 无法添加重复内容
MariaDB [jangrui]> insert into t1 values(1,"jangrui",26), (1,"jangrui",26);
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'

# 使用 insert ignore into 可以添加重复内容
MariaDB [jangrui]> insert ignore into t1 values(1,"jangrui",26), (1,"jangrui",26);
Query OK, 1 row affected, 1 warning (0.00 sec)
Records: 2  Duplicates: 1  Warnings: 1

MariaDB [jangrui]> select * from t;
+------+---------+------+
| id   | name    | age  |
+------+---------+------+
|    1 | jangrui |   26 |
|    1 | jangrui |   26 |
+------+---------+------+
2 rows in set (0.00 sec)

# 使用 replace into 删除与主键重复内容，再更新记录
MariaDB [jangrui]> replace into t1 values(1,"jangrui",27);
Query OK, 2 rows affected (0.00 sec)

MariaDB [jangrui]> select * from t1;
+----+---------+------+
| id | name    | age  |
+----+---------+------+
|  1 | jangrui |   27 |
+----+---------+------+
1 row in set (0.00 sec)
```

> - `insert ignore into`: 当插入数据时，在设置了记录的唯一性后，如果插入重复数据，将不返回错误，只以警告形式返回。
> - `replace into`: 如果存在 primary 或 unique 相同的记录，则先删除掉。再插入新记录。

### 统计重复数据

```bash
MariaDB [jangrui]> select id,name,count(*) as count from t group by id, name having count > 1;
+------+---------+-------+
| id   | name    | count |
+------+---------+-------+
|    1 | jangrui |     2 |
+------+---------+-------+
1 row in set (0.00 sec)
```

> - 确定哪一列包含的值可能会重复。
> - 使用 COUNT(*) 代表可能重复出现的内容。
> - 在 GROUP BY 子句中进行分组查询可能出现的列。
> - HAVING 子句设置重复数大于 1。
> - count > 1 代表出现 2 次或 2 次以上。

### 过滤重复数据

```bash
# 使用 distinct 关键字过滤重复数据
MariaDB [jangrui]> select distinct * from t;
+------+---------+------+
| id   | name    | age  |
+------+---------+------+
|    1 | jangrui |   26 |
+------+---------+------+
1 row in set (0.00 sec)

# 使用 group by 分组查询读取不重复的数据
MariaDB [jangrui]> select * from t group by id;
+------+---------+------+
| id   | name    | age  |
+------+---------+------+
|    1 | jangrui |   26 |
|    2 | jangrui |   25 |
+------+---------+------+
2 rows in set (0.00 sec)
```

### 删除重复数据

```bash
# 重命名方式
MariaDB [jangrui]> create table tmp select * from t group by id;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> drop table t;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> alter table tmp rename to t;
Query OK, 0 rows affected (0.00 sec)

MariaDB [jangrui]> select * from t;
+------+---------+------+
| id   | name    | age  |
+------+---------+------+
|    1 | jangrui |   26 |
|    2 | jangrui |   25 |
+------+---------+------+
2 rows in set (0.00 sec)

MariaDB [jangrui]> insert into t values(2,"jangrui",25),(1,"jangrui",26),(1,"jangrui",25);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

MariaDB [jangrui]> select * from t;
+----+---------+-----+
| id | name    | age |
+----+---------+-----+
|  1 | jangrui |  26 |
|  2 | jangrui |  25 |
|  2 | jangrui |  25 |
|  1 | jangrui |  26 |
|  1 | jangrui |  25 |
+----+---------+-----+
5 rows in set (0.00 sec)

# 添加索引和主键方式（简单暴力不推荐）
MariaDB [jangrui]> alter ignore table t add primary key(name(20));
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 4  Warnings: 0

MariaDB [jangrui]> select * from t;
+----+---------+-----+
| id | name    | age |
+----+---------+-----+
|  1 | jangrui |  26 |
+----+---------+-----+
1 row in set (0.00 sec)
```

## 导出数据

- `select into outfile` 语句导出数据

```bash
MariaDB [jangrui]> select * from notes into outfile '/tmp/notes.sql';
Query OK, 5 rows affected (0.00 sec)
```

- `mysqldump` 导出数据

```bash
# 导出数据表
mysqldump -uroot -p jangrui notes > /tmp/notes-`date +%F`.sql

# 导出数据库
mysqldump -uroot -p jangrui > /tmp/jangrui-`date +%F`.sql

# 导出所有数据库
mysqldump -uroot -p --all-databases > /tmp/databases-`date +%F`.sql

# 只接导入远程服务器上
mysqldump -uroot -p jangrui | mysql -h jangrui.com jangrui

# 导出远程主机数据库到本地
mysqldump -h jangrui.com -uroot -p jangrui > /tmp/jangrui-`date +%F`.sql
```

## 导入数据

- mysql 命令导入

```bash
mysql -uroot -p < /tmp/jangrui.sql
```
- source 命令导入

```bash
MariaDB [mysql]> create database dump_jangrui;
MariaDB [mysql]> drop database dump_jangrui;
MariaDB [mysql]> create database jangrui_bak;
MariaDB [mysql]> use jangrui_bak;
MariaDB [jangrui_bak]> set names utf8mb4;
MariaDB [jangrui_bak]> source /tmp/jangrui.sql
MariaDB [jangrui_bak]> show tables;
```

## 其它常用命令

```bash
select database(); # 查看当前操作的库
show table status from mysql; # 显示 mysql 库中所有表的信息
show table status from mysql like 'innodb%'; # 显示 mysql 库中所有以 innodb 开头的表信息
show variables like '%datadir%'; # 查看数据存放路径
show variables like 'Threads%'; # 查看当前连接数，并发数
show variables like '%max_connections%'; # 查看最大连接数
select user,host from mysql.user; # 查看所有用户
drop user 'admin'@'%'; # 删除用户
show grant for 'root'@'localhost'; # 查看用户的权限
select @@sql_mode; # 查看 sql_mode
select @@foreign_key_checks; # 查看外键约束
set foreign_key_checks=0; # 临时禁用外键约束
set foreign_key_checks=1; # 临时启用外键约束
```
