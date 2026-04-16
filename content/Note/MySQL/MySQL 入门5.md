### 1 知识回顾

#### 1.1 约束

**1.1.1 概述** 
约束就是限制字段的规则。为了保证可读性与规范性，本笔记对主键、唯一键、外键和检查约束**统一采用表级约束规范**。

**1.1.2 种类**

- **主键 (PRIMARY KEY):** 唯一标识表中的记录，默认包含 UNIQUE 和 NOT NULL 属性。
    
- **唯一 (UNIQUE):** 保证字段的值在全表中唯一。
    
- **外键 (FOREIGN KEY):** 保证该字段的值必须在参考表的主键或唯一键中存在。
    
- **检查 (CHECK):** 限制字段的值必须满足指定的条件表达式（例如：`age >= 18`）。_(注：MySQL 8.0.16 起才真正支持并强制执行，老版本仅解析不生效)_。


_(注：以下三种为列的固有属性，语法上必须写在列级)_

- **自增 (AUTO_INCREMENT):** 自动为新行生成唯一标识，通常配合主键使用。
    
- **非空 (NOT NULL):** 限制该字段不能录入 NULL 值。
    
- **默认值 (DEFAULT):** 为没有显式赋值的字段提供一个预设值。


_> 注意点：AUTO_INCREMENT、NOT NULL 和 DEFAULT 在 MySQL 的底层实现中，并不属于传统意义上的“约束（Constraint）”，而是属于“列属性（Column Attributes）”或“列标志（Column Flags）”，所以不存在约束名。_

**1.1.3 约束常用 SQL 语句**

**1.1.3.1 查看约束信息**

```SQL
SELECT * FROM information_schema.table_constraints WHERE table_name = '你的表名';
```

**1.1.3.2 增加约束（建表阶段）**

```SQL
-- 推荐规范：先定义所有列及其固有属性，最后在表级统一声明约束
CREATE TABLE [table_name] (
    [col_1] INT NOT NULL AUTO_INCREMENT,  -- NOT NULL 和自增只能写在列级
    [col_2] VARCHAR(50) DEFAULT '未知',     -- DEFAULT 只能写在列级
    [col_3] INT,
    [col_4] INT,
    
    -- 【表级约束规范区域】格式：CONSTRAINT [自定义约束名] 约束类型 (字段名或表达式)
    CONSTRAINT pk_col1 PRIMARY KEY (col_1),                          -- 声明主键
    CONSTRAINT uk_col2 UNIQUE (col_2),                               -- 声明唯一约束
    CONSTRAINT fk_col3 FOREIGN KEY (col_3) REFERENCES [other_table]([ref_col]), -- 声明外键
    CONSTRAINT chk_col4_positive CHECK (col_4 > 0)                   -- 声明检查约束（要求 col_4 必须大于 0）
);

-- 复合约束演示（例如使用两个字段共同组成一个主键）
CREATE TABLE [table_name] (
    [col_1] INT NOT NULL,
    [col_2] INT NOT NULL,
    
    CONSTRAINT PRIMARY KEY (col_1, col_2)
);
```

_> 注意点：主键约束在一张表中只能存在一个。虽然通过 CONSTRAINT 给主键起了别名（如 pk_col1），但在 MySQL 底层，主键的名字会被引擎强制固定为 PRIMARY。保留自定义命名主要是为了与其他关系型数据库保持兼容习惯。_

**1.1.3.3 增加约束（建表后使用 ALTER）**

```SQL
-- 统一使用表级约束语法 (ADD CONSTRAINT)
ALTER TABLE [table_name]
ADD CONSTRAINT [自定义约束名] UNIQUE (col_1, col_2);

ALTER TABLE [table_name]
ADD CONSTRAINT [自定义主键名] PRIMARY KEY (col_1);

-- 增加检查约束
ALTER TABLE [table_name]
ADD CONSTRAINT [自定义检查名] CHECK (col_1 >= 18 AND col_1 <= 65);
```

**1.1.3.4 删除约束**

```SQL
-- 删除主键 (因为表里只有一个主键，不需要指定名字)
ALTER TABLE [table_name] 
DROP PRIMARY KEY;

-- 删除唯一约束/检查约束 (MySQL 8.0.16+ 引入了标准规范，推荐统一使用 DROP CONSTRAINT)
ALTER TABLE [table_name] 
DROP CONSTRAINT [constraint_name], -- 可用于删唯一、外键、检查
DROP CONSTRAINT [constraint_name];

-- 删除唯一约束 (老版本 MySQL 兼容写法，本质是摧毁支撑该约束的底层 B+ 树索引)
ALTER TABLE [table_name] 
DROP INDEX [constraint_name];

-- 删除外键约束 (老版本/兼容写法)
ALTER TABLE [table_name] 
DROP FOREIGN KEY [constraint_name];

-- 删除检查约束 (MySQL 8.0.16+ 专用语法)
ALTER TABLE [table_name] 
DROP CHECK [constraint_name];

-- 针对只能列级的约束(属性)，使用 MODIFY 覆盖，data_type 后面不写该属性即可擦除
ALTER TABLE [my_table]
MODIFY [col_id] [data_type];
```

