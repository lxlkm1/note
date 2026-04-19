
#MySQL #数据库 #后端

# 1知识点回顾

## 1.1 数据库引擎

>[!note] 概述
>数据库引擎就是数据库存储数据的部分，决定了存储数据的方式，而数据库引擎之外的部分就是数据库服务器

>[!danger] 混淆点
>在MySQL, 数据库引擎不是数据库级的，而是表级的，一个数据库可以有多个使用不同数据库的表
## 1.1.1 数据库引擎的种类

MySQL数据库引擎常见的是Innodb 和 Myisam，MySQL 8.0 创建数据库的默认数据库引擎是Innodb

## 1.1.2 不同数据库引擎的区别(Innodb, Myisam)


#### 1.1.2.1 功能区别（面试重点）

_> Innodb 支持行锁，外键，事务。而Myisam 支持表锁，不支持外键，事务。

#### 1.1.2.2 索引 

>[!note] 概述
>索引就类似于一本书的目录，而数据记录就是这本书的内容，这本书的目录可以从内容不同的角度去指定，一本书可以存在多个目录，目录可以加快我们查询内容的速度，但是目录也会增加内容维护的难度，当内容发生改变的时候，一般情况下一张表不会存在多于6的索引，当一索引指定的字段出现在where里面，索引采用可能发挥作用，不然就全表扫描

>[!danger] 误区
>1 不是说建立了索引，数据库就一定会使用索引，当数据量比较小的时候，数据库优化器认为走索引反而更慢,就不会使用索引



##### 1.1.2.2.1 索引常用SQL

1 建立索引(建表前)
```mysql
-- 普通索引
CREATE TABLE [表名](
[字段1] [字段1类型],
[字段2] [字段2类型],
[字段3] [字段3类型],

INDEX [索引名字] ([索引字段])
);

-- 唯一索引
CREATE TABLE [表名](
[字段1] [字段1类型],
[字段2] [字段2类型],
[字段3] [字段3类型],

UNIQUE INDEX [索引名字] ([索引字段])
);

-- 联合索引
CREATE TABLE [表名](
[字段1] [字段1类型],
[字段2] [字段2类型],
[字段3] [字段3类型],

UNIQUE INDEX [索引名字] ([索引字段], [索引字段])
);

```

2 建立索引(建表后)
```mysql
-- 方式1
Alter TABLE [表名字]
ADD INDEX [索引名字]([索引字段]);
-- 方式2
ADD INDEX  [索引名字]([索引字段]) ON [表名字];
```

3 删除索引(建表后)
```mysql
Alter TABLE [表名字]
DROP INDEX [索引名字];
```

4 查询是否使用了索引
```mysql
EXPLAIN [sql语句];
```

##### 1.1.2.2.2 如何使索引尽可能被使用

>[!note] 最左连续法则(适用于联合索引，面试题)
>比如建立了一个联合索引，索引顺序是 字段1，字段2，字段3，我们建立了这个索引有三种情况，生效。部分生效，完全不生效，我们只要在where 没有使用字段1，我们就是完全不生效，如果只使用了字段3,那就字段1，2生效，如果所有字段都被使用就是完全生效

###### 1.1.2.2.2.1 适合建立索引的字段

1 字段经常出现在where, group, order 关键词里面

2 字段具有唯一性，这样的字段通常被做成唯一索引

3 字段类型占用空间比较小的情况，这样每个数据页可以有更多的叶子节点和非叶子节点，B+树会变得更矮

4 经常需要去重的字段

###### 1.1.2.2.2.1 不适合建立索引的字段

1 字段值没有规律，没有大小变化，像人的身份证，不能够保证下一条插入的一定递增

2 数据库的数据量比较小不需要建立索引

3 经常更新的表无需过多索引，经常修改意味着，要同时改动许多需要维护的索引表

4 有大量重复数据的字段不需要索引

5 已经出现在联合索引的字段或者作为主键的字段不需要再设置索引


#### 1.1.2.3 存储结构区别

##### 1.1.2.3.1 聚集索引下

_> 他们存储数据的结构整体都是B+ 树，他们最大的区别在于叶子节点保存数据的形式，Myisam 的叶子节点保存的是索引和指向真实数据的物理位置指针，而Innodb下，B+ 树的叶子节点是完整的记录。


<figure>
  <img src="Pasted image 20260420001914.png" width="500" alt="B+树结构">
  <figcaption style="text-align:center; font-size:0.9em; color:gray;">
    图 1：Inno B+树索引的页结构示意图
  </figcaption>
</figure>


