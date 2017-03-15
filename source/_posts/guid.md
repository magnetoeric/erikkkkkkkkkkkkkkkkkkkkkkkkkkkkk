title: guid
tags:
  - arch
categories: 
  - arch
date: 2016-11-29 19:55:00
---

# 分布式id生成器

## 概念

有些人也喜欢叫发号器，一般是作为服务运行，主要目的是就是为一个系统的数据对象分配一个唯一id。

## 特点
*   唯一性
*   时间段内粗略有序
*   可制造
*   可反解

一般来说，id生成器的要求主要是一下几点:

*   快速响应，可以毫秒级并发
*   高可用性
*   体积小(存储)

这里不讨论uuid[uuid-vs-int-insert-performance](http://kccoder.com/mysql/uuid-vs-int-insert-performance/)

## 生成方法
### 自增有序
这种方式可以利用mysql的自增id。但是使用mysql有一些瓶颈:

*   单点问题(高可用性)
*   并发


flickr巧妙的利用replace实现分布式发号器
自增id使用big int  为了提高并发，使用多台mysql机器，每台机器设置不同的起始值和步长，sql使用replace唯一id的方式使得mysql每次增长一个步长(insert on duplacate key也可以，update不可以,是因为update的返回值是成功还是失败)
表结构

```
CREATE TABLE `Tickets64` (
  `id` bigint(20) unsigned NOT NULL auto_increment,
  `stub` char(1) NOT NULL default '',
  PRIMARY KEY  (`id`),
  UNIQUE KEY `stub` (`stub`)
) ENGINE=MyISAM
```

sql语句

```
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();
```

设置步长和起始值(步长也可以直接在session中指定)

```
TicketServer1:
auto-increment-increment = 2
auto-increment-offset = 1

TicketServer2:
auto-increment-increment = 2
auto-increment-offset = 2
```

php 实现，java无法使用jpa，可以用jdbcTemplate实现

```
<?php
include_once 'pdoFactory.php';
// insert
//$sql = "INSERT INTO Tickets64 (stub) VALUES ('a') ON DUPLiCATE KEY UPDATE id=id+1";
// replace
$sql = "REPLACE INTO Tickets64 (stub) VALUES ('a')";

$pdo = pdoFactory::getInstance()->getPdo('default');
$p = $pdo->exec($sql);
echo "<pre>";print_r($pdo->lastInsertId());exit;
```

优缺点

优点是简单，实现快速。

缺点是mysql中需要设置起始值，需要多台mysql服务，可扩展性差。并发取决于mysql节点数量

架构图

![](http://githubforericwang.qiniudn.com/hexo/eric/autoincreament.png)

腾讯的seqsvr [万亿级调用系统：微信序列号生成器架构设计及演变](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2650992918&idx=1&sn=be5121c3c57257291a30715ef7130a90&scene=23&srcid=0628Xb1bDyZ2EdvPffymi6za#rd) 
简单来讲，seqsvr是一个巨大的long数组,每个元素代表一个用户，比如小明的uid是12,那么seq[12]就是他产生的guid，每个用户(元素)之间产生的uid互不影响,每个uid产生的guid严格递增(为了保证这一点，付出的代价很高，不能使用负载均衡，同一时间只有一个服务器产生该用户的guid)，发号分为两层，下层是持久化层，主要存储当前uid产生的最大的guid，上层为缓存层，缓存一个步长，用来发号。也做了很多优化，比如设置最大临时guid,相邻的uid共享该max_guid，这样在服务器重启时可以大大减少向存储层发起的请求，减少内存使用。容灾问题上做了很多处理，前面说到这种方式存在单点问题，seqsrv做了很多，分set做隔离，路由和配置服务做失败请求，仲裁服务做服务切换

### 时间有序
主流的方式都是类似twitter的snowflake，一个long是64字节，可以根据自身业务特点选择秒级别还是毫秒级别，一般系统寿命在30年左右，所以毫秒级别需要41位表示时间，秒级别需要30位，剩下的需要几位去表示机器号，也可以预留2位表示版本，方便切换
 
摘自[一乐:业务系统需要什么样的ID生成器](http://ericliang.info/what-kind-of-id-generator-we-need-in-business-systems/)

```
SnowFlake

41bit留给毫秒时间，10bit给MachineID，也就是机器要预先配置，剩下12位留给Sequence。

Weibo

微博使用了秒级的时间，用了30bit，Sequence 用了15位，理论上可以搞定3.2w/s的速度。用4bit来区分IDC，也就是可以支持16个 IDC，对于核心机房来说够了。剩下的有2bit 用来区分业务，由于当前发号服务是机房中心式的，1bit 来区分热备。是的，也没有用满64bit。

Ticktick

也就是当前在环信系统里要用到的。使用了30bit 的秒级时间，20bit 给Sequence。这里是有个考虑，第一版实现还是希望到毫秒级，所以20bit 的前10bit给了毫秒来用，剩下10bit给 Sequence。等到峰值提高的时候可以暂时回到秒级。

前面说到的三十年问题，因此我在高位留了2bit 做 Version，或者到时候改造使用更长字节数，用第一位来标识不同 ID，或者可以把这2bit 挪给时间用，可以给系统改造留出一定的时间。

剩下的10bit 留给 MachineID，也就是说当前 ID 生成可以直接内嵌在业务服务中，最多支持千级别的服务器数量。最后有2bit 做Tag 用，可能区分群消息和单聊消息。同时你也看出，这个 ID 最多支持一天10亿消息，也是怕系统增速太快，这2bit 可以挪给 Sequence，可以支持40亿级别消息量，或者结合前面的版本支持到百亿级别。

修正：评论里指出上面一个计算错误，不挪借的话应该是支持一天约千亿级别。对比当前 Whatsapp 的600亿和腾讯 QQ 的200亿，已经足够了。
```

snowflake

优点:可用性强

缺点:依赖zookeeper，要么人肉做配置，总之需要在服务中配置机器号识别。


Instagram
[sharding-ids-at-instagram](http://instagram-engineering.tumblr.com/post/10853187575/sharding-ids-at-instagram)

```
CREATE OR REPLACE FUNCTION insta5.next_id(OUT result bigint) AS $$
DECLARE
    our_epoch bigint := 1314220021721;
    seq_id bigint;
    now_millis bigint;
    shard_id int := 5;
BEGIN
    SELECT nextval('insta5.table_id_seq') %% 1024 INTO seq_id;

    SELECT FLOOR(EXTRACT(EPOCH FROM clock_timestamp()) * 1000) INTO now_millis;
    result := (now_millis - our_epoch) << 23;
    result := result | (shard_id << 10);
    result := result | (seq_id);
END;
$$ LANGUAGE PLPGSQL;
```
也是类似snowflake,直接在postgrsql中实现的。简单看了一下，类似触发器，id自增换成函数生成
达达的也类似[达达-高性能服务端优化之路](http://www.infoq.com/cn/articles/imdada-high-performance-server-optimization/)

优： 开发成本低
劣： 基于postgreSQL的存储过程，较为偏门

基于redis的id生成器

基于redis的id生成器(http://blog.csdn.net/hengyunabc/article/details/44244951)

优点:利用redis lua脚本实现，少量机器即可满足需求

缺点: 需要配置redis起始值及步长

一些变种

Hibernate [github](https://github.com/hibernate/hibernate-orm/blob/master/hibernate-core/src/main/java/org/hibernate/id/uuid/CustomVersionOneStrategy.java)

```
- 时间戳(6bytes, 48bit)：毫秒级别的，从1970年算起，能撑8925年....
- 顺序号(2bytes, 16bit, 最大值65535): 没有时间戳过了一秒要归零的事，各搞各的，short溢出到了负数就归0。
- 机器标识(4bytes 32bit): 拿localHost的IP地址，IPV4呢正好4个byte，但如果是IPV6要16个bytes，就只拿前4个byte。
- 进程标识(4bytes 32bit)： 用当前时间戳右移8位再取整数应付，不信两条线程会同时启动。

值得留意就是，机器进程和进程标识组成的64bit Long几乎不变，只变动另一个Long就够了。
```


mongodb [Mongdodb objectId](https://docs.mongodb.com/manual/reference/bson-types/#objectid) 非long型

```
- 时间戳(4 bytes 32bit): 是秒级别的，从1970年算起，能撑136年。

- 自增序列(3bytes 24bit, 最大值一千六百万)： 是一个从随机数开始（机智）的Int不断加一，也没有时间戳过了一秒要归零的事，各搞各的。因为只有3bytes，所以一个4bytes的Int还要截一下后3bytes。

- 机器标识(3bytes 24bit): 将所有网卡的Mac地址拼在一起做个HashCode，同样一个int还要截一下后3bytes。搞不到网卡就用随机数混过去。

- 进程标识(2bytes 16bits)：从JMX里搞回来到进程号，搞不到就用进程名的hash或者随机数混过去。

可见，MongoDB的每一个字段设计都比Hibernate的更合理一点，比如时间戳是秒级别的。总长度也降到了12 bytes 96bit，但如果果用64bit长的Long来保存有点不上不下的，只能表达成byte数组或16进制字符串。


```







## 参考
[江南白衣:分布式Unique ID的生成方法一览](http://calvin1978.blogcn.com/articles/uuid.html)

[一乐:业务系统需要什么样的ID生成器](http://ericliang.info/what-kind-of-id-generator-we-need-in-business-systems/)

[flickr Ticket Servers: Distributed Unique Primary Keys on the Cheap](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)
