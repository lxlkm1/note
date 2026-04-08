
#MySQL函数 #聚和函数 #子表查询


# 1 MySQL函数


## 1.1 常用

### 1.1.1时间

#### 1.1.1.1获取时间

```SQL
/*
* @fuction:获取系统当前时间，包括小时和分钟秒
*
*/
NOW() 
/*
* @fuction:获取系统当前时间，不包括小时和分钟秒
*
*/
CURDATE()
```
#### 1.1.1.2处理时间

1. 时间可以直接使用 > < = 符号直接比较
2. 比较日期差可以使用DATEDIFF() 函数去比较
```SQL
# 返回date_1 - date_2的天数
DATEDIFF([date_1], [date_2])  
# 返回time_1 - time_2的时间差，时间差距小于24h使用
TIMEDIFF([time_1], [time_2])  
```


### 1.1.2 字符串

#### 1.1.2.1连接字符串

```SQL
CONCATE([str_1]， [str_2], [str_3] ....)

# 每个字符中间加入x
CONCATE_W([str_1]， [str_2], [str_3] ...., x)
```

#### 1.1.2.2大小写转换

```SQL
LOWER()
UPPER()
```

#### 1.1.2.3获取字符串长度

```SQL
# 和字符集有关系
LENGTH() 
# 和字符集没有关系，推荐中文使用
CHAR_LENGTH()
```


#### 1.1.2.4获取字符串长度

```SQL
# 和字符集有关系
LENGTH() 
# 和字符集没有关系，推荐中文使用
CHAR_LENGTH()
```


#### 1.1.2.5处理字符串空格

```SQL
LTRIM()
RTRIM()
TRIM()
```

#### 1.1.2.6替换字符串

```SQL
# 使用b 替代s1出现的a
REPLACE(s1， a, b)
```


### 1.1.3 流程控制

```SQL
# 如果val_1 为 true 就返回 [val_2] 否则 val_3
IF([val_1], [val_2], [val_3])

# 如果val_1 不为 NULL 就返回val_1, 否则val_2,通常把NULL 换成 0
IFNULL([val_1], [val_2])
```

# 2 聚和函数


## 2.1常用函数

```SQL
# 获取参数中最小的值
MIN([val_1],[val_2]...)

# 获取参数中最大的值
MAX([val_1],[val_2]...)

# 获取参数中平均值
AVG([val_1],[val_2]...)

# 获取col_name 不为空的记录数量
COUNT([col_name])

```

## 2.2 分组查询



```SQL
# 分组查询常搭配 sql 函数，常用于完成获取一些部门平均的工资等等任务
SELECT [col_1] [sql_fuc]([col_2])
FROM [table_name]
GROUP BY
[col_x]
# having 类似于where,但是having是约束分组之后的数据，比如过滤掉一些平均工资高于多少的部门
HAVING []
```

# 3子表查询


##  3.1抽象


想象一下在一张员工表，我需要查询king的下属的信息，那正常情况下，我们需要经过两次查询，写两次查询语句，第一次查询king的员工id，然后根据king的员工id去查询他的下属，子表查询就是写一次查询语句即可

```sql
# 假设返回king_employee_id
SELECT e.employee_id FROM employees e where e.last_name = 'king';
SELECT * FROM employees e where e.manager_id = king_employee_id;

# 使用子表查询
SELECT * FROM employees e where e.manager_id = (
SELECT e2.employee_id FROM employees e2 where e2.last_name = 'king'
);

```

## 3.2 子表查询类型

### 3.2.1相关子查询

把子表放在where后面 就是相关子查询，外层查询每次查询一个数据，就要完成一次子查询，子查询要经历多次


### 3.2.2不相关子查询

把子表放在select后面 就是不相关子查询，把子查询当作一张新表，只经历一次子查询，效率比相关子查询性能更高



## 3.3 多行子查询


### 3.3.1抽象


多行子查询顾名思义就是子查询的结果有多行，这个时候需要搭配 ALL, ANY 等关键词，把查询结果的多行变成单行

```sql
# 查询工资最高的人的信息
SELECT * FROM employees e
WHERE e.salary > ALL
(SELECT e2.salary FROM 
employees e2);

# 查询工资不是最低的人的信息
SELECT * FROM employees e
WHERE e.salary > ANY
(SELECT e2.salary FROM 
employees e2);
```

# 4 EXISTS 和 NOT EXISTS 的使用

## 4.1抽象

如果在我们不关注子表查询的具体数据，而在在意子表查询是否有结果，就引入了 EXIST, 这通常搭配相关子查询

```sql

# 查询有领导的员工信息,这种写法仅仅展示
SELECT * FROM employees e
WHERE EXISTS
(SELECT * FROM 
employees e2 WHERE e.manager_id = e2.employee_id);


# 查询无领导的员工信息,这种写法仅仅展示
SELECT * FROM employees e
WHERE NOT EXISTS
(SELECT * FROM 
employees e2 WHERE e.manager_id = e2.employee_id);