<figure>
  <img src="Pasted image 20260420002630.png" width="500" alt="B+树结构">
  <figcaption style="text-align:center; font-size:0.9em; color:gray;">
    图 2：Myisam B+树索引的页结构示意图
  </figcaption>
</figure>

##### 1.1.2.3.2 非聚集索引（辅助索引）下

>[!note] 补充
>1 所谓的辅助索引和聚集索引最大的区别就在于选择字段作为索引的不同，聚集索引就是使用主键作为索引，而使用非主键的字段制作的索引就叫做辅助索引，当同时作为索引的字段数量多于1的时候，这个索引也就叫联合索引了
>
>2 约束和辅助索引的关系，在有的时候一些约束需要通过辅助索引去实现，比如UNIQUE, 数据唯一约束需要在数据被插入的时候判断该数据是否破坏此约束，此时需要用唯一索引，把唯一字段做成唯一索引，去快速查找被约束的字段数据，而NOT NULL 约束不需要借助于索引

_> 在Myisam 数据库引擎里，辅助索引和聚集索引唯一的区别就在于制作索引选择字段的不同，而在Innodb 里，辅助索引叶子节点不再保存完整数据记录，而是保存主键的值，如果找数据使用了辅助索引，那到了主键的值，那么Innodb 还需要主键值去聚集索引去查找，这个叫回表




### 1.1.3常用SQL

1 查看默认的数据库引擎

```mysql
SELECT @@default_storage_enine;
```

2 修改默认的数据库引擎(不要重启)

```mysql
SET @@default_storage_enine := [值]；
```



## 1.2 触发器

>[!note] 运用场景
>触发器就是在改动数据库某个表的时候触发一些操作，比如在新表里面同时写入一些数据，这就决定了它和适合去写日志表

```mysql

-- 事前准备
CREATE TABLE emp(
  id INT(11) NOT NULL AUTO_INCREMENT ,
  NAME VARCHAR(50) NOT NULL COMMENT '姓名',
  age INT(11) COMMENT '年龄',
  salary INT(11) COMMENT '薪水',
  PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8 ;

CREATE TABLE emp_logs(
  id INT(11) NOT NULL AUTO_INCREMENT,
  operation VARCHAR(20) NOT NULL COMMENT '操作类型, insert/update/delete',
  operate_time DATETIME NOT NULL COMMENT '操作时间',
  operate_id INT(11) NOT NULL COMMENT '操作表记录的ID',
  operate_params VARCHAR(500) COMMENT '操作参数',
  PRIMARY KEY(`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

-- 建立触发器
DELIMITER $

CREATE TRIGGER emp_logs_insert_trigger
AFTER INSERT 
ON emp 
FOR EACH ROW 
BEGIN
  INSERT INTO emp_logs (id,operation,operate_time,operate_id,operate_params) VALUES(NULL,'insert',NOW(),new.id,CONCAT('插入后(id:',new.id,', name:',new.name,', age:',new.age,', salary:',new.salary,')'));	
END $

DELIMITER ;

DELIMITER $

CREATE TRIGGER emp_logs_update_trigger
AFTER UPDATE 
ON emp 
FOR EACH ROW 
BEGIN
  INSERT INTO emp_logs (id,operation,operate_time,operate_id,operate_params) VALUES(NULL,'update',NOW(),new.id,CONCAT('修改前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,') , 修改后(id',new.id, 'name:',new.name,', age:',new.age,', salary:',new.salary,')'));                                                                      
END $

DELIMITER ;

DELIMITER $

CREATE TRIGGER emp_logs_delete_trigger
AFTER DELETE 
ON emp 
FOR EACH ROW 
BEGIN
  INSERT INTO emp_logs (id,operation,operate_time,operate_id,operate_params) VALUES(NULL,'delete',NOW(),old.id,CONCAT('删除前(id:',old.id,', name:',old.name,', age:',old.age,', salary:',old.salary,')'));                                                                      
END $

DELIMITER ;

-- 验证结果
INSERT INTO emp(id,NAME,age,salary) VALUES(NULL, '冲虚道长',30,3500);
INSERT INTO emp(id,NAME,age,salary) VALUES
(NULL,'金毛狮王谢逊',55,3800),
(NULL,'飞天蝙蝠柯镇恶',60,4000),
(NULL,'全真七子丘处机',38,2800),
(NULL,'西毒欧阳锋',42,1800);

UPDATE emp SET age = 39 WHERE id = 3;
DELETE FROM emp WHERE id = 5;

SELECT * FROM emp_logs;

```