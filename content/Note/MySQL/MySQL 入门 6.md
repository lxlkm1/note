#后端 #数据库 #MySQL 

# 1 知识回顾

## 1.1 变量

> [!note] 知识点标签：#数据库/变量分类
> MySQL 中的变量主要分为**系统变量**（使用 `@@` 表示）和**用户自定义变量**：
> - **系统变量**：数据库引擎内部定义的变量。进一步分为全局系统变量（`GLOBAL`）和会话系统变量（`SESSION`）。
> - **用户变量**：根据作用域和定义方式，可分为用户会话变量（通常带 `@` 前缀）和局部变量（在 `BEGIN...END` 块中使用 `DECLARE` 定义）。

### 1.1.1 常用SQL

#### 1.1.1.1 查询系统变量

```sql
-- 查询系统变量全集
SHOW GLOBAL VARIABLES;
SHOW SESSION VARIABLES;

-- 模糊匹配查询系统变量
SHOW GLOBAL VARIABLES LIKE '[匹配模式表达式]';
SHOW SESSION VARIABLES LIKE '[匹配模式表达式]';

-- 精确查询特定变量
SELECT @@global.[系统变量名称];
SELECT @@session.[系统变量名称];
```

#### 1.1.1.2 修改系统变量

```sql
-- 动态修改系统变量配置
SET @@global.[系统变量名称] = [目标参数值];
SET @@session.[系统变量名称] = [目标参数值];
```

#### 1.1.1.3 使用用户会话变量

```sql
-- 定义与赋值（推荐使用 := 以区分比较运算符）
SET @[用户会话变量名称] := [初始赋值];

-- 查询验证
SELECT @[用户会话变量名称];

-- 搭配查询语句动态赋值
SELECT [目标字段名称] INTO @[用户会话变量名称] FROM [数据表名称] WHERE [筛选条件表达式];
```

#### 1.1.1.4 使用局部变量

> [!danger] 语法差异警示
> 1. 位置限制：`DECLARE` 语句必须严格放置在 `BEGIN...END` 块的最前面，任何业务操作或控制流语句之前。

```sql
DELIMITER $

CREATE PROCEDURE [存储过程名称]()
BEGIN
    -- 必须最优先声明
    DECLARE [局部变量名称] [局部变量类型] DEFAULT [默认初始值];
    
    -- 变量赋值
    SET [局部变量名称] = [动态计算值];
    
    -- 其他业务逻辑...
END$

DELIMITER ;
```

## 1.2 流程控制语句

### 1.2.1 IF ELSE 选择语句

```sql
-- IF 控制流仅能在存储程序（如存储过程、函数）的上下文中使用
IF [布尔条件表达式1] THEN 
    [执行逻辑块1];
ELSEIF [布尔条件表达式2] THEN 
    [执行逻辑块2];
ELSE 
    [默认执行逻辑块];
END IF;
```

### 1.2.2 LOOP 循环语句

> [!warning] 死循环风险
> `LOOP` 自身不带有终止条件，必须在循环体内部配合 `IF` 语句和 `LEAVE` 关键字来显式跳出循环。

```sql
[循环标签名称]: LOOP
    IF [退出循环条件表达式] THEN
        LEAVE [循环标签名称];
    END IF;
    
    -- 需要被循环执行的操作...
END LOOP [循环标签名称];
```

### 1.2.3 CASE WHEN 分支语句

```sql
CASE
    WHEN [布尔条件表达式1] THEN [执行逻辑块1];
    WHEN [布尔条件表达式2] THEN [执行逻辑块2];
    WHEN [布尔条件表达式3] THEN [执行逻辑块3];
    ELSE [默认执行逻辑块];
END CASE;
```

## 1.3 游标

> [!note] 知识点标签：#数据库/游标机制
> 游标（Cursor）可以被视为一个指向关系型数据集的只读迭代器。它允许程序在 `SELECT` 返回的多行结果集中，每次单向提取一行数据赋给变量，以供精细化逐行处理。


```sql
DELIMITER $

CREATE PROCEDURE [存储过程名称]()
BEGIN
    -- 1. 声明用于接收数据的局部变量
    DECLARE [接收字段1的变量] [对应数据类型];
    DECLARE [接收字段2的变量] [对应数据类型];
    
    -- 2. 声明终止标志位
    DECLARE [游标到达末尾标志] INT DEFAULT 0;
    
    -- 3. 声明游标实体并绑定查询
    DECLARE [游标名称] CURSOR FOR SELECT [字段1名称], [字段2名称] FROM [数据表名称];
    
    -- 4. 声明异常处理器，当 FETCH 抓取不到数据时触发标志位变更
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET [游标到达末尾标志] = 1;
    
    -- 5. 开启游标
    OPEN [游标名称];
    
    -- 6. 循环抓取数据
    [游标循环标签]: LOOP
        -- 抓取单行数据并将指针下移
        FETCH [游标名称] INTO [接收字段1的变量], [接收字段2的变量];
        
        -- 验证是否触底
        IF [游标到达末尾标志] = 1 THEN
            LEAVE [游标循环标签];
        END IF;
        
        -- 数据清洗或流转逻辑...
    END LOOP [游标循环标签];
    
    -- 7. 强制关闭释放内存
    CLOSE [游标名称];
END$

DELIMITER ;
```

## 1.4 窗口函数



### 1.4.1 核心语法结构

```sql
[目标函数名称]([函数参数]) OVER (
    PARTITION BY [分组依据字段] 
    ORDER BY [排序依据字段] 
    [窗口滑动帧边界定义]
)
```


### 1.4.2 常用场景示例

```sql
-- 场景：计算每位员工在其部门内部的薪资排名，同时输出员工个人明细
SELECT 
    [员工姓名],
    [所属部门名称],
    [薪资数额],
    DENSE_RANK() OVER (
        PARTITION BY [所属部门名称] 
        ORDER BY [薪资数额] DESC
    ) AS [部门内薪资名次]
FROM 
    [员工薪资明细表];
```



