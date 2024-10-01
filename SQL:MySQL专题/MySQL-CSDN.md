[toc]

# SQL简述

Structure Query Language(结构化查询语言)简称SQL，它被美国国家标准局(ANSI)确定为关系型数据库语言的美国标准，后被国际化标准组织(ISO)采纳为关系数据库语言的国际标准。数据库管理系统可以通过SQL管理数据库；定义和操作数据，维护数据的完整性和安全性。

> SQL的优点
>
> 1、简单易学，具有很强的操作性
> 2、绝大多数重要的数据库管理系统均支持SQL
> 3、<u>***高度非过程化***</u>；用[SQL操作](https://so.csdn.net/so/search?q=SQL操作&spm=1001.2101.3001.7020)数据库时大部分的工作由DBMS自动完成

> SQL的分类
>
> 1、DDL(Data Definition Language) 数据定义语言，用来操作数据库、表、列等； 常用语句：CREATE、 ALTER、DROP
> 2、DML(Data Manipulation Language) 数据操作语言，用来操作数据库中表里的数据；常用语句：INSERT、 UPDATE、 DELETE
> 3、DCL(Data Control Language) 数据控制语言，用来操作访问权限和安全级别； 常用语句：GRANT、DENY
> 4、DQL(Data Query Language) 数据查询语言，用来查询数据 常用语句：SELECT 

# 数据库的三大范式

1、第一范式(1NF)是指数据库表的每一列都是不可分割的基本数据线；也就是说：每列的值具有原子性，不可再分割。
2、第二范式(2NF)是在第一范式(1NF)的基础上建立起来得，满足第二范式(2NF)必须先满足第一范式(1NF)。如果表是单主键，那么主键以外的列必须完全依赖于主键；如果表是复合主键，那么主键以外的列必须完全依赖于主键，不能仅依赖主键的一部分。
3、第三范式(3NF)是在第二范式的基础上建立起来的，即满足第三范式必须要先满足第二范式。第三范式(3NF)要求：表中的非主键列必须和主键直接相关而不能间接相关；也就是说：非主键列之间不能相关依赖。

# 数据库的数据类型

使用MySQL数据库存储数据时，不同的数据类型决定了 MySQL存储数据方式的不同。为此，MySQL数据库提供了多种数据类型，其中包括整数类型、浮点数类型、定点 数类型、日期和时间类型、字符串类型、二进制…等等数据类型。

## 整数类型

根据数值取值范围的不同MySQL 中的整数类型可分为5种，分别是TINYINT、SMALUNT、MEDIUMINT、INT和 BIGINT。下图列举了 MySQL不同整数类型所对应的字节大小和取值范围而最常用的为INT类型.  (可以定义无符号 [ unsigned ] )

| 数据类型  | 字节数 | 无符号数的取值范围     | 有符号的取值范围                         |
| --------- | ------ | ---------------------- | ---------------------------------------- |
| TINYINT   | 1      | 0~255                  | -128~127                                 |
| SMALLINT  | 2      | 0~65535                | -32768~32767                             |
| MEDIUMINT | 3      | 0~16777215             | -8388608~8388607                         |
| INT       | 4      | 0~4294967295           | -2147483648~2147483647                   |
| BIGINT    | 8      | 0~18446744073709551615 | -9223372036854775808~9223372036854775807 |

## 浮点数类型和定点类型

在MySQL数据库中使用浮点数和定点数来存储小数。浮点数的类型有两种：单精度浮点数类型（FLOAT)和双精度浮点数类型（DOUBLE)。而定点数类型只有一种即DECIMAL类型。下图列举了 MySQL中浮点数和定点数类型所对应的字节大小及其取值范围：

| 数据类型                                                     | 字节数 | 有符号的取值范围                                 | 无符号的取值范围                                   |
| ------------------------------------------------------------ | ------ | ------------------------------------------------ | -------------------------------------------------- |
| FLOAT                                                        | 4      | -3.402823466E+38~-1.175494351E-38                | 0和1.175494351E-38~3.402823466E+38                 |
| DOUBLE                                                       | 8      | -1.7976931348623157E+308~2.2250738585072014E-308 | 0和2.2250738585072014E-308~1.7976931348623157E+308 |
| DECIMAL(M,D)                                                 | M+2    | -1.7976931348623157E+308~2.2250738585072014E-308 | 0和2.2250738585072014E-308~1.7976931348623157E+308 |
| 从上图中可以看出：DECIMAL类型的取值范围与DOUBLE类型相同。但是，请注意：DECIMAL类型的有效取值范围是由M和D决定的。其中，M表示的是数据的长 度，D表示的是小数点后的长度。比如，将数据类型为DECIMAL(6,2)的数据6.5243 插入数据库后显示的结果为6.52 |        |                                                  |                                                    |

