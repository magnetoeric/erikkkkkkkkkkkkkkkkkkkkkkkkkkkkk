title: activemq 通信方式总结
category: other
date: 2017-01-10 20:59:00

---
## 简介
activemq是JMS消息通信规范的一个实现。总的来说，消息规范里面定义最常见的几种消息通信模式主要有发布-订阅、点对点这两种。

## 基础流程

按照JMS的规范，主要的几个步骤如下：
* 获得JMS connection factory. 通过我们提供特定环境的连接信息来构造factory。
* 利用factory构造JMS connection
* 启动connection
* 通过connection创建JMS session.
* 指定JMS destination.
* 创建JMS producer或者创建JMS message并提供destination.
* 创建JMS consumer或注册JMS message listener.
* 发送和接收JMS message.
* 关闭所有JMS资源，包括connection, session, producer, consumer等。

发布订阅模式
