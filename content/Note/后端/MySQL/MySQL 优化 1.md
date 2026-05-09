
#优化 #mysql #后端 


# 1 找慢SQL

>[!note] 定义
>慢sql 就是执行时间超过一个阈值的sql 语句，这个阈值由我们自己去设置，默认是10s

## 1.1 定义慢sql 阈值

>[!danger] 注意点
>需要重新打开一个会话才生效

```mysql
-- 看慢sql阈值
SHOW VARIABLES like 'long_query_time';
-- 新定义慢sql阈值
SET GLOBAL long_query_time = [慢sql阈值];
-- 检查修改结果
SHOW VARIABLES like 'long_query_time';
```

## 1.2 开启慢sql 日志

>[!note] 适用场景
>该日志仅仅适用于存在慢sql,但是不知道哪个sql是慢sql的情况

>[!danger] 注意点
>进入生产环境要记得关闭此服务，不然会影响数据库性能

```mysql
-- 查看是否开启了日志
SHOW VARIABLES like 'slow_query_log';
-- 开启日志
SET GLOBAL slow_query_log = 'ON';
-- 检查
SHOW VARIABLES like 'slow_query_log';
```

## 1.3 寻找慢sql 日志位置

```mysql
-- 查看日志位置
SHOW VARIABLES like 'slow_query_log_file';
```

![[Pasted image 20260428233051.png]]

![[Pasted image 20260428233123.png|468]]
![[Pasted image 20260428234909.png|486]]
# 2 分析sql 执行慢的原因

## 2.1 寻找sql 哪一步慢

>[!note] 适用场景
>当我们要发现sql 整体很慢，但是不知道哪一步慢的使用使用

```mysql
EXPLAIN ANALYZE [要分析的sql 语句];
```

example

input:
```mysql
 EXPLAIN ANALYZE
  SELECT * FROM user_data
  UNION
  SELECT * FROM user_data;
```

output:
```shell
 -> Table scan on <union temporary>  (cost=2.50 rows=0) (actual time=0.003..157.150 rows=1000000 loops=1)
      -> Union materialize with deduplication  (actual time=30415.647..30715.562 rows=1000000 loops=1)
          -> Table scan on user_data  (cost=102549.60 rows=996938) (actual time=9.640..1027.584 rows=1000000 loops=1)
          -> Table scan on user_data  (cost=102549.60 rows=996938) (actual time=0.742..1141.142 rows=1000000 loops=1)
```

这里的时间都是毫秒，而且表示的都是大概时间，越里面的操作是最先被执行的操作，在这里我们可以发现在Union 操作花费的时间是30415.647 ms 到 30715.562ms 样子，我们就知道了这个sql最慢的地方在哪

## 2.2 寻找sql 为什么慢

>[!note] 适用场景
>当我们要发现sql 很慢，但是不知道为什么慢可以使用此工具分析，看看有没有走索引等等

```mysql
EXPLAIN [要分析的sql 语句];
```

output:
![[Pasted image 20260429001245.png]]

>[!note] 补充说明
>这里的id表示sql 操作的执行顺序，id 越大的越早执行，如果id一样的，就越上面越先执行，id为null 代表 union 去重 操作，最后一步执行


# 3 统计sql 的执行情况

>[!note] 适用场景
>在一个会话里面测试sql 执行速度的时候，而不是在全局测试的时候

## 3.1 开启性能测试(会话级别)

```mysql
-- 检查是否打开
SHOW VARIABLES like 'profiling';
-- 打开
SET profiling := 'ON';
SHOW VARIABLES like 'profiling';
```

然后在该会话执行一些sql 语句

## 3.2 获得性能统计

```mysql
SHOW profiles;
```

output:
![[Pasted image 20260429002007.png]]

Duration 就是每个sql 语句的执行时间