
# 1规范

sql 语句统一 关键词区别大写，表名等全部使用下划线加小写字母

# 2查询进阶

## 2.1别名

```sql
SELECT [col_name] FROM [table_name] AS [other_name] WHERE []; # AS 后面是表别名
```

## 2.2 指定查询顺序

```sql
SELECT [col_name] FROM [table_name] AS [other_name] WHERE [] ORDER BY 
[col_name_1] DESC,[col_name_2] ASC; # ORDER BY 开始指定顺序， DESC是指降顺，ASC是升序
```

## 2.3 分页查询

```sql
SELECT [col_name] FROM [table_name] AS [other_name] WHERE [] LIMIT [start_index] [page_count]; # start_index 是指代从第几行数据的索引开始，第一个数据索引为0

SELECT [col_name] FROM [table_name] AS [other_name] WHERE [] LIMIT [page_count] OFFSET [start_index]; # start_index 是指代从第几行数据的索引开始，第一个数据索引为0
```

## 2.4 查询过滤

```sql
SELECT DISTINCT [col_name] FROM [table_name] WHERE []; # DISTINCT 会过滤相同的数据行
```

## 2.5 常数查询

```sql
SELECT DISTINCT [const] AS [other_name] FROM DUAL; # 不需要查表使用DUAL代替表
```

## 2.6 运算查询

```sql
SELECT DISTINCT [math] AS [other_name] FROM DUAL; # math 代表一个数学表达式
```

## 3 运算

### 3.1陷阱
1. NULL 参与大多数运算不管结果是什么，结果往往是NULL， NULL = NULL 结果也是NULL

### 3.2 判断是否为空

```sql
SELECT [col_name] FROM [table_name] WHERE [col_name] IS NULL #找出[col_name]属性为空的
SELECT [col_name] FROM [table_name] WHERE [col_name] IS NOT NULL #找出[col_name]属性不为空的
```

### 3.3 指定查询的范围

```sql
SELECT [col_name] FROM [table_name] WHERE [col_name] BETWEEN [val_1] AND [VAL_2]
# 指定[col_name] 大小范围在 [val_1] 与 [VAL_2] 之间

SELECT [col_name] FROM [table_name] WHERE [col_name] NOT BETWEEN [val_1] AND [VAL_2]
# 效果和上面相反

SELECT [col_name] FROM [table_name] WHERE [col_name] IN ('[val_1]', '[val_2]')
# 给col_name 指定一个更加具体的范围

SELECT [col_name] FROM [table_name] WHERE [col_name] NOT IN ('[val_1]', '[val_2]')
# 给col_name 指定一个更加具体的范围
```
### 3.4 找最大最小

```sql
SELECT LEAST ([col_name_1],[col_name_2])  FROM [table_name] WHERE []
# 只显示[col_name_1],[col_name_2]中最小的属性，不是看长度，最大使用GREATEST

SELECT LEAST (LENGTH([col_name_1]),LENGTH([col_name_2]))  FROM [table_name] WHERE []
# 只显示[col_name_1],[col_name_2]中最短的属性，不是看长度，最大使用GREATEST

```
### 3.5等于
```sql
SELECT [col_name] FROM [table_name] WHERE [col_name] = NULL
# 结果为空

SELECT [col_name] FROM [table_name] WHERE [col_name] <=> NULL
# 把为NULL 全部找出来

SELECT [col_name] FROM [table_name] WHERE [col_name] ！= NULL
# 结果为空，！=， <>表示不等于,因为和NULL参与计算没有结果
```

## 4匹配

### 4.1 匹配查询

```sql
SELECT [col_name] FROM [table_name] WHERE [col_name] LIKE '%a%' #找出[col_name] 属性含a的， %表示匹配多个字符

SELECT [col_name] FROM [table_name] WHERE [col_name] LIKE '_a%' #找出[col_name] 属性第二字符含a的， _表示匹配一个字符，后面的%容易忘记
```


## 5多表查询

待续。。。