title: dubbo 服务治理
tags:
  - java
  - rpc
category: java
date: 2015-08-23 20:46:00

---
之前打算使用consul作为服务注册发现,thrift来做远程服务调用.很可惜，一些原因没有使用.
使用dubbo来作为服务治理
dubbo的[官方文档](http://dubbo.io)也比较详细，使用过程中也踩了不少坑
缺点是只能使用java，不能满足其他语言之间的互相通信。
而thrift 是一个语言框架，在spring中使用swift可以自动生成idl文件，通过idl文件生成其他语言，这点很强大,关于thrit，虽然以后不一定会接触了，[这篇](http://calvin1978.blogcn.com/articles/apache-thrift.html)写的还是不错的

最近很累，不想写太多，就转个不错的吧，是一个系列文章
[Dubbo框架应用之（一）--服务体系](http://blog.csdn.net/lishehe/article/details/46652497)
[Dubbo框架应用之（二）--服务治理](http://blog.csdn.net/lishehe/article/details/46652743)
[ Dubbo框架应用之（三）--Zookeeper注册中心、管理控制台的安装及讲解](http://blog.csdn.net/lishehe/article/details/46653915)
[Dubbo框架应用之（四）--Dubbo基于Zookeeper实现分布式实例](http://blog.csdn.net/lishehe/article/details/46653793)