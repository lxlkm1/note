
#MySQL #数据库 #后端

# 1 知识点回顾

## 1.1 创建和管理数据库、数据表

### 1.1.1 创建数据库

```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS [database_name];

-- 创建数据库并指定字符集（推荐）
CREATE DATABASE IF NOT EXISTS [database_name] CHARACTER SET = 'utf8mb4';
```
> [!tip] 最佳实践
> 推荐使用 `utf8mb4` 而不是 `utf8`，因为 `utf8mb4` 是 MySQL 中真正的 UTF-8 实现，能够完整支持 Emoji 表情和复杂的 Unicode 字符。

### 1.1.2 删除数据库

```sql
DROP DATABASE IF EXISTS [database_name];
```
> [!danger] 高危操作警告
> 删除数据库是不可逆操作，执行前务必确认数据已备份，且当前连接的环境不是生产环境（Production）。

### 1.1.3 更新数据库

```sql
ALTER DATABASE [database_name] CHARACTER SET 'utf8mb4';
```

### 1.1.4 展示数据库表格

```sql
SHOW TABLES FROM [database_name];
```

### 1.1.5 数据库重命名

> [!warning] 版本差异与坑点
> MySQL 目前**没有**重新命名数据库的安全命令。
> **安全做法**：只能创建新的数据库，然后把老数据库数据迁移过去，验证无误后删除旧数据库。

### 1.1.6 数据库常识补充

> [!note] 知识点标签：#分类/基础常识
> 1. **大小写敏感性**：数据库名、表数据、表字段在 Windows 上完全不区分大小写。
> 2. **时间数据类型**：`TIME` 用于存储时间，格式为 `HH:MM:SS`；`DATETIME` 用于存储日期时间，格式为 `YYYY-MM-DD HH:MM:SS`。
> 3. **命名规范**：数据库处理具体记录数据以外的部分命名（如表字段、数据库名字等）强烈推荐全部使用数字、小写字母加下划线（`_`）。

## 1.2 数据表操作

### 1.2.1 增加字段

```sql
ALTER TABLE [table_name]
ADD COLUMN [col_name] [data_type];
```

### 1.2.2 删除字段

```sql
ALTER TABLE [table_name] 
DROP COLUMN [col_1], 
DROP COLUMN [col_2];
```

### 1.2.3 修改字段数据类型或位置

```sql
-- 仅修改数据类型
ALTER TABLE [table_name]
MODIFY COLUMN [col_name] [data_type];

-- 修改数据类型并调整列位置（底层执行时是在重新定义列，因此必须指定类型）
ALTER TABLE [table_name]
MODIFY COLUMN [col_name] [data_type] AFTER [col2_name];
```

### 1.2.4 修改字段名字

```sql
-- CHANGE 关键字既可以改名也可以改类型，必须连带数据类型一起写上
ALTER TABLE [table_name]
CHANGE COLUMN [old_col_name] [new_col_name] [data_type]; 
```

### 1.2.5 删除表格

```sql
DROP TABLE IF EXISTS [table_name];
```

### 1.2.6 重命名表格

```sql
RENAME TABLE [old_table_name] TO [new_table_name];
```

### 1.2.7 克隆表格

```sql
-- 创建一个结构像 table2 的新表（推荐方式，包含主键、索引等约束）
CREATE TABLE [table_name] LIKE [table2];

-- 等效于子查询创建（坑点：不包含索引、主键等约束）
CREATE TABLE [table_name] AS
SELECT * FROM [table2] WHERE 1 = 2;
```

## 1.3 数据表记录操作

### 1.3.1 增加记录

```sql
INSERT INTO [table_name] ([col_1], [col_2]) VALUES
([data1]),
([data2]);

INSERT INTO [table_name] ([col_1], [col_2])
SELECT [col_1], [col_2] FROM [other_table];
```

### 1.3.2 删除记录

```sql
DELETE FROM [table_name] WHERE [condition];
```

### 1.3.3 修改记录

```sql
UPDATE [table_name] SET [col_name] = [new_value] WHERE [condition];
```

# 2 Q&A 疑难解答

> [!info] 为什么删除字段要加一个 COLUMN，有没有更好的做法？
> **解答**：其实 `COLUMN` 可以被省略。网上大多数开发者在 `ADD` 时习惯省略，但在 `DROP` 时为了明确语义、防止与删除表混淆，通常不会省略。
> **最佳实践**：为了代码可读性和统一性，建议无论是增、删、改，都显式加上 `COLUMN`。

> [!info] 为什么修改字段名字使用 CHANGE 还要写字段类型，有没有更好的写法？
> **解答**：MySQL 8.0 引入了极简的重命名语法，不需要再重写数据类型：
> ```sql
> ALTER TABLE [table_name] 
> RENAME COLUMN [old_name] TO [new_name];
> ```

> [!info] 为什么修改字段的位置需要写字段类型？
> **解答**：因为 `MODIFY COLUMN` 在底层执行时实际上是在重新定义该列。解析器需要完整的数据类型约束才能确保表结构的安全更新。

> [!tip] 最佳实践：克隆空表的通俗写法
> 相比于 `CREATE TABLE ... AS SELECT ... WHERE 1 = 2`（该方法会丢失主键和索引属性），更标准且通俗的空表结构克隆方式是使用 `LIKE`：
> ```sql
> CREATE TABLE employees_blank LIKE employees;
> ```

> [!info] 为什么这一章没有给数据库重命名的教程？
> **解答**：因为 MySQL 缺乏安全直接重命名数据库的指令。标准且安全的工程处理方案就是：建立新数据库 -> 迁移旧库的表和数据 -> 验证无误 -> 删除旧数据库。

# 3 最佳实践与证据

> [!tip] 最佳实践：始终显式指定字符集以规避乱码
> 在创建数据库时，永远不要依赖系统或环境的默认字符集。强烈建议显式声明使用 `utf8mb4`，以确保对 Emoji 和四字节冷门 Unicode 字符的完美兼容。
> *(证据：[MySQL 8.0 Reference Manual - 10.1.1 Character Set Repertoire](https://dev.mysql.com/doc/refman/8.0/en/charset-repertoire.html))*
