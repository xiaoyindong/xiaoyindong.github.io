## 1. 启动

```s
service mysql start | stop | restart | status
```

## 2. 登录

用户名为```root```时
```s
mysql -uroot -p
```

输入密码完成登录。

## 3. 创建数据库

创建名字为```test_db```的数据库

```s
CREATE DATABASE test_db;
```

1.判断是否已经存在。如不存在则创建

```s
CREATE DATABASE IF NOT EXISTS test_db;
```

2.指定其默认字符集为 utf8，默认校对规则为 utf8_chinese_ci

```s
CREATE DATABASE IF NOT EXISTS test_db DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_chinese_ci;
```
3.查看数据库创建信息

```s
SHOW CREATE DATABASE test_db;
```

4.查看所有数据库

```s
SHOW DATABASES;
```

## 4. 修改数据库

数据库中只能对数据库使用的字符集和校对规则进行修改，数据库的这些特性都储存在 ```db.opt``` 文件中。可以使用 ```ALTER``` ```DATABASE``` 来修改已经被创建或者存在的数据库的相关参数。

```s
ALTER DATABASE test_db DEFAULT CHARACTER SET gb2312 DEFAULT COLLATE gb2312_chinese_ci;
```

```ALTER``` ```DATABASE``` 用于更改数据库的全局特性。使用 ```ALTER``` ```DATABASE``` 需要获得数据库 ```ALTER``` 权限。```CHARACTER``` ```SET``` 子句用于更改默认的数据库字符集。

## 5. 删除数据库

```s
DROP DATABASE test_db;
# or
DROP DATABASE IF EXISTS test_db;
```

## 6. 选择数据库

```s
USE test_db;
```

## 7. 数据类型

数据库中的每个列都应该有适当的数据类型，用于限制或允许该列中存储的数据。如果使用错误的数据类型可能会严重影响应用程序的功能和性能，所以在设计表时，应该特别重视数据列所用的数据类型。更改包含数据的列不是一件小事，这样做可能会导致数据丢失。因此，在创建表时必须为每个列设置正确的数据类型和长度。

MySQL 的数据类型有大概可以分为 5 种，分别是整数类型、浮点数类型和定点数类型、日期和时间类型、字符串类型、二进制类型等。

1.数值类型

整数：```TINYINT```、```SMALLINT```、```MEDIUMINT```、```INT```、```BIGINT```。

| 类型名称 | 说明 | 存储需求 |
| --- | --- | --- |
| TINYINT | 很小的整数 | 1个字节 |
| SMALLINT | 小的整数 | 2个宇节 |
| MEDIUMINT | 中等大小的整数 | 3个字节 |
| INT (INTEGHR) | 普通大小的整数 | 4个字节 |
| BIGINT | 大整数 | 8个字节 |

| 类型名称 | 说明 | 存储需求 |
| --- | --- | --- |
| TINYINT | -128〜127 | 0 〜255 |
| SMALLINT | -32768〜32767 | 0〜65535 |
| MEDIUMINT | -8388608〜8388607 | 0〜16777215 |
| INT (INTEGER) | -2147483648〜2147483647 | 0〜4294967295 |
| BIGINT | -9223372036854775808〜9223372036854775807 | 0〜18446744073709551615 |

浮点数：```FLOAT```和 ```DOUBLE```，定点数类型为 ```DECIMAL```。

| 类型名称 | 说明 | 存储需求 |
| -- | -- | -- |
| FLOAT | 单精度浮点数 | 4 个字节 |
| DOUBLE | 双精度浮点数 | 8 个字节 |
| DECIMAL (M, D)，DEC | 压缩的“严格”定点数 | M+2 个字节 |

2.日期/时间类型

```YEAR```、```TIME```、```DATE```、```DATETIME``` 和``` TIMESTAMP```。

| 类型名称 | 说明 | 存储需求 |
| -- | -- | -- |
| YEAR | YYYY | 1901 ~ 2155 | 1 个字节 |
| TIME | HH:MM:SS | -838:59:59 ~ 838:59:59 | 3 个字节 |
| DATE | YYYY-MM-DD | 1000-01-01 ~ 9999-12-3 | 3 个字节 |
| DATETIME | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59 | 8 个字节 |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 1980-01-01 00:00:01 UTC ~ 2040-01-19 03:14:07 UTC | 4 个字节 |

3.字符串类型

```CHAR```、```VARCHAR```、```BINARY```、```VARBINARY```、```BLOB```、```TEXT```、```ENUM``` 和 ```SET``` 等。

