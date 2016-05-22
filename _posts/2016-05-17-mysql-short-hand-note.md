---
title: "【MySQL】拾遗系列-随手记"
category: [tech]
tag: [mysql]
---


## MySQL的时区

> http://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html

在MySQL启动的时候就会读取系统的时区参数作为自己的时区设置

我们可以通过命令查看MySQL服务器的系统变量来获取当前设置的时区

> http://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_system_time_zone

```
mysqld --verbose --help
```

或者

```
mysqladmin variables
```

**关于设置**

* 在MySQL启动服务的时候加上参数 ```--timezone=timezone_name```
* 在启动后修改全局的时区 ```mysql> SET GLOBAL time_zone = timezone;```
* 在启动后修改某个连接的时区 ```mysql> SET time_zone = timezone;```
* 修改配置文件 ```default-time-zone='timezone'```

**要注意的是**

时区修改之后MySQL中某些函数或者变量的值会出现不一样,特别要注意的是 ```timestamp``` 类型的字段存储和读取是与时区相关的


```
存储时  timestamp + current time zone -> UTC
读取时 stored UTC + current time zone -> timestamp
```

**举个小例子**

```
create table timeset
(
    date date,
    datetime datetime,
    timestamp timestamp
);
 
insert into timeset
values(
    now(),
    now(),
    now()
);
 
select * from timeset;
 
# 修改时区到UTC+12, 可以发现timestamp的值发生了改变
set time_zone = '+12:00';
select * from timeset;
 
insert into timeset
values(
    now(),
    now(),
    now()
);
 
# 将时区改回去, timestamp总能忠实地反映插入的时间
set time_zone = '+8:00';
select * from timeset;
```

## DATE, DATETIME和TIMESTAMP

> http://dev.mysql.com/doc/refman/5.7/en/datetime.html

|     \     | Date part | Time part |                       Range                       |                              remark                             |
|:---------:|:---------:|:---------:|:-------------------------------------------------:|:---------------------------------------------------------------:|
|    DATE   |     O     |     X     |              1000-01-01 ~ 9999-12-31              |                                                                 |
|  DATETIME |     O     |     O     |     1000-01-01 00:00:00 ~ 9999-12-31 23:59:59     |                      可带毫秒数,与时区无关                      |
| TIMESTAMP |     O     |     O     | 1970-01-01 00:00:01 UTC ~ 2038-01-19 03:14:07 UTC | 可带毫秒数,读取和存储的时候都会根据本连接的time zone来与UTC转换 |

## 时间相关的运算

> http://dev.mysql.com/doc/refman/5.6/en/date-and-time-functions.html

关于时间的运算一定要使用时间的函数来进行.