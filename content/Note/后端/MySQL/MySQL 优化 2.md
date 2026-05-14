

# 1 explain 工具的使用补充

>[!warning] id 的定义
>id 表示的select 语句的查询块id,每个单独select 语句会有不同查询块，一般情况下我们可以认为id越大的越先被执行，如果id一样自上而下执行，在union id会为NULL ，因为最后一步没有select 操作


## 1.1 select_type

>[!note] 简介
>select_type 可以帮助我们反映一个select 语句的真实结构


 1 simple
表示简单的查询语句，就一个select块操作

2 primary 意为首要的，外面的， 在嵌套子查询和union 等有多个select 语句的操作才有，表示前面的或者外层的查询

3 union 联合查询，在联合查询中表示后面的查询语句

4  union result 表示联合查询去除重复，因为没有select,id 为null

>[!warning] 
> 判断一个查询是否是相关还是不相关的子查询，就要判断里面的查询结果会不会被外层查询影响，被影响的是相关的，否则是不相关的，不相关子查询只执行一次

5 subquery 不相关子查询 非 primary 部分

6 dependent subquery 不相关子查询 非 primary 部分

7 derived 嵌套子查询，一个查询变成另外一个查询的输入的情况下， 非primary的部分
这个情况下primary 操作对应的表的名字是derived + select_type 为 derived 的操作的id

>[!note] 为什么要把嵌套子查询变成多表查询(fix)
>嵌套子查询中相关子查询情况下，子查询可能会被执行多次，如果变成多表查询原来的子查询只需要执行一次

8 materialized 物化表  sql优化器把嵌套查询变成了多表连接查询， 这种情况下select_type 为 materialized 的查询部分会执行生成临时存在的表，名字为subquery + id,然后拿去给前面的部分连接处理，好处是可以减少表的查询次数


## 1.2 type(重点)

>[!note] 简介
>type 可以帮助我们反映查询的性能，以下性能由高到低

null 不需要访问任何表的查询，比如查日期，这些不在磁盘上，在服务器内存里面速度最快

system 要求被查的数据是myisam引擎，字段和记录为1，经常被拿去给数据库放置常量，存在于内存，速度快，常被用于保存mysql 服务器常量

const 使用非唯一键或者主键匹配的查询

ref 使用非唯一键或者主键的键匹配的查询(大多数查询)

eq_ref  在多表连接使用唯一的键连接的查询，在多表查询里面，会有一张表会被全量扫描，而没有被全量扫描的表的查询方式就是eq_ref，具体有哪张表被全量扫描取决于sql优化器

index_merge 在where 中使用or 的查询

ALL 全量扫描，没有走任何索引的查询



