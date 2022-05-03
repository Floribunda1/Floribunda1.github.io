---
title: MySQL CheatSheet
tags:
  - mysql
categories:
  - cheatsheet
---

### 登录

```bash
$ mysql --host=localhost --user=xx --password
```

### 数据库

```sql
SHOW DATABASES; # 显示所有数据库

CREATE DATABASE IF NOT EXISTS database_name; # 创建数据库

DROP DATABASE IF EXISTS database_name; # 删除数据库

USE DATABASE database_name; # 使用数据库
```

<!-- more -->

### 表

```sql
# 展示所有的表
SHOW TABLES;

# 删除表
DROP TABLE table_1, tabel_2;

# 创建表
CREATE TABLE IF NOT EXISTS table_name (
	field_name VARCHAR(100),
  field_name2 INT,
  field_name3 ENUM('a', 'b', 'c')
) COMMENT '这是表的注释';

# 查看表结构
DESCRIBE table_name;
DESC table_name;
EXPLAIN table_name;
SHOW COLUMNS FROM table_name;
SHOW FIELDS FROM table_name;
# 看表的 ascii 图
SHOW CREATE TABLE table_name;

# 从某个数据库看某张表
SHOW TABLES FROM database_name;
# or
SHOW TABLE database_name.table_name;
```

修改表名称

```sql
# 修改表名称
ALTER TABLE old_name RENAME TO new_name;
# or
RENAME TABLE xxx TO xx, yyy TO yy;
```

增加列

```sql
# 向表中添加一列
ALTER TABLE table_name ADD COLUMN column_name CHAR(4);
# 添加到第一列
ALTER TABLE table_name ADD COLUMN column_name CHAR(4) FIRST;
# 添加到某一列后面
ALTER TABLE table_name ADD COLUMN column_name CHAR(4) AFTER some_column;
```

删除列

```sql
ALTER TABLE table_name DROP COLUMN column_name;
```

修改列

```sql
# 修改列
ALTER TABLE table_name MODIFY column_name VARCHAR(20);
# 将一个列改为另一个列
ALTER TABLE table_name CHANGE old_name new_name VARCHAR(20);
# 修改列的位置，可以将 FIRST 替换为 AFTER xx
ALTER TABLE table_name MODIFY column_name VARCHAR(20) FIRST;
```

### 列的属性

```sql
# 格式为 列名 + 数据格式 + 属性

# 默认值
column1 VARCHAR(100) DEFAULT 'aa'  # 不设置默认值就相当于 DEFAULT NULL
# 非空
NOT NULL
```

主键

```sql
column1 INT PRIMARY KEY
```

UNIQUE

```sql
# 不允许重复
column1 VARCHAR(100) UNIQUE

# 约束的另一种格式 UNIQUE KEY [约束名] (column1, column2....)
CREATE TABLE student_info (
    number INT PRIMARY KEY,
    name VARCHAR(5),
    sex ENUM('男', '女'),
    id_number CHAR(18),
    department VARCHAR(30),
    major VARCHAR(30),
    enrollment_time DATE,
    UNIQUE KEY uk_id_number (id_number)
);
```

外键
