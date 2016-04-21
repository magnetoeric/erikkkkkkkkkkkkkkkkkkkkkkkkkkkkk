title: mysqldump 
tags:
  - mysql
category: mysql
date: 2015-08-10 10:28:00

---
## mysqldump
测试环境数据寥寥无几，测试又不会批量生成数据,只能从备份数据库中拉出来一些小表的数据填充
命令行下具体用法如下： 

mysqldump -u用戶名 -p密码 -d 数据库名 表名 > 脚本名

* 导出整个数据库结构和数据
```
mysqldump -h localhost -uroot -p123456 database > dump.sql
```
* 导出单个数据表结构和数据
```
mysqldump -h localhost -uroot -p123456  database table > dump.sql
```

问题来了,
Got error: 1044: Access denied for user '*****' to database '****' when doing LOCK TABLES

应该是数据库user的锁表权限问题,在导表时会锁表。加上--skip-lock-tables 就可以了
```
mysqldump -h localhost -uroot -p123456  --skip-lock-tables database table  > dump.sql
```

## information_schema
在对表做操作时,最好看一下表的一些信息

```
select * from information_schema. tables where talbe_name = 'xxx';#真是醉了 这个sql竟然不让放到blog里，我把.tables中间加了个空格
```
关于information_schema,具体看[这里](http://en.wikipedia.org/wiki/Information_schema)