## 字符串类型

在MySQL中常用CHAR 和 VARCHAR 表示字符串。两者不同的是：VARCHAR存储可变长度的字符串。
**当数据为CHAR(M)类型时，不管插入值的长度是实际是多少它所占用的存储空间都是M个字节；而VARCHAR(M)所对应的数据所占用的字节数为实际长度加1**

| 插入值 | CHAR(3) | 存储需求 | VARCHAR(3) | 存储需求 |
| ------ | ------- | -------- | ---------- | -------- |
| ''     | ''      | 3字节    | ''         | 1字节    |
| 'a'    | 'a'     | 3字节    | 'a'        | 2字节    |
| 'ab'   | 'ab'    | 3字节    | 'ab'       | 3字节    |
| 'abc'  | 'ab'    | 3字节    | 'abc'      | 4字节    |
| 'abcd' | 'ab'    | 3字节    | 'abc'      | 4字节    |

## 文本类型

文本类型用于表示大文本数据，例如，文章内容、评论、详情等，它的类型分为如下4种：

| 数据类型   | 存储范围                |
| ---------- | ----------------------- |
| TINYTEXT   | 0~255字节 2^8-1         |
| TEXT       | 0~65535字节 2^16-1      |
| MEDIUMTEXT | 0~16777215字节 2^24-1   |
| LONGTEXT   | 0~4294967295字节 2^32-1 |

## 日期与时间类型

MySQL提供的表示日期和时间的数据类型分别是 ：YEAR、DATE、TIME、DATETIME 和 TIMESTAMP。下图列举了日期和时间数据类型所对应的字节数、取值范围、日期格式以及零值：

| 数据类型  | 字节数 | 取值范围                                | 日期格式            | 零值                |
| --------- | ------ | --------------------------------------- | ------------------- | ------------------- |
| YEAR      | 1      | 1901~2155                               | YYYY                | 0000                |
| DATE      | 4      | 1000-01-01~9999-12-31                   | YYYY-MM-DD          | 0000-00-00          |
| TIME      | 3      | -838:59:59~838:59:59                    | HH:MM:SS            | 00:00:00            |
| DATETIME  | 8      | 1000-01-01 00:00:00~9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 0000-00-00 00:00:00 |
| TIMESTAMP | 4      | 1970-01-01 00:00:01~2038-01-19 03:14:07 | YYYY-MM-DD HH:MM:SS | 0000-00-00 00:00:00 |

### YEAR类型

YEAR类型用于表示年份，在MySQL中，可以使用以下三种格式指定YEAR类型 的值。

1. 使用4位字符串或数字表示，范围为’1901’—'2155’或1901—2155。例如，输入 ‘2019’或2019插入到数据库中的值均为2019。
2. 使用两位字符串表示，范围为’00’—‘99’。其中，‘00’—'69’范围的值会被转换为 2000—2069范围的YEAR值，‘70’—'99’范围的值会被转换为1970—1999范围的YEAR 值。例如，输入’19’插入到数据库中的值为2019。
3. 使用两位数字表示，范围为1—99。其中，1—69范围的值会被转换为2001— 2069范围的YEAR值，70—99范围的值会被转换为1970—1999范围的YEAR值。例 如，输入19插入到数据库中的值为2019。
4. 请注意：当使用YEAR类型时，一定要区分’0’和0。因为字符串格式的’0’表示的YEAR值是2000而数字格式的0表示的YEAR值是0000。

### TIME类型

TIME类型用于表示时间值，它的显示形式一般为HH:MM:SS，其中，HH表示小时， MM表示分,SS表示秒。在MySQL中，可以使用以下3种格式指定TIME类型的值。

1. 以’D HH:MM:SS’字符串格式表示。其中，D表示日可取0—34之间的值, 插入数据时，小时的值等于(DX24+HH)。例如，输入’2 11:30:50’插入数据库中的日期为59:30:50。
2. 以’HHMMSS’字符串格式或者HHMMSS数字格式表示。 例如，输入’115454’或115454,插入数据库中的日期为11:54:54
3. 使用CURRENT_TIME或NOW()输入当前系统时间。

### DATETIME类型

