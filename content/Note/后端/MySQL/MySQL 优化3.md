

# 1 知识回顾(demo)

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

假设存在一个联合索引 key1,  key2 , key3
key4 不在索引里面

未完待续。。。。