| 类型名称 | 说明 | 存储需求 |
| -- | -- | -- |
| CHAR(M) | 固定长度非二进制字符串 | M 字节，1<=M<=255 |
| VARCHAR(M) | 变长非二进制字符串 | L+1字节，在此，L< = M和 1<=M<=255 |
| TINYTEXT | 非常小的非二进制字符串 | L+1字节，在此，L<2^8 |
| TEXT | 小的非二进制字符串 | L+2字节，在此，L<2^16 |
| MEDIUMTEXT | 中等大小的非二进制字符串 | L+3字节，在此，L<2^24 |
| LONGTEXT | 大的非二进制字符串 | L+4字节，在此，L<2^32 |
| ENUM | 枚举类型，只能有一个枚举字符串值 | 1或2个字节，取决于枚举值的数目 (最大值为65535) |
| SET | 一个设置，字符串对象可以有零个或 多个SET成员 | 1、2、3、4或8个字节，取决于集合 成员的数量（最多64个成员） |

4.二进制类型

```BIT```、```BINARY```、```VARBINARY```、```TINYBLOB```、```BLOB```、```MEDIUMBLOB``` 和 ```LONGBLOB```。

| 类型名称 | 说明 | 存储需求 |
| -- | -- | -- |
| BIT(M) | 位字段类型 | 大约 (M+7)/8 字节 |
| BINARY(M) | 固定长度二进制字符串 | M 字节 |
| VARBINARY (M) | 可变长度二进制字符串 | M+1 字节 |
| TINYBLOB (M) | 非常小的BLOB | L+1 字节，在此，L<2^8 |
| BLOB (M) | 小 BLOB | L+2 字节，在此，L<2^16 |
| MEDIUMBLOB (M) | 中等大小的BLOB | L+3 字节，在此，L<2^24 |
| LONGBLOB (M) | 非常大的BLOB | L+4 字节，在此，L<2^32 |

## 8. 创建表

```s
CREATE TABLE tb_emp1(
    id INT(11),
    name VARCHAR(25),
    deptId INT(11),
    salary FLOAT
)
```

1.查看表

```s
SHOW TABLES;
```

使用 ```SHOW``` ```CREATE``` ````TABLE```` 语句查看表 ```tb_emp1``` 的详细信息

```s
SHOW CREATE TABLE tb_emp1;
```

以表格的形式展示表结构

```s
DESCRIBE tb_emp1;
```

## 9. 修改表

1.修改表名

```s
ALTER TABLE oldname RENAME TO newname;
```

2.修改表字符集

字符集修改为 ```gb2312```，校对规则修改为 ```gb2312_chinese_c```。

```s
ALTER TABLE newname CHARACTER SET gb2312  DEFAULT COLLATE gb2312_chinese_ci;
```

3.修改表中的字段

使用 ```ALTER``` ```TABLE``` 修改表 ```tb_emp1``` 的结构，将 ```col1``` 字段名称改为 ```col3```，同时将数据类型变为 ```CHAR(30)```

```s
ALTER TABLE tb_emp1 CHANGE col1 col3 CHAR(30);
```

使用 ```ALTER``` ```TABLE``` 修改表 ```tb_emp1``` 的结构，将 ```name``` 字段的数据类型由 ```VARCHAR(22)``` 修改成 ```VARCHAR(30)```

```s
ALTER TABLE tb_emp1 MODIFY name VARCHAR(30);
```

```CHANGE``` 也可以只修改数据类型，实现和 ```MODIFY``` 同样的效果，方法是将 ```SQL``` 语句中的```新字段名```和```旧字段名```设置为相同的名称，只改变```数据类型```。

4.删除字段

使用 ```ALTER``` ```TABLE``` 修改表 ```tb_emp1``` 的结构，删除 ```col2``` 字段。

```s
ALTER TABLE tb_emp1 DROP col2;
```

5.在末尾添加字段

使用 ```ALTER``` ```TABLE``` 语句添加一个 ```INT``` 类型的字段 ```age```。

```s
ALTER TABLE student ADD age INT(4);
```

6.在开头添加字段

使用 ```ALTER``` ```TABLE``` 语句在表的第一列添加 ```INT``` 类型的字段 ```stuId```。

```s
ALTER TABLE student ADD stuId INT(4) FIRST;
```

7.在中间位置添加字段

使用 ```ALTER``` ```TABLE``` 语句在 ````student```` 表中添加名为 ```stuno```，数据类型为 ```INT``` 的字段，```stuno``` 字段位于 ```name``` 字段的后面。

```s
ALTER TABLE student ADD stuno INT(11) AFTER name;
```

## 10. 删除表

删除数据表 ```tb_emp3```。

```s
DROP TABLE tb_emp3;
```

数据表之间经常存在外键关联的情况，这时如果直接删除父表，会破坏数据表的完整性，也会删除失败，可以先删除与它关联的子表，再删除父表；但是这样会同时删除两个表中的数据。还可以将关联表的外键约束取消，再删除父表；适用于需要保留子表的数据，只删除父表的情况。

## 11. 约束

1.PRIMARY KEY 主键约束