DATETIME类型用于表示日期和时间，它的显示形式为’YYYY-MM-DD HH: MM:SS’，其中，YYYY表示年，MM表示月，DD表示日，HH表示小时，MM表示分，SS 表示秒。在MySQL中，可以使用以下4种格式指定DATETIME类型的值。
以’YYYY-MM-DD HH:MM:SS’或者’YYYYMMDDHHMMSS’字符串格式表示的日期和时间，取值范围为’1000-01-01 00:00:00’—‘9999-12-3 23:59:59’。例如，输入’2019-01-22 09:01:23’或 ‘20140122_0_90123’插入数据库中的 DATETIME 值都为 2019-01-22 09:01:23。

1. 以’YY-MM-DD HH:MM:SS’或者’YYMMDDHHMMSS’字符串格式表示的日期和时间，其中YY表示年，取值范围为’00’—‘99’。与DATE类型中的YY相同，‘00’— '69’范围的值会被转换为2000—2069范围的值，‘70’—'99’范围的值会被转换为1970—1999范围的值。
2. 以YYYYMMDDHHMMSS或者YYMMDDHHMMSS数字格式表示的日期 和时间。例如，插入20190122090123或者190122090123,插入数据库中的DATETIME值都 为 2019-01-22 09:01:23。
3. 使用NOW来输入当前系统的日期和时间。

### TIMESTAMP类型

TIMESTAMP类型用于表示日期和时间，它的显示形式与DATETIME相同但取值范围比DATETIME小。在此，介绍几种TIMESTAMP类型与DATATIME类型不同的形式：

1. 使用CURRENT_TIMESTAMP输入系统当前日期和时间。
2. 输入NULL时系统会输入系统当前日期和时间。
3. 无任何输入时系统会输入系统当前日期和时间。

### 二进制类型

在MySQL中常用BLOB存储二进制类型的数据，例如：图片、PDF文档等。BLOB类型分为如下四种：

| 数据类型   | 存储范围         |
| ---------- | ---------------- |
| TINYBLOB   | 0~255字节        |
| BLOB       | 0~65535字节      |
| MEDIUMBLOB | 0~16777215字节   |
| LONGBLOB   | 0~4294967295字节 |

# 数据库的基本操作

## 数据库的基本操作

MySQL安装完成后，要想将数据存储到数据库的表中，首先要创建一个数据库。创 建数据库就是在数据库系统中划分一块空间存储数据，语法如下：

`create database 数据库名称;`

创建一个叫db1的数据库MySQL命令:

```mysql
# 创建一个叫db1的数据库
 dacreate database db1;
# 运行效果
Query OK, 1 row affected (0.01 sec)

# 创建数据库后查看该数据库基本信息MySQL命令：
show create database db1;
# 运行效果
+----------+-------------------------------------------------------------------------------------------------------------------------------+
| Database | Create Database                                                                                                               |
+----------+-------------------------------------------------------------------------------------------------------------------------------+
| db1      | CREATE DATABASE `db1` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */ |
+----------+-------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 删除mysql数据库的命令
drop database db1;
# 运行效果
Query OK, 0 rows affected (0.01 sec)

#查询出MySQL中所有的数据库MySQL命令：
show databases;
# 运行效果
+--------------------+
| Database           |
+--------------------+
| db01               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

# 将数据库的字符集修改为gbk MySQL命令：
alter database db1 character set gbk;
# 运行效果
mysql> alter database db01 character set gbk;
Query OK, 1 row affected (0.01 sec)

mysql> show create database db01;
+----------+-------------------------------------------------------------------------------------------------+
| Database | Create Database                                                                                 |
+----------+-------------------------------------------------------------------------------------------------+
| db01     | CREATE DATABASE `db01` /*!40100 DEFAULT CHARACTER SET gbk */ /*!80016 DEFAULT ENCRYPTION='N' */ |
+----------+-------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

# 切换数据库 MySQL命令：
use db02;
# 运行效果
Database changed

# 查看当前使用的数据库MySQL命令:
select database();
# 运行效果
+------------+
| database() |
+------------+
| db02       |
+------------+
1 row in set (0.00 sec)
```

# 数据表的基本操作

数据库创建成功后可在该数据库中创建数据表(简称为表)存储数据。请注意：在操作数据表之前应使用“USE 数据库名;” 指定操作是在哪个数据库中进行相关操作，否则会抛出“No database selected”错误。

语法如下：

```mysql
 create table 表名(
         字段1 字段类型,
         字段2 字段类型,
         …
         字段n 字段类型
);
```

