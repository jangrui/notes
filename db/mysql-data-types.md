# MySQL 数据类型

MySQL 支持多种类型，大致可以分为三类：数值、日期/时间、字符串和枚举类型。

## 数值类型

- 整型

|类型|大小|范围（有符号）|范围（无符号）|用途|
|-|-|-|-|-|
|`TINYINT`|1 字节|(-128，127)|(0，255)|小整数值|
|`SMALLINT`|2 字节|(-32768，32767)|(0，65535)|大整数值|
|`MEDIUMINT`|3 字节|(-8388608，8388607)|(0，16777215)|大整数值|
|`INT` 或 `INTEGER`|4 字节|(-2147483648，2147483647)|(0，4294967295)|大整数值|
|`BIGINT`|8 字节|(-9233372036854775808，9223372036854775807)|(0，18446744073709551615)|极大整数值|

- 整形示例

```bash
# 创建表，一个是默认宽度的int，一个是指定宽度的int(5)，一个是指定宽度的int(13)
mysql> create table jangrui.t1 ( id1 int, id2 int(5), id3 int(13), primary key (id1,id2,id3) );
Query OK, 0 rows affected (0.01 sec)

# 查看 t1 表属性
mysql> desc jangrui.t1;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| id1   | int(11) | NO   | PRI | NULL    |       |
| id2   | int(5)  | NO   | PRI | NULL    |       |
| id3   | int(13) | NO   | PRI | NULL    |       |
+-------+---------+------+-----+---------+-------+
3 rows in set (0.00 sec)

# 向 t1 表中插入数据
mysql> insert into jangrui.t1 values(99,99,99);
Query OK, 1 row affected (0.01 sec)

mysql> insert into jangrui.t1 values(99999,99999,99999);
Query OK, 1 row affected (0.00 sec)

# 向 t1 表中插入大于指定宽度的数值，依然可以写入，说明宽度与存储大小或类型包含的值的范围无关。
mysql> insert into jangrui.t1 values(999991,999991,999991);
Query OK, 1 row affected (0.00 sec)

# int 类型不区分符号最大支持 2^32-1，也就是 2147483647。
mysql> insert into jangrui.t1 values(2147483647,2147483647,2147483647);
Query OK, 1 row affected (0.00 sec)

# 向 t1 表中插入大于类型包含值范围的数值，出现报错，说明只与存储大小或类型包含的值的范围相关。
mysql> insert into jangrui.t1 values(2147483648,2147483648,2147483648);
ERROR 1264 (22003): Out of range value for column 'id1' at row 1

# 查看 t1 表中的数据
mysql> select * from jangrui.t1;
+------------+------------+------------+
| id1        | id2        | id3        |
+------------+------------+------------+
|         99 |         99 |         99 |
|      99999 |      99999 |      99999 |
|     999991 |     999991 |     999991 |
| 2147483647 | 2147483647 | 2147483647 |
+------------+------------+------------+
4 rows in set (0.00 sec)

# 修改 t1 表类型
mysql> alter table jangrui.t1 modify id1 int unsigned, modify id2 int unsigned, modify id3 int unsigned;
Query OK, 4 rows affected (0.02 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> desc jangrui.t1;
+-------+------------------+------+-----+---------+-------+
| Field | Type             | Null | Key | Default | Extra |
+-------+------------------+------+-----+---------+-------+
| id1   | int(10) unsigned | NO   | PRI | NULL    |       |
| id2   | int(10) unsigned | NO   | PRI | NULL    |       |
| id3   | int(10) unsigned | NO   | PRI | NULL    |       |
+-------+------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)

# unsigned 无符号 int 类型最大支持 2^32，也就是 4294967295
mysql> insert into jangrui.t1 values (2147483648,2147483648,2147483648);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t1;
+------------+------------+------------+
| id1        | id2        | id3        |
+------------+------------+------------+
|         99 |         99 |         99 |
|      99999 |      99999 |      99999 |
|     999991 |     999991 |     999991 |
| 2147483647 | 2147483647 | 2147483647 |
| 2147483648 | 2147483648 | 2147483648 |
+------------+------------+------------+
5 rows in set (0.00 sec)
```

> `unsigned` 的作用：就是将数字类型无符号化，例如 int 型的范围：-2^31 ~ 2^31 - 1，而 int unsigned 的范围：0 ~ 2^32。尤其适合用在自增或者没有负数的情况。
>
> `zerofill` 的作用：当插入 mysql 中该字段的值的长度小于定义的长度时，会在数值前面补全相应数据的 0。

