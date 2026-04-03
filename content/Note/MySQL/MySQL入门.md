

## 数据类型

1. INT
2. VARCHAR
3. .....


## 命令行登录

### 陷阱

1. MySQL 下的大部分指令后面要加;

```bash
mysql -u root -p
```
## 命令行退出
```SQLBash
exit
```

## 查看数据库
```SQLBash
show databases; // 不要丢失最后的s
```

## 查看表结构

```sql
DESC [table_name];
```


## 创建数据库

### 陷阱
1. 数据库名字不区分大小写，不要创建重复数据库
```SQLBash
create database [database_name]; 
```

## 选择数据库
### 解释

1. 后面的操作默认基于此数据库
```SQLBash
use [database_name]; 
```

## 查看表格

```SQLBash
show tables; 
```

## 创建表格

```SQLBash
create table [table_name](
[col_name] [data_type]
...
)
```

## 查询数据

```SQLBash
select * from [table_name]
```

### 插入数据
```SQLBash
insert into stu([col_name_1], [col_name_2]) values([row_val_1],[row_val_2]);
```

### 删除数据表
```SQLBash
drop table [table_name];
```

### 删除数据库
```SQLBash
drop database [database_name];
```

## 重置root用户密码

```Bash
mysqld --console --skip-grant-tables --shared-memory # 使用管理员权限
```



# 实践问题

 1. MySQL安装界面和教程不同（官网上面的8.0与课件里面的不一样,官网配置方式发生了改变）
 2. MySQL 8.0 在C:\ProgramData\MySQL\MySQL Server 8.0 目录下面没发现my.ini


# 理论问题

1. MySQL客户端访问服务端为什么使用TCP而不是UDP
2. 什么是线程池
3. MySQL 服务层是什么样子的
4. MySQL 的注册表起什么作用