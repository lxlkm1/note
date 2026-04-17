
#MySQL #数据库 #后端

# 1 规范

> [!tip] 最佳实践
> SQL 语句统一规范：SQL 关键字建议全部大写，表名、列名等标识符全部使用小写字母加下划线（Snake Case）命名，以提高代码可读性和跨平台兼容性。

# 2 查询进阶

## 2.1 别名

```sql
-- AS 关键字用于为列或表指定别名，提升可读性
SELECT [列名] FROM [表名] AS [表别名] WHERE [条件表达式];
```

## 2.2 指定查询顺序

```sql
-- ORDER BY 用于指定结果集的排序方式
-- DESC 表示降序（从大到小），ASC 表示升序（从小到大，默认值）
SELECT [列名] FROM [表名] AS [表别名] WHERE [条件表达式] 
ORDER BY [排序字段_1] DESC, [排序字段_2] ASC;
```

## 2.3 分页查询

> [!warning] 语法纠错
> 原文中的 `LIMIT [start_index] [page_count]` 缺少逗号，标准语法应为 `LIMIT [起始索引], [返回行数]`。MySQL 中第一行数据的索引为 0。

```sql
-- 方式一：使用逗号分隔，第一个参数是起始索引，第二个参数是返回行数
SELECT [列名] FROM [表名] AS [表别名] WHERE [条件表达式] 
LIMIT [起始索引], [返回行数];

-- 方式二：使用 OFFSET 关键字，语义更加清晰（推荐写法）
SELECT [列名] FROM [表名] AS [表别名] WHERE [条件表达式] 
LIMIT [返回行数] OFFSET [起始索引];
```

## 2.4 查询过滤

```sql
-- DISTINCT 关键字会过滤掉结果集中完全相同的数据行，实现去重
SELECT DISTINCT [列名] FROM [表名] WHERE [条件表达式];
```

## 2.5 常数查询

```sql
-- 当查询不需要基于任何实际的表时，可以使用虚拟表 DUAL 占位
SELECT DISTINCT [常量值] AS [别名] FROM DUAL;
```

## 2.6 运算查询

```sql
-- 可以直接在 SELECT 子句中执行数学表达式或函数运算
SELECT DISTINCT [数学表达式] AS [别名] FROM DUAL;
```

# 3 运算

## 3.1 运算陷阱

> [!danger] 隐患（坑点）
> `NULL` 参与绝大多数数学或逻辑运算时，其结果往往都是 `NULL`。即使是 `NULL = NULL` 的判断，逻辑结果也是 `NULL`（非真）。唯一例外是使用专用的判空运算符 `IS NULL` 或 `<=>`。

## 3.2 判断是否为空

```sql
-- 找出 [列名] 属性为 NULL 的记录
SELECT [列名] FROM [表名] WHERE [列名] IS NULL;

-- 找出 [列名] 属性不为 NULL 的记录
SELECT [列名] FROM [表名] WHERE [列名] IS NOT NULL;
```

## 3.3 指定查询的范围

```sql
-- 指定 [列名] 的值在 [起始值] 与 [结束值] 之间（包含边界）
SELECT [列名] FROM [表名] WHERE [列名] BETWEEN [起始值] AND [结束值];

-- 排除在 [起始值] 与 [结束值] 之间的数据
SELECT [列名] FROM [表名] WHERE [列名] NOT BETWEEN [起始值] AND [结束值];

-- 给 [列名] 指定一个离散的、具体的值集合范围
SELECT [列名] FROM [表名] WHERE [列名] IN ([值_1], [值_2]);

-- 排除离散的具体值集合
SELECT [列名] FROM [表名] WHERE [列名] NOT IN ([值_1], [值_2]);
```

## 3.4 找最大最小

```sql
-- 返回多个字段值或常量集中的最小值（需要最大值时使用 GREATEST）
SELECT LEAST([列名_1], [列名_2]) FROM [表名] WHERE [条件表达式];

-- 比较字符串长度并返回最短的长度数值
SELECT LEAST(LENGTH([列名_1]), LENGTH([列名_2])) FROM [表名] WHERE [条件表达式];
```

## 3.5 判等运算

```sql
-- 结果永远为空！绝对不能使用普通的等号直接判断 NULL
SELECT [列名] FROM [表名] WHERE [列名] = NULL;

-- 安全等于运算符（<=>），即使两端都是 NULL 也能安全返回 TRUE
SELECT [列名] FROM [表名] WHERE [列名] <=> NULL;

-- 结果永远为空！ != 或 <> 与 NULL 计算无结果，排除 NULL 必须使用 IS NOT NULL
SELECT [列名] FROM [表名] WHERE [列名] != NULL;
```

# 4 匹配查询

## 4.1 模糊匹配

