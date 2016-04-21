title: linux TIME_WAIT
tags:
  - linux
  - http
category: linux
date: 2016-01-22 00:30:00

---
看到这篇[记一次TIME_WAIT网络故障](http://huoding.com/2012/01/19/142),想起之前处理线上故障。
当时的情况是公司用的分布式job服务器load过高，杀掉所有的脚本，load就降下去了，但是再启动这些load又上去，重启服务，发现竟然无法启动了，原因是端口被占用了
当时也不知道如何去分析，使用
```
netstat -ant | awk '
    {++s[$NF]} END {for(k in s) print k,s[k]}
```
查看，再查看sysctl net.ipv4.ip_local_port_range="min max"
TIMEWAIT的数量 大约等于max-min，
原因很明显，有脚本大量地占用端口，导致端口没有释放，重启服务端口被占用的原因是因为分布式脚本监控服务有一个监听端口，需要，同时也要发送心跳包到服务器上，而其重启时，由于是高端口，导致被占用，无法释放，服务也无法启动了。
但是并不知道是哪个脚本产生的问题，当时也不知道strace这类命令，用tcpdump去抓包，发现请求solr的http请求特别多，于是去查看代码，原来是shell一个常驻脚本，调用php脚本去执行查询solr服务，做一些简单逻辑后就退出，这样一直循环，因为启动和结束速度非常快，所以load也会飙升上去了。
解决方法，当时也只是在代码层面做了些sleep。系统方面因为不是很熟悉，也没有做优化。如今看了这篇，收货颇丰