**1.1.3.4 修改约束**

_>修改约束的最佳方法，对于表级约束，就是先删除约束再重新建立，而列级约束直接使用modify覆盖即可



#### 1.2 视图

**1.2.1 概念**

_> 视图就是把其他表的数据拼接成一张非真实存在的表，因为视图不是真正的表，没有真实数据，当在数据库表非常大的时候，使用视图去展示数据的时候比较建立一张新的表可以节约许多磁盘空间，视图和表在语法上最大区别就是把table 换成 view


**1.2.2 常用SQL**

**1.2.2.1 创建视图**

```SQL

CREATE VIEW [view_name] AS SELECT * FROM [table_name];

CREATE VIEW [view_name] AS SELECT * FROM [VIEW_name];

CREATE VIEW [view_name]([col_name] ...) AS [其他查询语句，分组查询什么的都可以];

```

**1.2.2.2 查询视图**

_>  和查询table 没有区别


**1.2.2.3 查看视图结构**

```SQL
DESC [view_name];
```

**1.2.2.4 更新视图数据**

_> 修改视图数据方式和修改表没有区别，在一些情况下视图不能被修改，比如创建视图使用了聚合函数的情况下，还有一些其他情况

**1.2.2.5 修改视图**
```SQL
CREATE OR REPLACE [view_name] AS [查询语句];
```

**1.2.2.6 删除视图**
```SQL
DROP VIEW IF EXISTS [view_name];
```


#### 1.3 存储过程

**1.3.1 概念**

_> 存储过程就是把提前写好的常用sql语句提前放置在指定数据库里面，然后使用call 去调用，类似于存储函数，在语法上区别是存储函数必须被放入查询语句，而存储过程可以单独被使用，在使用场景上区别是应当将 Function 严格限制为“查询（Query）”的一部分，它们只负责计算并返回结果而不改变系统状态而将 Procedure 视为“命令（Command）”，专门用于执行改变系统状态的操作。这种分离可以最大程度降低数据库死锁的风险

**1.3.2 常用**

**1.3.2.1 创建存储过程**

```SQL
-- 推荐加上
DROP PROCEDURE IF EXISTS [procedure_name];
-- 使用 $ 代替sql语句结束符号
DELIMITER $
CREATE PROCEDURE [procedure_name]([参数类型 IN,OUT,INOUT等等][参数名称][数据类型]...)
BEGIN
[所有普通的sql语句];
END$
-- 改回;
DELIMITER ;

--example
DROP PROCEDURE IF EXISTS beauty_limit;
DELIMITER $
CREATE PROCEDURE beauty_limit(OUT msg VARCHAR(30), IN bg INT, IN lim INT)
BEGIN
SELECT CONCATE(last_name, age) INTO  FROM beauty LIMIT lim OFFSET bg;
END$

DELIMITER ;
```
_> 参数名字不要和字段有重复

**1.3.2.1 调用存储过程**

```SQL
CALL [procedure_name] ([参数])

--example
-- 推荐使用SET := 表示赋值
SET @bg := 0;
SET @lim := 5;
CALL beauty_limit(@msg, @bg, @lim);
SELECT @msg;
```

#### 1.4 存储函数


**1.4.1 常用**

**1.4.1.1 创建存储函数**

```SQL
-- 推荐加上
DROP FUNCTION IF EXISTS [function_name];
-- 使用 $ 代替sql语句结束符号
DELIMITER $
CREATE PROCEDURE [function_name]([参数名称][数据类型]...) RETURNS [数据类型]
BEGIN
RETURN(
[所有普通的sql语句];
)
END $
-- 改回;
DELIMITER ;

--example
DROP FUNCTION IF EXISTS count_by_id;
DELIMITER $
CREATE FUNCTION count_by_id(dept_id INT)
RETURNS INT
BEGIN
	RETURN (SELECT COUNT(*) FROM employees WHERE department_id = dept_id);
END $
DELIMITER ;
```
_> 函数外面的return 要加上s, 返回数据类型，这里不需要指定参数类型IN OUT等等

**1.3.2.1 调用存储函数**

```SQL
--和sql 普通函数一样

--example
SET @dept_id := 50;
SELECT count_by_id(@dept_id);
```