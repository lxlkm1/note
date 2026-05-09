
#MySQL #数据库 #后端

# 1 知识回顾

## 1.1 约束 (Constraints)

### 1.1.1 概述

> [!note] 知识点标签：#分类/数据库约束
> **约束（Constraints）**是用于限制表中字段录入规则的机制，在定义主键、唯一键、外键和检查约束时，推荐统一采用**表级约束规范**进行集中声明。

### 1.1.2 种类

> [!note] 知识点标签：#分类/约束类型
> 核心约束主要分为以下四类：
> * **主键 (PRIMARY KEY)**: 唯一标识表中的记录，默认强制包含 `UNIQUE` 和 `NOT NULL` 属性。
> * **唯一 (UNIQUE)**: 保证字段的值在全表中唯一，但允许录入多个 `NULL` 值。
> * **外键 (FOREIGN KEY)**: 保证该字段的值必须在参考表的主键或唯一键中存在，用于维持表间级联一致性。
> * **检查 (CHECK)**: 限制字段的值必须满足指定的条件表达式（例如：`age >= 18`）。*(注：MySQL 8.0.16 起才真正支持并强制执行，老版本仅解析但实际上不生效)*。

> [!danger] 核心差异与坑点（列固有属性）
> 以下三种在 MySQL 的底层实现中，并不属于传统意义上的“约束（Constraint）”，而是属于“列属性（Column Attributes）”或“列标志（Column Flags）”，因此它们**不存在约束名**。在语法上，它们必须严格绑定在列定义上（即只能写在**列级**）：
> * **自增 (AUTO_INCREMENT)**: 自动为新行生成唯一标识数字，通常配合主键使用。
> * **非空 (NOT NULL)**: 限制该字段绝对不能录入 `NULL` 值。
> * **默认值 (DEFAULT)**: 为没有显式赋值的字段提供一个预设的兜底值。

### 1.1.3 约束常用 SQL 语句

#### 1.1.3.1 查看约束信息

```sql
SELECT * FROM information_schema.table_constraints WHERE table_name = '[你的表名]';
```

#### 1.1.3.2 增加约束（建表阶段）

> [!tip] 最佳实践：分离定义
> 建表时，建议先定义所有列及其固有属性，最后在表级统一声明所有核心约束。这样可以使表结构一目了然，极大地便于后续的集中代码审查。

```sql
CREATE TABLE [表名] (
    -- 1. 列定义与固有属性 (必须在列级声明)
    [列名_1] INT NOT NULL AUTO_INCREMENT,  
    [列名_2] VARCHAR(50) DEFAULT '[默认文本]',     
    [列名_3] INT,
    [列名_4] INT,
    
    -- 2. 表级约束声明区 (格式：CONSTRAINT [自定义约束名] [约束类型])
    CONSTRAINT [主键约束名] PRIMARY KEY ([列名_1]),                                  
    CONSTRAINT [唯一约束名] UNIQUE ([列名_2]),                                       
    CONSTRAINT [外键约束名] FOREIGN KEY ([列名_3]) REFERENCES [参考表名]([参考列名]), 
    CONSTRAINT [检查约束名] CHECK ([列名_4] > 0)                           
);

-- 复合约束演示（例如使用两个字段共同组成一个复合主键）
CREATE TABLE [表名] (
    [列名_1] INT NOT NULL,
    [列名_2] INT NOT NULL,
    
    CONSTRAINT [复合主键名] PRIMARY KEY ([列名_1], [列名_2])
);
```

> [!warning] 命名规范隐患
> 主键约束在一张表中物理上只能存在一个。虽然你可以通过 `CONSTRAINT` 给主键起别名，但在 MySQL 的底层系统表中，主键的名称会被引擎强制重写并固定为 `PRIMARY`。保留自定义命名主要是为了与其他主流关系型数据库（如 Oracle/PostgreSQL）保持跨平台的 DDL 兼容习惯。

#### 1.1.3.3 增加约束（建表后使用 ALTER）