```s
CREATE TABLE tb_emp2
(
    id INT(11) PRIMARY KEY,
    name VARCHAR(25),
    deptId INT(11),
    salary FLOAT
);
# or
CREATE TABLE tb_emp2
(
    id INT(11),
    name VARCHAR(25),
    deptId INT(11),
    salary FLOAT,
    PRIMARY KEY(id)
);
# or
ALTER TABLE tb_emp2 ADD PRIMARY KEY(id);
# delete
ALTER TABLE tb_emp2 DROP PRIMARY KEY;
```

2.AUTO_INCREMENT 主键自增长

```s
CREATE TABLE tb_student(
    id INT(4) PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(25) NOT NULL
);
# 指定初始值
CREATE TABLE tb_student(
    id INT(4) PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(25) NOT NULL
)AUTO_INCREMENT=100;
```

## 11. 新增数据

```s
INSERT INTO tb_courses (course_id,course_name,course_grade,course_info) VALUES(1,'Network',3,'Computer Network');
# or
INSERT INTO tb_courses VLAUES(3,'Java',4,'Java EE');
```

## 12. 更新数据

```s
UPDATE tb_courses SET course_grade=4;
```

## 13. 删除数据

```s
DELETE FROM tb_courses;
```

删除表中的全部数据，自增主键会重置

```s
TRUNCATE TABLE tb_student_course;
```

## 14. 查询数据

```s
SELECT * FROM tb_student;
```
1.查询name字段

```s
SELECT name FROM tb_student;
```

2.去重查询name及age字段

```s
SELECT DISTINCT name,age FROM tb_student;
```

3.指定别名

```s
SELECT name AS student_name, age AS student_age FROM tb_student;
```

4.分页查询

从第 ```4``` 条开始的返回 ```5``` 条记录。

```s
SELECT * FROM tb_student LIMIT 3,5;
# or
SELECT * FROM tb_student LIMIT 5 OFFSET 3;
```

5.排序

按```height```字段进行排序

```s
SELECT * FROM tb_student ORDER BY height;
```

先按 ```height``` 排序，再按 ```name``` 排序

```s
SELECT name,height FROM tb_student ORDER BY height,name;
```

```DESC``` 关键字只对前面的列进行降序排列，```ASC```升序排序

先按 ```height``` 降序排序，再按 ```name``` 升序排序

```s
SELECT name,height FROM tb_student ORDER BY height DESC,name ASC;
```

6.条件查询

```s
# 查询身高为 170cm 的信息
SELECT name,height FROM tb_student WHERE height=170;
# 查询年龄小于 22 的信息
SELECT name,age FROM tb_student WHERE age<22;
# 查询 age 大于 21，并且 height 大于等于 175 的信息
SELECT name,age,height FROM tb_student WHERE age>21 AND height>=175;
# 查询 age 大于 21，或者 height 大于等于 175 的信息
SELECT name,age,height FROM tb_student WHERE age>21 OR height>=175;
# 查询 age 大于 21，并且 height 小于 175 的信息和 age 小于 21，并且 height 大于等于 175 的信息
SELECT name,age,height FROM tb_student WHERE age>21 XOR height>=175;
```

7.模糊查询

查找所有以字母```T```开头的姓名

```s
SELECT name FROM tb_students_info WHERE name LIKE 'T%';
```

查找所有以字母```y```结尾，且```y```前面只有 ```4``` 个字母的姓名

```s
SELECT name FROM tb_students_info WHERE name LIKE '____y';
```

查找所有以字母```t```开头的姓名，区分大小写和不区分大小写

```s
SELECT name FROM tb_students_info WHERE name LIKE 't%';
# or
SELECT name FROM tb_students_info WHERE name LIKE BINARY 't%';
```

8.范围查询

查询年龄在 ```20``` 到 ```23``` 之间的姓名和年龄

```s
SELECT name,age FROM tb_students_info WHERE age BETWEEN 20 AND 23;
```

查询年龄不在 ```20``` 到 ```23``` 之间的姓名和年龄

```s
SELECT name,age FROM tb_students_info WHERE age NOT BETWEEN 20 AND 23;
```

查询注册日期在 ```2015-10-01``` 和 ```2016-05-01``` 之间的信息

```s
SELECT name,login_date FROM tb_students_info WHERE login_date BETWEEN '2015-10-01' AND '2016-05-01';
```

9.空值查询

查询 ```tb_students_info``` 表中 ```login_date``` 字段是 ```NULL``` 的记录

```s
SELECT `name`,`login_date` FROM tb_students_info WHERE login_date IS NULL;
```

查询 tb_students_info 表中 ```login_date``` 字段不为空的记录

```s
SELECT `name`,login_date FROM tb_students_info WHERE login_date IS NOT NULL;
```

10.内连查询

在 ```tb_students_info``` 表和 ```tb_course``` 表之间，使用内连接查询姓名和相对应的课程名称

```s
SELECT tb_students_info.name,tb_course.course_name FROM tb_students_info INNER JOIN tb_course ON tb_students_info.course_id = tb_course.id;
```