## 创建数据表

示例：创建学生表 MySQL命令：

```mysql
 create table student(
 id int,
 name varchar(20),
 gender varchar(10),
 birthday date
 );
 
 # 运行效果
 Query OK, 0 rows affected (0.01 sec)
```

## 查看数据表

示例：查看当前数据库中所有表 MySQL命令：

`show tables;`

运行效果如下

`+----------------+`
`| Tables_in_db01 |`
`+----------------+`
`| student        |`
`+----------------+`
`1 row in set (0.01 sec)`

示例：查看表的基本信息 MySQL命令：

`show create table student;`

运行效果:

```mysql
+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table   | Create Table                                                                                                                                                                                     |
+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| student | CREATE TABLE `student` (
  `id` int DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `gender` varchar(10) DEFAULT NULL,
  `birthday` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 |
+---------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

示例：查看表的字段信息 MySQL命令：

```mysql
desc student;

# 运行效果
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int         | YES  |     | NULL    |       |
| name     | varchar(20) | YES  |     | NULL    |       |
| gender   | varchar(10) | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
4 rows in set (0.01 sec)
```

## 修改数据表

有时，希望对表中的某些信息进行修改，例如：修改表名、修改字段名、修改字段 数据类型…等等。在MySQL中使用alter table修改数据表.

```mysql
# 示例：修改表名 MySQL命令：
alter table student rename to stu;
# 运行效果
Query OK, 0 rows affected (0.01 sec)

# 示例：修改字段名 MySQL命令：
alter table stu change name sname varchar(10);
# running result
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

# 示例：修改字段数据类型 MySQL命令：
alter table stu modify sname int;
# running result
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

#示例：增加字段 MySQL命令：
alter table stu add address varchar(50);
# running result
Query OK, 0 rows affected (0.01 sec)
mysql> desc students
Records: 0  Duplicates: 0  Warnings: 0
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int         | YES  |     | NULL    |       |
| stu_name | int         | YES  |     | NULL    |       |
| gender   | varchar(10) | YES  |     | NULL    |       |
| birthday | date        | YES  |     | NULL    |       |
| address  | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)

# 示例：删除字段 MySQL命令：
alter table stu drop address;
# running result
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

## 删除数据表

`drop table 表名;`

示例: `drop table stu;`

running result 

`Query OK, 0 rows affected (0.01 sec)`

# 数据表的约束

为防止错误的数据被插入到数据表，MySQL中定义了一些维护数据库完整性的规则；这些规则常称为表的约束。常见约束如下：

| 约束条件                                                     | 说明                             |
| ------------------------------------------------------------ | -------------------------------- |
| PRIMARY KEY                                                  | 主键约束用于唯一标识对应的记录   |
| FOREIGN KEY                                                  | 外键约束                         |
| NOT NULL                                                     | 非空约束                         |
| UNIQUE                                                       | 唯一性约束                       |
| DEFAULT                                                      | 默认值约束，用于设置字段的默认值 |
| 以上五种约束条件针对表中字段进行限制从而保证数据表中数据的正确性和唯一性。换句话说，表的约束实际上就是表中数据的限制条件。 |                                  |

## 主键约束

主键约束即primary key用于唯一的标识表中的每一行。被标识为主键的数据在表中是<u>***唯一的且其值不能为空***</u>。这点类似于我们每个人都有一个身份证号，并且这个身份证号是唯一的。

主键约束基本语法：

`字段名 数据类型 primary key;`

> **设置主键约束(primary key)的第一种方式**
>
> ```mysql
> # 示例：MySQL命令：
> create table student(
> id int primary key,
> name varchar(20)
> );
> 
> # running result
> Query OK, 0 rows affected (0.01 sec)
> ```
>
> **设置主键约束(primary key)的第二种方式**
>
> ```mysql
> # 示例：MySQL命令：
> create table student01(
> id int,
> name varchar(20),
> primary key(id)
> );
> # 运行效果展示：
> Query OK, 0 rows affected (0.01 sec)
> mysql> desc student
>     -> ;
> +-------+-------------+------+-----+---------+-------+
> | Field | Type        | Null | Key | Default | Extra |
> +-------+-------------+------+-----+---------+-------+
> | id    | int         | NO   | PRI | NULL    |       |
> | name  | varchar(20) | YES  |     | NULL    |       |
> +-------+-------------+------+-----+---------+-------+
> ```

## 非空约束

