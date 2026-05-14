

# 1 知识回顾

>[!note] 回表补充
>有的时候使用联合索引或者其他辅助索引不一定会回表，此现象叫做过覆盖索引,触发条件是需要索引至少部分生效并且where里面的条件必须只能有联合索引里面指定的key，SELECT 查询的字段，全部包含在索引中,发生覆盖索引的时候不需要回表，查询性能更高

比如建立一个联合索引 (key1,key2,key3)

```mysql
SELECT key2, key3 FROM tablexx WHERE key1 = xxx
```

key2, key3 在联合索引里面，并且key1 生效，所以不需要回表

# 2 EXPLAIN 工具补充

## 2.1 possible_keys 和 key

possible_key 就是在sql 语句中看来可能会被使用的索引，而key是经过sql 优化器评估后真正使用的索引，联合索引通常会全部被使用，而多个单独的索引，sql优化器通常会选择一个代价最低的索引。如果使用多个单索引，那mysql 需要扫描多个b+ 树，然后合并索引这些通常比较消耗性能，有时不如选择一个使用性能更高的单索引

## 2.2 key_len

key_len 表示联合索引生效的数量，值越大联合索引中生效的索引越多


## 2.3 ref

ref 表示使用索引字段过滤数据的方式
比如建立一个联合索引 (key1,key2,key3)
```mysql
SELECT * FROM tablexx WHERE key1 = xxx AND key2 = xxx AND key3 = xxx;
```
这种查询ref 通常是 3 个const,表示使用了三个等值匹配


## 2.4 rows 和 filter

rows 是数据库认为需要查的数据，但是还没有检查是否全部符合条件，filter 表示rows 有多少数据符合要求，rows 越大表示查询性能低，filter 越低表示越多数据白查了
比如 WHERE age=20 AND address='xxx'，符合age 条件的数据有5w条，而其中符合address 条件的数据只有1w条，数据库提前查询了5w条数据，但是只有1w条数据符合要求，filter是20%


## 2.5 extra 

extra 是对当前sql执行步骤进行额外的说明

### 2.5.1 效率

高效 using index | using index condition

低效 using where | using join buffer | using filesort | using temporary

### 2.5.2 详细解释

using index

指查询数据的时候使用二级索引/联合索引就可以查出需要数据的情况下，不需要回表

using index condition(索引下推)

指的是查询数据的时候本来不需要走索引的查询却走了索引，而且也不需要回表

假设存在一个联合索引 key1,  key2 , key3
key4 不在索引里面

假设有一个sql 语句

```mysql
SELECT key1, key2 FROM tablexx WHERE key3 = xx OR key4 = xx;
```

key3 这个条件需要走索引，但是key4 只能全表扫描，所以mysql 就不如全部走全表扫描，把两个条件都检查一遍，这样还可以少一个走索引的步骤，这个会导致索引失效

而
```mysql
SELECT key1, key2 FROM tablexx WHERE key3 = xx AND key4 = xx;
SELECT key1, key2 FROM tablexx WHERE key3 = xx AND key2 LIKE xx;
```

本来mysql 需要两步操作，使用key3走索引和key4 去全表扫描，但是数据库却在key3走索引的时候顺便检查了AND 后面的条件，这个就是索引下推

>[!note] 索引实现的其他情况
>给索引加上like 通配符也会导致索引失效

比如
```mysql
SELECT key1, key2 FROM tablexx WHERE key3 LIKE xx;
```

---

using where

表示WHERE 后面条件是非索引字段

```mysql
EXPLAIN SELECT * FROM s1 WHERE key4 = 'abc';
```

using join buffer

```mysql
EXPLAIN SELECT * FROM s1 INNER JOIN s2 ON s1.`common_field` = s2.`common_field`;
```

表示在进行多表连接的时候,有一张表会被全表扫描，每次扫到对应字段的值，就使用那个字段的值去另一张表去查数据，如果对应的字段没有被建立索引，那第二步骤的查询会变得很慢，于是数据库把两张表的数据放入服务器缓冲区（内存），提升查询速度

using filesort

表示在给查询结果排序的时候使用了非索引字段排序，在B+ 树中，数据本身就是根据索引的大小排序好了的，如果没有使用索引里面的字段排序就需要再给数据排序，在磁盘排序太慢，mysql 会在内存创建临时文件，在内存里面排序

using temporary

表示在分组排序时，group by  后面字段不在索引，所以数据不是按 group by 字段排好的，所以 MySQL 要先“整理”一下数据，再统计。整理过程中可能会使用临时表