```sql
-- 统一使用表级约束语法追加约束
ALTER TABLE [表名]
ADD CONSTRAINT [自定义唯一约束名] UNIQUE ([列名_1], [列名_2]);

ALTER TABLE [表名]
ADD CONSTRAINT [自定义主键名] PRIMARY KEY ([列名_1]);

-- 增加检查约束
ALTER TABLE [表名]
ADD CONSTRAINT [自定义检查约束名] CHECK ([列名_1] >= 18 AND [列名_1] <= 65);
```

#### 1.1.3.4 删除约束

```sql
-- 删除主键 (因为表里固定只有一个主键，不需要指定名字)
ALTER TABLE [表名] 
DROP PRIMARY KEY;

-- 删除唯一/外键/检查约束 (MySQL 8.0.16+ 引入了标准规范，推荐统一使用 DROP CONSTRAINT)
ALTER TABLE [表名] 
DROP CONSTRAINT [目标约束名];

-- 删除唯一约束 (老版本 MySQL 兼容写法，本质是摧毁支撑该唯一约束的底层 B+ 树索引)
ALTER TABLE [表名] 
DROP INDEX [目标约束名];

-- 删除外键约束 (老版本兼容写法)
ALTER TABLE [表名] 
DROP FOREIGN KEY [目标约束名];

-- 删除检查约束 (MySQL 8.0.16+ 专用语法)
ALTER TABLE [表名] 
DROP CHECK [目标约束名];

-- 针对只能定义在列级的属性 (如 NOT NULL / DEFAULT)，使用 MODIFY 重新覆盖定义即可（不写属性即视为擦除）
ALTER TABLE [表名]
MODIFY [列名] [数据类型];
```

#### 1.1.3.5 修改约束

> [!info] 核心操作思路
> 修改约束的最佳方法：对于表级约束，直接**先执行删除（DROP），再重新建立（ADD）**；对于列级约束属性，直接使用 `MODIFY` 覆盖原定义即可。

---

## 1.2 视图 (Views)

### 1.2.1 概念

> [!note] 知识点标签：#分类/数据库对象/视图
> **视图（View）**本质上是一条被封装的 SQL 语句，它是基于其他物理表的数据动态拼接成的一张**虚拟表**。
> * **优势**：视图不存储真实的底层数据，可以节约磁盘空间；可以用于隐藏敏感列以提高安全性。
> * **特性**：视图和物理表在查询语法上没有区别，只是把定义关键字从 `TABLE` 换成了 `VIEW`。

### 1.2.2 常用 SQL

#### 1.2.2.1 创建视图

```sql
-- 基于单表创建视图
CREATE VIEW [视图名] AS SELECT * FROM [物理表名];

-- 基于已有视图嵌套创建新视图
CREATE VIEW [新视图名] AS SELECT * FROM [旧视图名];

-- 指定视图字段名并结合复杂查询
CREATE VIEW [视图名]([自定义列名_1], [自定义列名_2]) AS [任何包含分组/过滤等逻辑的 SELECT 语句];
```

#### 1.2.2.2 查询视图

```sql
-- 查询视图与查询物理 TABLE 语法完全一致
SELECT * FROM [视图名] WHERE [条件表达式];
```

#### 1.2.2.3 查看视图结构

```sql
DESC [视图名];
```

#### 1.2.2.4 更新视图数据

> [!warning] 更新限制
> 尝试修改视图数据（INSERT/UPDATE/DELETE），其本质是在修改底层物理表的数据。但并非所有视图都是“可更新的（Updatable）”。如果视图的定义中包含了聚合函数（`COUNT`, `SUM` 等）、`DISTINCT`、`GROUP BY`、`HAVING`、`UNION` 等会破坏数据行“一一对应关系”的操作，则该视图将被引擎锁定为**只读视图**。

#### 1.2.2.5 修改视图定义

```sql
-- 覆盖修改：如果视图存在则替换，不存在则创建
CREATE OR REPLACE VIEW [视图名] AS [新的 SELECT 查询语句];
```

#### 1.2.2.6 删除视图

```sql
DROP VIEW IF EXISTS [视图名];
```

---

## 1.3 存储过程 (Stored Procedures)

### 1.3.1 概念

