title: rabbitmq笔记
tags:
  - rabbitmq
category: other
date: 2017-03-03 22:01:00

---
### rabbitmq cluster
queue 的信息只在一台node上存储，其他节点不会存储，所有节点只知道queue的信息在这个node上，当这个node 挂掉之后，message将进入黑洞。
单节点的rabbitmq是disk模式的，集群模式下，可以设置至少一个disk节点，其他节点设置为RAM模式，这样可以提高路由的性能，
如果只有一个disk节点并且disk节点挂掉，创建queue，exchange，bings，添加user，修改permission，添加删除节点将会失败，直到disk节点恢复

### rabbitmq reply_to
rabbitmq 屏蔽了生产者和消费者，生产者不知道消费者是谁，同样消费者也不知道生产者是谁
但是如果生产者如果需要一个回复该如何解决？
在header中设置reply_to

生产者声明临时匿名队列为exclusive，同时设置reply_to为接受队列名称。当然有时候计算比较大，消费者还需要确定生产者是否还在监听，[direct-reply-to](http://www.rabbitmq.com/direct-reply-to.html)中给出了一些建议

### rabbitmq 三种分布式方案
规约：
exchange 交换机
queue      队列
broker      中间服务器/服务器
cluster     集群
node       节点
route       路由

RabbitMQ可以通过三种方法部署分布式系统：集群、联盟（federation）和shovel。

集群通过连接多个机器组成单个逻辑中间服务器。机器之间通信要借助于Erlang的消息传输，要求集群中所有节点必须有相同的Erlang cookie；节点之间的网络必须是可靠的，且运行相同版本的RabbitMQ和Erlang。
虚拟主机、交换机、用户信息和权限会自动镜像到集群中各个节点。队列可能位于单个节点或镜像到多个节点。连接到任意节点的客户端能够看到集群中所有队列，即使该队列不位于连接节点上。
通常可以使用集群来提高可靠性和吞吐量，前提是在分布同一个区域内的机器，不支持网络分段。

联盟允许单台服务器上的交换机或队列接收发布到另一台服务器上交换机或队列的消息，可以是单独机器或集群。服务器之间通过AMQP协议通信，因此两个联盟交换机或联盟队列要求设置相应的用户权限。
联盟交换机之间由单向点对点链接关联，默认消息只会由联盟链接转发一次，但允许有更复杂的路由拓扑来提高转发次数。消息也可以不进行转发；如果消息到达联盟交换机之后不会路由到队列，那么它再也不会被转发。
联盟队列类似于单向点对点连接，消息会在联盟队列之间转发任意次，直到被消费者接受。
通常使用联盟来连接internet上的中间服务器，用作订阅分发消息或工作队列。

shovel连接方式与联盟的连接方式类似，但它工作在更低层次。shovel接受队列上的消息，转发到另一台服务器上的交换机。
shovel和联盟类似，但它比联盟提供更多控制。

联盟/shovel vs 集群：
1）前者中间服务器逻辑分离，后者组成一个逻辑中间服务器；
2）前者可以运行不同版本RabbitMQ和Erlang，后者要求RabbitMQ和Erlang的版本保持一致；
3）前者可以分布在WAN上，采用AMQP协议通信，要求设置权限，后者必须分布在LAN上，结合Erlang内部节点通信，要求有相同的Erlang cookie；
4）前者的拓扑结构可自行设计，链接可以单向或双向，后者要求节点之间必须保持双向链接；
5）前者遵循CAP理论的可用性和分区容错性，后者遵循CAP理论的一致性和可用性（可选择一致性和分布容错性）；
6）前者服务器中的交换机可以选择联盟或本地，后者孤注一掷；
7）前者客户端只能看到所连接服务器上的队列，后者客户端可以看到所有节点的队列。


Federation / Shovel 属于AP 系统(高可用性)   cluster属于CP系统(强一致性)

参考：https://www.rabbitmq.com/distributed.html


### rabbitmq 一些参数说明
为队列设置唯一消费者,设置exclusive参数,如果想设置临时队列，可以加入auto_delete参数
声明队列时，如果声明参数相同，rabbit不会做任何动作，如果参数不同，则会报错
生产者投递消息到exchange中后，如果没有被路由将会被抛弃(black holed)
消费者无法在订阅一个queue的时在同一个channel中声明其他queue
exchange type :fanout,direct,topic,header(性能问题，不常用)
重启后queue和exchange都会消失，原因是durable默认时false
message需要persitant参数，queue和exchange durable参数设置为true才能保证重启后不丢失
可以通过basic.ack做好事务,auto_ack会在消息到达后自动删除，所以消费者无法做事务

### 其他
amqp connection 是建立在tcp之上的,由于tcp建立握手需要耗费过多的资源，所以引入了channel的概念，一个amqp connection可以建立无数个channels。
从queue中消费有两种方式，一种是basic.consume 一种是basic.get,consume会一直消费消息直到停止订阅，而get一次只消费一个消息，直到再次调用get才会消费下一个消息，
不要再循环中使用get，因为它每次都是订阅队列->消费一个消息->取消订阅
一个队列有多个消费者时，采用的是round robin方式分配消息