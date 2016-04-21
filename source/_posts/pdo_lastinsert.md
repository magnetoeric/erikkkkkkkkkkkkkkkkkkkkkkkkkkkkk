title: pdo lastInserId问题
tags:
  - mysql 
  - php
category: php
date: 2015-07-18 19:05:55

---

##问题
同事问了一个问题:
框架底层对executeSql进行了封装(对sql进行了处理，做主从分离),在执行insert xxx values xxx on duplicate key update语句后，使用lastInsertId时返回结果是0。  

## 原因
由于要判断是否插入失败，所以分析了一下框架executeSql，除了select，desc 其他语句使用master进行处理(主要是为了实现框架级别的主从分离),在PDOStatement::execute之后，  
对操作语句的类型进行了判断,对于insert的返回，执行成功的结果都是返回PDO::lastInsertId,如果执行失败，则返回false.  
查看了PDO::lastInsertId 的说明手册
```
 string PDO::lastInsertId ([ string $name = NULL ] )
 返回最后插入行的ID，或者是一个序列对象最后的值，取决于底层的驱动。比如，PDO_PGSQL() 要求为 name 参数指定序列对象的名称。 
     Note:

    在不同的 PDO 驱动之间，此方法可能不会返回一个有意义或一致的结果，因为底层数据库可能不支持自增字段或序列的概念。  
```
但是我的数据库底层支持自增字段或序列啊，不应该返回0，结果仔细看表发现，由于表是从其他表中获取有效数据同步过来的,所以没有自增字段(auto-increment)，  
这样，判断executeSql是否执行失败，需要使用result === false来判断了