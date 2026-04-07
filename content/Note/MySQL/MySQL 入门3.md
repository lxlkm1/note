
#MySQL函数 #聚和函数


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


未完待续。。。