非空约束即 NOT NULL指的是字段的值不能为空，基本的语法格式如下所示：

`字段名 数据类型 NOT NULL;`

```mysql
#示例：MySQL命令：
create table student02(
id int
name varchar(20) not null
);
# 运行效果展示：
Query OK, 0 rows affected (0.01 sec)
mysql> desc student02;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | NO   | PRI | NULL    |       |
| name  | varchar(20) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

## 默认值约束

默认值约束即DEFAULT用于给数据表中的字段指定默认值，即当在表中插入一条新记录时若未给该字段赋值，那么，数据库系统会自动为这个字段插入默认值；其基本的语法格式如下所示：

`字段名 数据类型 DEFAULT 默认值；`

```mysql
# 示例：MySQL命令：
create table student03(
id int,
name varchar(20),
gender varchar(10) default 'male'
);
# 运行效果展示：
Query OK, 0 rows affected (0.01 sec)
mysql> desc student03;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int         | YES  |     | NULL    |       |
| name   | varchar(20) | YES  |     | NULL    |       |
| gender | varchar(10) | YES  |     | male    |       |
+--------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

## 唯一性约束

唯一性约束即UNIQUE用于保证数据表中字段的唯一性，即表中字段的值不能重复出现，其基本的语法格式如下所示：

`字段名 数据类型 UNIQUE;`

```mysql
# 示例：MySQL命令：
create table student04(
id int,
name varchar(20) unique
);
# 运行效果展示：
Query OK, 0 rows affected (0.01 sec)
mysql> desc student04;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int         | YES  |     | NULL    |       |
| name  | varchar(20) | YES  | UNI | NULL    |       |
+-------+-------------+------+-----+---------+-------+
2 rows in set (0.00 sec)
```

## 外键约束(待补充)

外键约束即FOREIGN KEY常用于多张表之间的约束。基本语法如下：

```mysql
-- 在创建数据表时语法如下：
CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段)
-- 将创建数据表创建后语法如下：
ALTER TABLE 从表名 ADD CONSTRAINT 外键名 FOREIGN KEY (从表外键字段) REFERENCES 主表 (主键字段);

```





# 数据表插入数据

在MySQL通过INSERT语句向数据表中插入数据。在此，我们先准备一张学生表，代码如下：

```mysql
 create table student(
 id int,
 name varchar(30),
 age int,
 gender varchar(30)
 );

```

## 为表中所有字段插入数据

每个字段与其值是严格一一对应的。也就是说：每个值、值的顺序、值的类型必须与对应的字段相匹配。但是，各字段也无须与其在表中定义的顺序一致，它们只要与 VALUES中值的顺序一致即可。
语法如下：

`INSERT INTO 表名（字段名1,字段名2,...) VALUES (值 1,值 2,...);`

示例：向学生表中插入一条学生信息 MySQL命令：

`insert into student (id,name,age,gender) values (1,'bob',16,'male');`

运行效果展示：

`Query OK, 1 row affected (0.01 sec)`

```mysql
mysql> select * from student;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | bob  |   16 | male   |
+------+------+------+--------+
1 row in set (0.00 sec)
```



## 为表中指定字段插入数据

`INSERT INTO 表名（字段名1,字段名2,...) VALUES (值 1,值 2,...);`

插入数据的方法基本和为表中所有字段插入数据，一样，只是需要插入的字段由你自己指定

## 同时插入多条记录

语法如下:

`INSERT INTO 表名 [(字段名1,字段名2,...)]VALUES (值 1,值 2,…),(值 1,值 2,…),...;`

在该方式中：(字段名1,字段名2,…)是可选的，它用于指定插入的字段名；(值 1,值 2,…),(值 1,值 2,…)表示要插入的记录，该记录可有多条并且每条记录之间用逗号隔开。

示例：向学生表中插入多条学生信息 MySQL命令：

`insert into student (id,name,age,gender) values (2,'lucy',17,'female'),(3,'jack',19,'male'),(4,'tom',18,'male');`

运行效果展示：

```mysql
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
mysql> select * from student;
+------+------+------+--------+
| id   | name | age  | gender |
+------+------+------+--------+
|    1 | bob  |   16 | male   |
|    2 | lucy |   17 | female |
|    3 | jack |   19 | male   |
|    4 | tom  |   18 | male   |
+------+------+------+--------+
4 rows in set (0.00 sec)
```

# 更新数据库

在MySQL通过UPDATE语句更新数据表中的数据。在此，我们将就用上节中的student学生表