- 浮点型

|类型|大小|范围（有符号）|范围（无符号）|用途|
|-|-|-|-|-|
|`FLOAT`|4 字节|(-3.402823466E+38，-1.175494351E-38)，0，(1.175494351E-38，3.402823466351E+38)|0，(1.175494351E-38，3.402823466E+38)|单精度浮点数值|
|`DOUBLE`|8 字节|(-1.7976931348623157E+308，-2.2250738585072014E-308)，0，(2.2250738585072014E-308，1.7976931348623157E+308)|0，(2.2250738585072014 E-308，1.7976931348623157E+308)|双精度浮点数值|
|`DECIMAL`|对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2|依赖于M和D的值|依赖于M和D的值|小数值|

- 浮点型示例

```bash
# 创建表的三个字段分别为 float，double 和 decimal 参数表示一共显示 5 位，小数部分占 2 位
mysql> create table jangrui.t2 (
    -> id1 float(5,2),
    -> id2 double(5,2),
    -> id3 decimal(5,2),
    -> primary key (id1,id2,id3)
    -> );
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t2;
+-------+--------------+------+-----+---------+-------+
| Field | Type         | Null | Key | Default | Extra |
+-------+--------------+------+-----+---------+-------+
| id1   | float(5,2)   | NO   | PRI | NULL    |       |
| id2   | double(5,2)  | NO   | PRI | NULL    |       |
| id3   | decimal(5,2) | NO   | PRI | NULL    |       |
+-------+--------------+------+-----+---------+-------+
3 rows in set (0.01 sec)

# 向表中插入 1.23，结果正常
mysql> insert into jangrui.t2 values (1.23,1.23,1.23);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t2;
+------+------+------+
| id1  | id2  | id3  |
+------+------+------+
| 1.23 | 1.23 | 1.23 |
+------+------+------+
1 row in set (0.00 sec)

# 向表中插入 1.345，会发现 float 和 double 的 5 都被截断了，decimal 会四舍五入。
mysql> insert into jangrui.t2 values (1.345,1.345,1.345);
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> insert into jangrui.t2 values (1.671,1.671,1.671);
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> insert into jangrui.t2 values (1.682,1.682,1.682);
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> select * from jangrui.t2;
+------+------+------+
| id1  | id2  | id3  |
+------+------+------+
| 1.23 | 1.23 | 1.23 |
| 1.34 | 1.34 | 1.35 |
| 1.67 | 1.67 | 1.67 |
| 1.68 | 1.68 | 1.68 |
+------+------+------+
4 rows in set (0.00 sec)

# 去掉约束参数创建新表
mysql> create table jangrui.t3 ( id1 float, id2 double, id3 decimal, primary key (id1,id2,id3));
Query OK, 0 rows affected (0.01 sec)

# 分别插入 1.345
mysql> insert into jangrui.t3 values (1.345,1.345,1.345);
Query OK, 1 row affected, 1 warning (0.00 sec)

# 发现decimal默认值是(10,0)的整数
mysql> select * from jangrui.t3;
+-------+-------+-----+
| id1   | id2   | id3 |
+-------+-------+-----+
| 1.345 | 1.345 |   1 |
+-------+-------+-----+
1 row in set (0.00 sec)

# 当对小数位没有约束的时候，输入超长的小数，会发现 float 和 double 的区别
mysql> insert into jangrui.t3 values (1.2355555555555555555,1.2355555555555555555,1.2355555555555555555555);
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> select * from jangrui.t3;
+---------+--------------------+-----+
| id1     | id2                | id3 |
+---------+--------------------+-----+
| 1.23556 | 1.2355555555555555 |   1 |
|   1.345 |              1.345 |   1 |
+---------+--------------------+-----+
2 rows in set (0.00 sec)
```

## 日期和时间类型

|类型|大小(字节)|范围|格式|用途|
|-|-|-|-|-|
|`DATE`       |3|1000-01-01/9999-12-31    |YYYY-MM-DD	|日期值|
|`TIME`       |3|'-838:59:59'/'838:59:59' |HH:MM:SS	|时间值或持续时间|
|`YEAR`       |1|1901/2155                |YYYY	|年份值|
|`DATETIME`   |8|1000-01-01 00:00:00/9999-12-31 23:59:59  |YYYY-MM-DD HH:MM:SS    |混合日期和时间值|
|`TIMESTAMP`  |4|1970-01-01 00:00:00/2038 结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07 |YYYYMMDD HHMMSS	|混合日期和时间值，时间戳|

