title: memcache 超详细介绍
date: 2016-04-23 23:43:10
tags:
---
# Memcache

##  简介

### MemCache是什么

[MemCache](http://memcached.org/)是一个自由、源码开放、高性能、分布式的分布式内存对象缓存系统，用于动态Web应用以减轻数据库的负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高了网站访问的速度。MemCaChe是一个存储键值对的HashMap，在内存中对任意的数据（比如字符串、对象等）所使用的key-value存储，数据可以来自数据库调用、API调用，或者页面渲染的结果。MemCache设计理念就是小而强大，它简单的设计促进了快速部署、易于开发并解决面对大规模的数据缓存的许多难题，而所开放的API使得MemCache能用于Java、C/C++/C#、Perl、Python、PHP、Ruby等大部分流行的程序语言。

### Memcache 和Memcached
* MemCache是项目的名称

* MemCached是MemCache服务器端可以执行文件的名称

## 特点
* 无容灾考虑，纯内存缓存，重启后所有数据丢失。存取数据比硬盘快，当内存达到上限后，通过LRU算法自动删除缓存。
* 基于libevent开发。将Linux的epoll、BSD类操作系统的kqueue(多路复用io模型)等事件处理功能封装成统一的接口,因此memcached在多种操作系统上都可以发挥较好的性能，即使服务器的连接数增加，也能发挥O(1)的性能([C10k问题](http://www.kegel.com/c10k.html))
* 协议基于文本行，直接通过telnet在memcached服务器上可进行存取数据操作
* 分布式，memcache算不上是一个真正的分布式系统，因为各个memcached之间并不会“感知“，不会互相通信。分布式部署取决于memcache客户端。

## 原理

### 路由算法
前面说到memcached服务之间并不能互相通信，所以对服务的“选择”落在了客户端上，所以路由算法决定了客户端对哪台memcached进行数据存取。      
先来看一个简单的路由算法
#### 余数算法
比方说，字符串str对应的HashCode是50、服务器的数目是3，取余数得到2，str对应节点Node2，所以路由算法把str路由到Node2服务器上。由于HashCode随机性比较强，所以使用余数Hash路由算法就可以保证缓存数据在整个MemCache服务器集群中有比较均衡的分布。

如果不考虑服务器集群的伸缩性(集群中新加入节点或者某节点down掉)，那么余数Hash算法几乎可以满足绝大多数的缓存路由需求。

就假设MemCache服务器集群由3台变为4台，更改服务器列表，仍然使用余数Hash，50对4的余数是2，对应Node2，但是str原来是存在Node1上的，这就导致了缓存没有命中。如果这么说不够明白，那么不妨举个例子，原来有HashCode为0~19的20个数据，那么：

![原始hash](http://githubforericwang.qiniudn.com/hexo/eric/memcache/1.png)

现在我扩容到4台，加粗标红的表示命中：

![扩容后](http://githubforericwang.qiniudn.com/hexo/eric/memcache/2.png)

如果我扩容到20+的台数，只有前三个HashCode对应的Key是命中的，也就是15%。当然这只是个简单例子，现实情况肯定比这个复杂得多，不过足以说明，使用余数Hash的路由算法，在扩容的时候会造成大量的数据无法正确命中（其实不仅仅是无法命中，那些大量的无法命中的数据还在原缓存中在被移除前占据着内存）。这个结果显然是无法接受的，在网站业务中，大部分的业务数据度操作请求上事实上是通过缓存获取的，只有少量读操作会访问数据库，因此数据库的负载能力是以有缓存为前提而设计的。当大部分被缓存了的数据因为服务器扩容而不能正确读取时，这些数据访问的压力就落在了数据库的身上，这将大大超过数据库的负载能力，严重的可能会导致数据库宕机。

#### 一致性Hash算法
一致性Hash算法通过一个叫做一致性Hash环的数据结构实现Key到缓存服务器的Hash映射

![一致性hash](http://githubforericwang.qiniudn.com/hexo/eric/memcache/3.png)

具体算法过程为：先构造一个长度为2^32 的整数环（这个环被称为一致性Hash环），根据节点名称的Hash值（其分布为[0, 2^32 -1]）将缓存服务器节点放置在这个Hash环上，然后根据需要缓存的数据的Key值计算得到其Hash值（其分布也为[0, 2^32 -1]），然后在Hash环上顺时针查找距离这个Key值的Hash值最近的服务器节点，完成Key到服务器的映射查找。

就如同图上所示，三个Node点分别位于Hash环上的三个位置，然后Key值根据其HashCode，在Hash环上有一个固定位置，位置固定下之后，Key就会顺时针去寻找离它最近的一个Node，把数据存储在这个Node的MemCache服务器中。使用Hash环如果加了一个节点会怎么样，看一下：

![新增节点后](http://githubforericwang.qiniudn.com/hexo/eric/memcache/4.png)

加了一个Node4节点，只影响到了一个Key值的数据，本来这个Key值应该是在Node1服务器上的，现在要去Node4了。采用一致性Hash算法，的确也会影响到整个集群，但是影响的只是加粗的那一段而已，相比余数Hash算法影响了远超一半的影响率，这种影响要小得多。更重要的是，集群中缓存服务器节点越多，增加节点带来的影响越小，很好理解。换句话说，随着集群规模的增大，继续命中原有缓存数据的概率会越来越大，虽然仍然有小部分数据缓存在服务器中不能被读到，但是这个比例足够小，即使访问数据库，也不会对数据库造成致命的负载压力。

至于具体应用，这个长度为2^32 的一致性Hash环通常使用二叉查找树实现。

## 内存管理
前面说到memcache是内存缓存，不会持久化数据，而且也会受到机器位数的限制，32位最多只有2G，64位可以认为没有上限。

### 内存分配
下面说下内存分配，传统的内存管理方式是，使用完通过malloc分配的内存后通过free来回收内存，这种方式容易产生内存碎片并降低操作系统对内存的管理效率，所以memcache采用的是固定空间分配，MemCache的这种内存分配的方式称为allocator。

![内存分配方式](http://githubforericwang.qiniudn.com/hexo/eric/memcache/5.png)

这张图片里面涉及了slab_class、slab、page、chunk四个概念，它们之间的关系是：

＊ MemCache将内存空间分为一组slab

＊ 每个slab下又有若干个page，每个page默认是1M，如果一个slab占用100M内存的话，那么这个slab下应该有100个page，page一旦分配，就不会被回收

＊ 每个page里面包含一组chunk，chunk是真正存放数据的地方，同一个slab里面的chunk的大小是固定的

＊ 有相同大小chunk的slab被组织在一起，称为slab_class


slab的数量是有限的，几个、十几个或者几十个，这个和启动参数的配置相关。

MemCache中的item存放的地方是由item的大小决定的，item总是会被存放到与chunk大小最接近的一个slab中，比如slab[1]的chunk大小为80字节、slab[2]的chunk大小为100字节、slab[3]的chunk大小为128字节（相邻slab内的chunk基本以1.25为比例进行增长，MemCache启动时可以用-f指定这个比例），那么过来一个88字节的item，这个item将被放到2号slab中。放slab的时候，首先slab要申请内存，申请内存是以page为单位的，所以在放入第一个数据的时候，无论大小为多少，都会有1M大小的page被分配给该slab。申请到page后，slab会将这个page的内存按chunk的大小进行切分，这样就变成了一个chunk数组，最后从这个chunk数组中选择一个用于存储数据。

如果这个slab中没有chunk可以分配了怎么办，如果MemCache启动没有追加-M（禁止LRU，这种情况下内存不够会报Out Of Memory错误），那么MemCache会把这个slab中最近最少使用的chunk中的数据清理掉，然后放上最新的数据。
这种内存分配方式的特点避免了管理内存碎片问题，同时也带来了内存浪费的问题，如88字节的item分配在128字节（紧接着大的用）的chunk中，就损失了40字节。


### 缓存策略
Memcached的缓存策略是LRU（最近最少使用）加上延迟删除策略

延迟删除是指memcache并不会监视和清理过期数据，而是在客户端get时检查。比如设置某个key存活30s，到了30s后，memcache并不会主动清理它，而是有get请求发现它后才会设置为过期方便以后优先利用，还有一种情况就是lru。
删除操作只会将chunk置为删除状态，这样下次申请的时候可以优先利用。
flush操作只会使所有的item失效。
这种删除方式，可以提高memcache的效率，因为不必每时每刻检查过期item,从而提高CPU工作效率。
LRU只是针对slab内的，并不是全局的，因为不同的slab的chunk不同。

## 应用
### 使用telnet操作memcache
* set 和 get

```
➜  ~ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
set foo 0 0 6
123456
STORED
get foo
VALUE foo 0 6
123456
END
```

* 查看memcache状态

```
➜  ~ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
stats
STAT pid 1
STAT uptime 275
STAT time 1463557642
STAT version 1.4.25
STAT libevent 2.0.21-stable
STAT pointer_size 64
...
STAT evictions 0
STAT reclaimed 0
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
END
```
* 查看各个slab使用状态

```
➜  ~ telnet localhost 11211
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
stats slabs
STAT 1:chunk_size 96
STAT 1:chunks_per_page 10922
STAT 1:total_pages 1
STAT 1:total_chunks 10922
STAT 1:used_chunks 1
STAT 1:free_chunks 10921
STAT 1:free_chunks_end 0
STAT 1:mem_requested 74
STAT 1:get_hits 1
STAT 1:cmd_set 1
STAT 1:delete_hits 0
STAT 1:incr_hits 0
STAT 1:decr_hits 0
STAT 1:cas_hits 0
STAT 1:cas_badval 0
STAT 1:touch_hits 0
STAT active_slabs 1
STAT total_malloced 1048512
END
```

* 简易监控
telnet是一个交互的工具，并不适合在监控脚本中使用，可以用netcat

```
➜  ~ (echo stats;sleep 1)|nc 127.0.0.1 11211
STAT pid 1
STAT uptime 1995
STAT time 1463559362
STAT version 1.4.25
STAT libevent 2.0.21-stable
STAT pointer_size 64
STAT rusage_user 0.048000
STAT rusage_system 0.048000
....
```

### php中的一些应用
memcache常常作为数据缓存，来缓解mysql的压力。
当item不存在时，memcache会返回false，可以根据memcache的返回结果。
注意判断只能使用 === 
当设置一个key为false时，memcache会返回一个空字符串。

* 一个简单的缓存

```
function getFoo($id,$cleanCache){
	$memcache =  CacheFactory:getInstance()->getMemcache("");
    $prefix = "foo_";
	$key = $prefix.$id;
	$value = $memcache->get($key);
	if($value === false || $cleanCache){
		$value = DaoFactory::get();
		...
		memcache->set($key,$value,0,self::$expireTime);
	}
	return $value;
}	
```

* 分布式会话管理

登陆信息等。

* memcache cas
memcache的每一个命令都是原子操作的,比如get，set，但是针对同一个key的多次get，set请求并不是原子的，是串行化的。

```
<?php
$m = new Memcached();
$m->addServer('localhost', 11211);

do {
    /* 获取ip列表以及它的标记 */
    $ips = $m->get('ip_block', null, $cas);
    /* 如果列表不存在， 创建并进行一个原子添加（如果其他客户端已经添加， 这里就返回false）*/
    if ($m->getResultCode() == Memcached::RES_NOTFOUND) {
        $ips = array($_SERVER['REMOTE_ADDR']);
        $m->add('ip_block', $ips);
    /* 其他情况下，添加ip到列表中， 并以cas方式去存储， 这样当其他客户端修改过， 则返回false */
    } else { 
        $ips[] = $_SERVER['REMOTE_ADDR'];
        $m->cas($cas, 'ip_block', $ips);
    }   
} while ($m->getResultCode() != Memcached::RES_SUCCESS);

?>
```
有兴趣的可以看一下这篇文章:[memcached 原子性操作 CAS模式](http://blog.csdn.net/jiangbo_hit/article/details/6211704)

* socket 操作memcache
一般都不会这么用，除非没有安装memcache扩展

```
//connect
$socket = fsockopen('10.207.26.234',11211);

//set
fwrite($socket,"set a 0 0 1\r\n1\r\n");
$response = fgets($socket);
echo "<pre>";print_r($response);
//get
fwrite($socket,"get a\r\n");
$response = fread($socket,1024);
echo "<pre>";print_r($response);

```



### 应用场景
适用的场景

* 对于mysql读>>写的场景，memcached可以显著地提高运行效率
* 一些很小但是频繁访问的文件
* session数据
* 计算量很大，可以用memcache缓存结果

不适用的场景

* 需要获取所有key
* key的长度超过250字符
* item过大
* 实时性要求高的场景

## 其他
* 安全
* 其他缓存系统，redis，mongodb，hbase等等
* 缓存一致性问题
* 缓存雪崩
* 缓存预热
* 缓存无底洞
* (twemproxy)nutcraker

## 参考
*  [MemCache超详细解读](http://www.cnblogs.com/xrq730/p/4948707.html?utm_source=tuicool&utm_medium=referral)



