title: redis分布式锁
tags:
  - redis
category: nosql
date: 2015-12-02 09:19:17

---
看了[并发编程网的这篇翻译](http://ifeve.com/redis-lock/)。
看了标题，首先想到的是如何实现一个单机锁，因为redis中大多数操作都是原子的，所以想法是：
    

 - 在redis中存在一个key，保证客户端在读到这个key的时候，阻塞
 - 如果一个客户端在读不到的这个key的时候，说明没有客户端持有锁，可以将其上锁
 - 获取到锁的客户端在执行逻辑完后，释放锁

redis中的命令刚好有一个这样的命令  setnx。
继续看下面的翻译，发现自己的想法过于简单，任何设计都应该遵循design for failure 的原则

 - 如果一个客户端在获取到锁后，因为某些原因，down掉了，所有的客户端就永远不会持有锁了
 - 针对上面的情况，可以为这个key设置一个存活时间（但是setnx并不支持存在时间，如果使用expire，不能保证原子性，看了文档，在2.6后续版本中，set已经可以替代setnx了[set的文档]http://redis.io/commands/set）
 - 针对上一条的情况，由于某些因素导致某客户端A获取锁后执行的时间过长，一致于超过了锁的缓存时间，这时又有其他客户端B获取到锁，A执行完成之后，释放锁，这时其他客户端就会获取锁。所以需要为每一个锁设置唯一的value，保证只有持有锁的客户端才能删除它

在其他的blog里也看到了类似的想法[谈谈Redis的SETNX](http://huoding.com/2015/09/14/463)

redis作者也给出了[代码](https://github.com/ronnylt/redlock-php)(php版本)，这里不贴了，分布式版本中，考虑的问题就更多了。[原文](https://github.com/antirez/redis-doc/blob/master/topics/distlock.md)中，针对分布式中的情况作了很多解释，翻译也有了，就不贴了，不过翻译有的地方是错的，以后还是看原文吧，英语看的慢点，多看几遍，不至于理解错