- data、time、datetime 示例

```bash
mysql> create table jangrui.t4 ( d date, t time, dt datetime, primary key(d,t,dt));
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t4;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| d     | date     | NO   | PRI | NULL    |       |
| t     | time     | NO   | PRI | NULL    |       |
| dt    | datetime | NO   | PRI | NULL    |       |
+-------+----------+------+-----+---------+-------+
3 rows in set (0.00 sec)

mysql> insert into jangrui.t4 values (now(), now(), now());
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> select * from jangrui.t4;
+------------+----------+---------------------+
| d          | t        | dt                  |
+------------+----------+---------------------+
| 2020-02-29 | 06:59:09 | 2020-02-29 06:59:09 |
+------------+----------+---------------------+
1 row in set (0.00 sec)
```

- timestamp 示例一

```bash
mysql> create table jangrui.t5 (id1 timestamp, primary key(id1));
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t5;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| id1   | timestamp | NO   | PRI | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+
1 row in set (0.00 sec)

# 插入数据 null，会自动插入当前时间的时间
mysql> insert into jangrui.t5 values (null);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t5;
+---------------------+
| id1                 |
+---------------------+
| 2020-02-29 07:01:24 |
+---------------------+
1 row in set (0.00 sec)

# 添加一列，默认值是'0000-00-00 00:00:00'
mysql> alter table jangrui.t5 add id2 timestamp;
ERROR 1067 (42000): Invalid default value for 'id2'

# 这是因为 sql_mode 中的 NO_ZEROR_DATE 导制的，在 strict mode 中不允许 '0000-00-00' 作为合法日期
mysql> select @@sql_mode;
+-------------------------------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                                                |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 将 NO_ZERO_DATE 改为 ALLOW_INVALID_DATES 临时允许
mysql> set sql_mode='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,ALLOW_INVALID_DATES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select @@sql_mode;
+--------------------------------------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                                                       |
+--------------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,ALLOW_INVALID_DATES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+--------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> alter table jangrui.t5 add id2 timestamp;
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select * from jangrui.t5;
+---------------------+---------------------+
| id1                 | id2                 |
+---------------------+---------------------+
| 2020-02-29 07:01:24 | 0000-00-00 00:00:00 |
+---------------------+---------------------+
1 row in set (0.00 sec)

# 手动修改新的列默认值为当前时间
mysql> alter table jangrui.t5 modify id2 timestamp default current_timestamp;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table jangrui.t5;
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                             |
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
| t5    | CREATE TABLE `t5` (
  `id1` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `id2` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id1`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 |
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> insert into jangrui.t5 values (null, null);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t5;
+---------------------+---------------------+
| id1                 | id2                 |
+---------------------+---------------------+
| 2020-02-29 07:01:24 | 0000-00-00 00:00:00 |
| 2020-02-29 07:29:18 | 2020-02-29 07:29:18 |
+---------------------+---------------------+
2 rows in set (0.00 sec)
```

> sql_mode 永久设置：my.cnf 配置文件 [mysqld] 中添加 `sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION`，重启生效。

- timestamp 示例二

```bash
mysql> create table jangrui.t6 (t1 timestamp, primary key(t1));
Query OK, 0 rows affected (0.02 sec)

mysql> desc jangrui.t6;
+-------+-----------+------+-----+-------------------+-----------------------------+
| Field | Type      | Null | Key | Default           | Extra                       |
+-------+-----------+------+-----+-------------------+-----------------------------+
| t1    | timestamp | NO   | PRI | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------+-----------+------+-----+-------------------+-----------------------------+
1 row in set (0.00 sec)

mysql> insert into jangrui.t6 values (19700101080001);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t6;
+---------------------+
| t1                  |
+---------------------+
| 1970-01-01 08:00:01 |
+---------------------+
1 row in set (0.00 sec)

# timestamp 时间的下限是 19700101080001
mysql> insert into jangrui.t6 values (19700101080000);
ERROR 1292 (22007): Incorrect datetime value: '19700101080000' for column 't1' at row 1
mysql> insert into jangrui.t6 values ('2038-01-19 11:14:07');
Query OK, 1 row affected (0.01 sec)