> [!note] 知识点标签：#分类/数据库对象/可编程组件
> **存储过程**就是把一组为了完成特定功能的复杂 SQL 语句集提前编译并保存在数据库引擎中，业务端通过 `CALL` 命令传入参数即可调用执行。
> * **语法区别**：存储函数必须作为表达式嵌套在普通 SQL 查询语句中使用；而存储过程可以独立执行。
> * **语义边界**：建议将函数（Function）严格限制为“无副作用的查询或计算器”，不应在其中改变数据状态；而将存储过程（Procedure）视为“操作命令（Command）”，用于执行增删改等具有业务副作用的复杂事务。

### 1.3.2 常用 SQL

#### 1.3.2.1 创建存储过程

```sql
-- 推荐前置清理，防止命名冲突报错
DROP PROCEDURE IF EXISTS [存储过程名];

-- 临时将 SQL 语句块的结束标识符从分号改为其他符号（如 $），防止过程体内部的分号导致提前截断编译
DELIMITER $

CREATE PROCEDURE [存储过程名]([参数传递模式] [参数名] [数据类型], ...)
BEGIN
    [内部待执行的各种 SQL 语句集];
END $

-- 编译完成后，务必将结束符重置回标准的分号
DELIMITER ;
```

> [!danger] 命名冲突隐患
> **参数名绝对不要和表中的实际物理列名（字段名）重复。** 否则在过程体内执行 `WHERE [列名] = [参数名]` 时，MySQL 引擎会将其解析为 `WHERE [列名] = [列名]` 从而永远返回 TRUE，引发严重的越权更新或全表数据泄露。

**安全示例 (Example):**

```sql
DROP PROCEDURE IF EXISTS p_get_beauty_limit;

DELIMITER $
-- OUT 表示输出参数，IN 表示输入参数
CREATE PROCEDURE p_get_beauty_limit(OUT p_msg VARCHAR(30), IN p_offset INT, IN p_limit INT)
BEGIN
    SELECT CONCAT(last_name, age) INTO p_msg 
    FROM beauty 
    LIMIT p_limit OFFSET p_offset;
END $

DELIMITER ;
```

#### 1.3.2.2 调用存储过程

```sql
CALL [存储过程名]([传入的具体参数]);
```

**示例 (Example):**

```sql
-- 推荐使用 := 操作符进行会话变量赋值，以区分等于比较符
SET @v_offset := 0;
SET @v_limit := 5;

-- 执行调用，并将返回值写入用户变量 @v_msg 中
CALL p_get_beauty_limit(@v_msg, @v_offset, @v_limit);

-- 查看输出结果
SELECT @v_msg;
```

---

## 1.4 存储函数 (Stored Functions)

### 1.4.1 常用 SQL

#### 1.4.1.1 创建存储函数

```sql
DROP FUNCTION IF EXISTS [存储函数名];

DELIMITER $

CREATE FUNCTION [存储函数名]([参数名] [数据类型], ...) 
RETURNS [返回的数据类型]
BEGIN
    RETURN (
        [核心逻辑查询语句，必须返回单值]
    );
END $

DELIMITER ;
```

> [!warning] 语法易错点
> 1. 函数声明头部的 `RETURNS` 必须带有末尾的 `s`。
> 2. 函数体内部具体返回值时使用的关键字是 `RETURN`（没有 `s`）。
> 3. 函数参数列表不需要明确指定 `IN/OUT` 等模式，因为它在语义上强制约束为仅接受输入。

**示例 (Example):**

```sql
DROP FUNCTION IF EXISTS f_get_count_by_dept;

DELIMITER $
CREATE FUNCTION f_get_count_by_dept(p_dept_id INT)
RETURNS INT
BEGIN
    -- 函数体必须有确定的返回值
    RETURN (SELECT COUNT(*) FROM employees WHERE department_id = p_dept_id);
END $

DELIMITER ;
```

#### 1.4.1.2 调用存储函数

> [!info] 调用方式
> 存储函数的调用方式和 MySQL 的内置自带函数（如 `LENGTH()`, `NOW()`）完全一样，直接将其作为表达式的一部分嵌入 `SELECT` 或 `WHERE` 子句即可。

**示例 (Example):**

```sql
SET @v_target_dept_id := 50;
SELECT f_get_count_by_dept(@v_target_dept_id) AS [部门总人数];
```

