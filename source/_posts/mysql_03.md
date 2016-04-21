title: MYSQL大小写问题
tags:
  - mysql
category: mysql
date: 2015-11-13 14:14:01

---
这个问题坑了自己2次
第一次  搭建superdiamond的时候，给出的表名都是大写的，但是启动后报错，原因是没有找到表
第二次  比较惨，改了很久，数据库里存储的用户名是大写的，取出用户名后，使用的是小写符号做的加密，自己被坑的很惨，因为测试环境里并没有大写的名字

今天比较空闲，学一下吧


```
insert into test values('A');  
insert into test values('a'); 
```
如果对其做主键约束的话，是会报错的，Duplicate entry。

MySQL在Linux下数据库名、表名、列名、别名大小写规则是这样的：

1、数据库名与表名是严格区分大小写的；
2、表的别名是严格区分大小写的；
3、列名与列的别名在所有的情况下均是忽略大小写的；
4、变量名也是严格区分大小写的；

**MySQL在查询字符串时是大小写不敏感的，在编绎MySQL时一般以ISO-8859字符集作为默认的字符集，这个字符集对大小写不敏感,因此在比较过程中中文编码字符大小写转换造成了这种现象。**


第1条就可以解释superdiamond为什么启动时会找不到表名，因为建表语句是大写的，而到了代码层，则是常规写法。

而如果想在插入时区分大小写，可以强制字段值需要设置BINARY属性

一些参考：
http://www.iteye.com/topic/766135
http://stackoverflow.com/questions/3936967/mysql-case-insensitive-select
http://dev.mysql.com/doc/refman/5.0/en/case-sensitivity.html