# timestamp 时间的上限是 2038-01-19 11:14:07
mysql> insert into jangrui.t6 values ('2038-01-19 11:14:08');
ERROR 1292 (22007): Incorrect datetime value: '2038-01-19 11:14:08' for column 't1' at row 1
mysql>
```

- year 示例

```bash
mysql> create table jangrui.t7 (y year,primary key(y));
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t7;
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
| y     | year(4) | NO   | PRI | NULL    |       |
+-------+---------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> insert into jangrui.t7 values (2020);
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t7;
+------+
| y    |
+------+
| 2020 |
+------+
1 row in set (0.00 sec)
```

- datetime 示例

```bash
mysql> create table jangrui.t8 (dt datetime,primary key(dt));
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t8;
+-------+----------+------+-----+---------+-------+
| Field | Type     | Null | Key | Default | Extra |
+-------+----------+------+-----+---------+-------+
| dt    | datetime | NO   | PRI | NULL    |       |
+-------+----------+------+-----+---------+-------+
1 row in set (0.00 sec)

mysql> insert into jangrui.t8 values ('2020-3-2 12:20:00');
Query OK, 1 row affected (0.00 sec)

mysql> insert into jangrui.t8 values ('2020/3/2 12+20+10');
Query OK, 1 row affected (0.00 sec)

mysql> insert into jangrui.t8 values ('20200302122015');
Query OK, 1 row affected (0.01 sec)

mysql> insert into jangrui.t8 values (20200302122020);
Query OK, 1 row affected (0.01 sec)

mysql> select * from jangrui.t8;
+---------------------+
| dt                  |
+---------------------+
| 2020-03-02 12:20:00 |
| 2020-03-02 12:20:10 |
| 2020-03-02 12:20:15 |
| 2020-03-02 12:20:20 |
+---------------------+
4 rows in set (0.00 sec)
```

## 字符串类型

|类型|大小|用途|
|-|-|-|
|`CHAR`         |0-255 字节  	    |定长字符串|
|`VARCHAR`      |0-65535 字节	    |变长字符串|
|`TINYBLOB`     |0-255 字节	        |不超过 255 个字符的二进制字符串|
|`BLOB`         |0-65535 字节	    |二进制形式的长文本数据|
|`MEDIUMBLOB`   |0-16777215 字节    |二进制形式的中等长度文本数据|
|`LONGBLOB`     |0-4294967295 字节  |二进制形式的极大文本数据|
|`TINYTEXT`     |0-255 字节	        |短文本字符串|
|`TEXT`         |0-65535 字节	    |长文本数据|
|`MEDIUMTEXT`   |0-16777215 字节    |中等长度文本数据|
|`LONGTEXT`     |0-4294967295 字节  |极大文本数据|

CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

CHAR列的长度固定为创建表是声明的长度,范围(0-255);而VARCHAR的值是可变长字符串范围(0-65535)。

```bash
mysql> create table jangrui.t9 (v varchar(4), c char(4),primary key(v,c));
Query OK, 0 rows affected (0.04 sec)

mysql> desc jangrui.t9;
+-------+------------+------+-----+---------+-------+
| Field | Type       | Null | Key | Default | Extra |
+-------+------------+------+-----+---------+-------+
| v     | varchar(4) | NO   | PRI | NULL    |       |
| c     | char(4)    | NO   | PRI | NULL    |       |
+-------+------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

mysql> insert into jangrui.t9 values ('ab  ', 'ab  ');
Query OK, 1 row affected (0.00 sec)

# 在检索的时候 char 数据类型会去掉空格
mysql> select * from jangrui.t9;
+------+----+
| v    | c  |
+------+----+
| ab   | ab |
+------+----+
1 row in set (0.00 sec)

# 来看看对查询结果计算的长度
mysql> select length(v), length(c) from jangrui.t9;
+-----------+-----------+
| length(v) | length(c) |
+-----------+-----------+
|         4 |         2 |
+-----------+-----------+
1 row in set (0.00 sec)

# 给结果拼上一个加号会更清楚
mysql> select concat(v,'+'), concat(c,'+') from jangrui.t9;
+---------------+---------------+
| concat(v,'+') | concat(c,'+') |
+---------------+---------------+
| ab  +         | ab+           |
+---------------+---------------+
1 row in set (0.00 sec)