```sql
-- 找出 [列名] 属性中包含字母 a 的记录。'%' 表示匹配任意数量的字符（包含 0 个）
SELECT [列名] FROM [表名] WHERE [列名] LIKE '%a%';

-- 找出 [列名] 属性第二个字符是 a 的记录。'_' 表示严格匹配单个占位字符，注意末尾别漏掉 '%'
SELECT [列名] FROM [表名] WHERE [列名] LIKE '_a%';
```

# 5 多表查询

## 5.1 笛卡尔积

### 5.1.1 概念

> [!note] 知识点标签：#分类/关系代数
> **什么是笛卡尔积？**
> 假设集合 A（颜色）= `{Red, Blue}`，集合 B（尺寸）= `{Small, Large, X-Large}`，它们的笛卡尔积就是两个集合元素的所有可能组合（共 2 * 3 = 6 种组合）：
> - Red + Small
> - Red + Large
> - ...以此类推。在 SQL 中，如果没有指定连接过滤条件，多表查询的返回结果直接就是笛卡尔积。

## 5.2 SQL 查询

### 5.2.1 执行多表查询

```sql
-- 结果是两张表所有行的交叉组合（笛卡尔积），无限制时通常伴随巨大的性能开销与脏数据
SELECT * FROM [表_1], [表_2];
```

### 5.2.2 消除无关系的笛卡尔积

```sql
-- 隐式连接：在 WHERE 子句中加入关联条件
SELECT [表_1.列名] FROM [表_1], [表_2] WHERE [表_1.关联列] = [表_2.关联列];

-- 显式连接（标准写法）：使用 JOIN ... ON 进行约束
SELECT [表_1.列名] FROM [表_1] JOIN [表_2] ON [表_1.关联列] = [表_2.关联列];
```

### 5.2.3 别名使用（最佳实践）

> [!tip] 最佳实践
> 为表起别名不仅可以使代码更加简洁，还能避免多张表出现同名字段时的引擎歧义报错。虽然 `AS` 关键字可以省略，但在工程实践中建议保留以增加源码的自我解释性。

```sql
SELECT [别名_1.列名] 
FROM [表_1] AS [别名_1] 
JOIN [表_2] AS [别名_2] 
ON [别名_1.关联列] = [别名_2.关联列];
```

### 5.2.4 多种连接方式

```sql
-- 【内连接 (INNER JOIN)】：只返回两张表中指定关联条件匹配的交集记录
SELECT [表_1.列名] 
FROM [表_1] JOIN [表_2] ON [表_1.关联列] = [表_2.关联列];

-- 【内连接语法糖：USING】：当两张表的关联字段名完全一致时，可简写关联条件
SELECT [表_1.列名] 
FROM [表_1] JOIN [表_2] USING([同名关联列]);

-- 【自然连接 (NATURAL JOIN)】：自动寻找两张表所有同名列作为关联条件（业务中极少使用，易因字段变更引发线上事故）
SELECT [表_1.列名] 
FROM [表_1] NATURAL JOIN [表_2];

-- 【左外连接 (LEFT JOIN)】：以左表为主表，即使右表没有匹配，左表数据也会被完全保留，右表缺失字段对应的空间补齐为 NULL
SELECT [表_1.列名] 
FROM [表_1] LEFT JOIN [表_2] ON [表_1.关联列] = [表_2.关联列];
```

> [!warning] 数据库版本差异
> **全外连接 (FULL OUTER JOIN)** 旨在保留两边表的全部数据（未匹配部分相互补 NULL）。但需要注意：**MySQL 原生不支持** 全外连接的专有关键字语法，生产中通常需要通过 `LEFT JOIN` 联合 (`UNION`) `RIGHT JOIN` 的组合变体来模拟实现。

# 6 联合查询

```sql
-- 【联合查询：去重合并】
-- 将两个结构一致的查询结果集垂直合并返回。UNION 默认会执行全局去重逻辑。
SELECT [列名] FROM [表名] WHERE [条件表达式_1]
UNION
SELECT [列名] FROM [表名] WHERE [条件表达式_2];

-- 【联合查询：不去重合并】（性能更优）
-- UNION ALL 直接合并结果集，不执行底层去重判断操作，业务适用时应优先使用。
SELECT [列名] FROM [表名] WHERE [条件表达式_1]
UNION ALL
SELECT [列名] FROM [表名] WHERE [条件表达式_2];
```

# 7最佳实践与证据

* **深分页（Deep Pagination）性能危机规避** — 当执行大跨度的分页语句如 `LIMIT 1000000, 10` 时，MySQL 引擎会被迫扫描前 1,000,010 行再丢弃前面的无效行，导致极严重的查询延迟。在系统架构层面，绝对不要暴漏超过指定深度的分页接口，底层 SQL 查询必须重构为“延迟关联分页（先基于聚簇索引查主键再 JOIN）”或“基于游标的条件流式查询（如 `WHERE id > 上一页最大ID LIMIT n`）”。 (证据：[MySQL 官方文档 - LIMIT Query Optimization 进阶策略](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html))