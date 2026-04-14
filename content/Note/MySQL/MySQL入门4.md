# 1 知识点回顾


## 1.1 创建和管理数据库、数据表

### 1.1.1 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS [database_name];

CREATE DATABASE IF NOT EXISTS [database_name] CHARACTER SET = 'utf8';
```

### 1.1.2 删除数据库

```sql
DROP DATABASE IF EXISTS [database_name];
```

### 1.1.3 更新数据库

```sql
ALTER DATABASE [database_name] CHARACTER SET 'utf8';
```

### 1.1.4 展示数据库表格

```sql
SHOW TABLES FROM [database_name];
```

### 1.1.5 数据库重命名

数据库目前没有重新命名的方法，只能创建新的，然后把老数据库数据迁移

### 1.1.5 数据库常识补充

#### 1.1.5.1 数据库特点

1. 数据库 表数据， 表字段 在windows 上完全不区别大小写 
2. TIME：用于存储时间，格式为HH:MM:SS，DATETIME：用于存储日期时间，格式为YYYY-MM-DD HH:MM:SS
3. 数据库处理具体记录数据以外的部分命名，如表字段，数据库名字等等全部使用数字，小写字母加_

## 1.2 数据表操作

### 1.2.1 增加字段

```sql
ALTER [table_name]
ADD [col_name] [data_type];
```

### 1.2.2 删除字段

```sql
ALTER TABLE employees 
DROP COLUMN [col_1], 
DROP COLUMN [col_2];
```

### 1.2.3 修改字段数据类型 或者 位置

```sql
ALTER [table_name]
MODIFY [col_name] [data_type];

ALTER [table_name]

-- 这里为什么需要指定类型
MODIFY [col_name] [data_type] AFTER [col2_name];
```

### 1.2.4 修改字段名字

```sql
ALTER [table_name]
-- 这里不能修改数据类型
CHANGE [col_name] [new_name] [data_type]; 
```

### 1.2.5 删除表格

```sql
DROP TABLE [table_name];
```

### 1.2.6 重命名表格

```sql
RENAME [table_name] TO [new_name]
```

### 1.2.7 克隆表格

```sql
-- 创建一个结构像table2的
CREATE TABLE [table_name] LIKE [table2]

--等效于
CREATE TABLE [table_name] AS
SELECT ... FROM ... WHERE 1=2

```


## 1.3 数据表记录操作

### 1.3.1 增加记录

```sql
INSERT INTO [table_name]([col_1, col_2]) VALUES
([data1])
([data2])

INSERT INTO [table_name]([col_1, col_2])
SELECT * FROM ....
```

### 1.3.2 删除记录

```sql
DELETE FROM [table_name] WHERE ....
```

### 1.3.3 修改记录

```sql
update [table_name] SET [col_name] = ... WHERE ...
```
# 2 Q&A

1数据库增加字段使用
```sql
ALTER TABLE employees
ADD favoriate_activity VARCHAR(100);
```
而删除字段使用
```sql
ALTER TABLE employees
DROP COLUMN note;
```
为什么删除字段要加一个COLUMN，感觉有点麻烦，有没有更好的做法

> **解答**：其实COLUMN 可以被省略，网上大多数开发者ADD 习惯省略COLUMN,但是DROP就不省略，我也不知道为什么

2为什么给数据库修改字段名字不直接使用MODIFY 关键词，而使用CHANGE 关键词，而且还要写字段类型，有没有更好的写法

> **解答**：

```sql
-- MySQL 8.0 极简重命名语法 不需要写数据类型
ALTER TABLE [table_name] 
RENAME COLUMN [old_name] TO [new_name];
```


3为什么修改字段的位置需要写字段类型，有没有更好的写法

> **解答**：不知道


4有没有参考一个表复制建立空表更通俗的写法，相比于
```sql
CREATE TABLE employees_blank
AS
SELECT *
FROM employees
#where department_id > 10000;
WHERE 1 = 2; 
```

> **解答**：
```sql
CREATE TABLE employees_blank LIKE employees
```

5为什么这一章没有给数据库重命名的教程

> **解答**：就是建立新数据库，然后把数据移进去