# 当存储的长度超出定义的长度，会报错，这是因为 sql_mode 中 STRICT_TRANS_TABLES 严格检查起作用
mysql> insert into jangrui.t9 values ('zxvbn','zxvbn');
ERROR 1406 (22001): Data too long for column 'v' at row 1

# 临时关闭 sql_mode
mysql> set sql_mode='';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> insert into jangrui.t9 values ('zxvbn','zxvbn');
Query OK, 1 row affected, 1 warning (0.00 sec)

# 当存储的长度超出定义的长度，会截断
mysql> select * from jangrui.t9;
+------+------+
| v    | c    |
+------+------+
| ab   | ab   |
| zxvb | zxvb |
+------+------+
2 rows in set (0.00 sec)
```

BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。

有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。

## 枚举类型

ENUM 中文名称叫枚举类型，它的值范围需要在创建表时通过枚举方式显示。ENUM只允许从值集合中选取单个值，而不能一次取多个值。

SET 和 ENUM 非常相似，也是一个字符串对象，里面可以包含 0-64 个成员。根据成员的不同，存储上也有所不同。set 类型可以允许值集合中任意选择 1 或多个元素进行组合。对超出范围的内容将不允许注入，而对重复的值将进行自动去重。

|类型|大小|用途|
|-|-|-|
|`ENUM`|对1-255个成员的枚举需要1个字节存储;对于255-65535个成员，需要2个字节存储;最多允许65535个成员。|单选：选择性别|
|`SET`|1-8个成员的集合，占1个字节;9-16个成员的集合，占2个字节;17-24个成员的集合，占3个字节;25-32个成员的集合，占4个字节;33-64个成员的集合，占8个字节;|多选：兴趣爱好|

```bash
mysql> create table jangrui.t10 ( name char(20), gender enum('male','female'), primary key(name) );
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t10;
+--------+-----------------------+------+-----+---------+-------+
| Field  | Type                  | Null | Key | Default | Extra |
+--------+-----------------------+------+-----+---------+-------+
| name   | char(20)              | NO   | PRI | NULL    |       |
| gender | enum('male','female') | YES  |     | NULL    |       |
+--------+-----------------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

# 选择 `enum('female','male')` 中的一项作为 gender 的值，可以正常插入
mysql> insert into jangrui.t10 values ('yangyang', 'male');
Query OK, 1 row affected (0.01 sec)

mysql> insert into jangrui.t10 values ('hanhan', 'female');
Query OK, 1 row affected (0.01 sec)

# 不能同时插入 'male,female' 两个值，也不能插入不属于 'male,female' 的值
mysql> insert into jangrui.t10 values ('bingbing', 'male,female');
ERROR 1265 (01000): Data truncated for column 'gender' at row 1

mysql> insert into jangrui.t10 values ('dingding', 'dangdang');
ERROR 1265 (01000): Data truncated for column 'gender' at row 1

mysql> create table jangrui.t11 ( name char(20), hobby set('painting', 'reading', 'singing', 'gaming'), primary key(name) );
Query OK, 0 rows affected (0.01 sec)

mysql> desc jangrui.t11;
+-------+----------------------------------------------+------+-----+---------+-------+
| Field | Type                                         | Null | Key | Default | Extra |
+-------+----------------------------------------------+------+-----+---------+-------+
| name  | char(20)                                     | NO   | PRI | NULL    |       |
| hobby | set('painting','reading','singing','gaming') | YES  |     | NULL    |       |
+-------+----------------------------------------------+------+-----+---------+-------+
2 rows in set (0.00 sec)

# 可以任意选择 `set('painting','reading','singing','gaming')` 中的项，并自带去重功能
mysql> insert into jangrui.t11 values ('xiaoming', 'gaming,singing,reading');
Query OK, 1 row affected (0.00 sec)

mysql> select * from jangrui.t11;
+----------+------------------------+
| name     | hobby                  |
+----------+------------------------+
| xiaoming | reading,singing,gaming |
+----------+------------------------+
1 row in set (0.00 sec)

# 不能选择不属于 `set('painting','reading','singing','gaming')` 中的项，
mysql> insert into jangrui.t11 values ('xiaoming', 'gaming,singing,dancing');
ERROR 1265 (01000): Data truncated for column 'hobby' at row 1
```

> 参考：[mysql支持的数据类型](https://www.cnblogs.com/aizhinong/p/11720261.html)
