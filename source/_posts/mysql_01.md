title: mysql  null 
tags:
  - mysql
category: mysql
date: 2015-07-18 19:08:00

---

## 问题
今天在处理dbrt时，类似的建表语句如下：
```
CREATE TABLE `xxx`(
     ...
    `column1` int(11) not null
    ...
) 
```
对于这种字段，需要赋予默认值,然后有人说,我设置的不为null，就是不希望有空值插入进去,所以这里不需要设置default。

---
## 深入探究null 和 not null
首先，我们要搞清楚“空值” 和 “NULL” 的概念:
*   空值是不占用空间的
*   mysql中的NULL其实是占用空间的，下面是来自于MYSQL官方的解释

```
NULL columns require additional space in the row to record whether their values are NULL. For MyISAM tables, each NULL column takes one bit extra, rounded up to the nearest byte
```
所以空值还是会被插入到db中，只有NULL列才不会被插入。

## 为什么要设置NOT NULL
NULL列需要更多的存储空间，还需要mysql内部进行特殊处理。NULL列被索引后，每条记录都需要一个额外的字节，还能导致MYisam 中固定大小的索引变成可变大小的索引。这在《高性能mysql》中有介绍的,
所以为了尽量避免NULL，我们指定列为NOT NULL。
可以看出，在MySQL中，含有NULL的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值。
