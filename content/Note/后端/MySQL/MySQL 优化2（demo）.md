

# 1 explain 工具的使用补充

>[!warning] id 不一定表示sql的执行顺序
>id 表示的select 语句的查询块id,每个单独select 语句会有不同查询块，而不是执行顺序，在简单的sql 语句往往是查询块id越小 语句先执行，在union id会为NULL ，因为最后一步去重操作不需要select，这一步也没有select 语句就没有查询块


## 1.1 select_type

>[!note] 简介
>select_type 可以帮助我们反映一个select 语句的真实结构


 1 simple
表示简单的查询语句，就一个select块操作

2 primary 意为首要的，外面的， 在嵌套子查询和union 等有多个select 语句的操作才有，表示前面的或者外层的查询

3 union 联合查询，在联合查询中表示后面的查询语句

4  union result 表示联合查询去除重复，因为没有select,id 为null

>[!warning]  判断一个查询是否是相关还是不相关的子查询，就要判断里面的查询结果会不会被外层查询影响，被影响的是相关的，否则是不相关的，不相关子查询只执行一次

5 subquery 不相关子查询 非 primary 部分

6 dependent subquery 不相关子查询 非 primary 部分

7 derived 嵌套子查询，一个查询变成另外一个查询的输入的情况下， 非primary的部分

8 materialized 物化表  sql优化器把嵌套查询变成了多表连接查询， 这种情况下select_type 为 materialized 的查询部分会执行生成临时存在的表，名字为subquery2,然后拿去给前面的部分连接处理，好处是可以减少表的查询次数


## 1.2 type(重点)

null 不需要访问任何表的查询，比如查日期，这些不在磁盘上，在服务器内存里面速度最快

system 要求被查的数据是myisam引擎，字段和记录为1，经常被拿去给数据库放置常量，存在于内存，速度快

const 使用唯一的键匹配的查询

ref 使用非唯一的键匹配的查询

eq_ref  在多表连接使用唯一的键连接的查询

index_merge 在where 中使用or 的查询

