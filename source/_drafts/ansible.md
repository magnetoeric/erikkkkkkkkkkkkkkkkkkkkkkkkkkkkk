---
title: ansible
tags:
---

# ansible使用
ansible可以作为自动化运维工具，一直以来都是知道有这东西，却没有尝试过，现在把吹过的牛逼补回来

1、简介

ansible是自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。主要包括：

* 连接插件connection plugins：负责和被监控端实现通信；

* host inventory：指定操作的主机，是一个配置文件里面定义监控的主机；

* 各种模块核心模块、command模块、自定义模块；

* 借助于插件完成记录日志邮件等功能；

* playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务。

## 安装
略过了，使用pip，安装很简单

## 总体架构
![](http://githubforericwang.qiniudn.com/hexo/eric/wKiom1Rsxz3ToUCAAAGROYAM3EI989.jpg)

## 特性

* no agents：不需要在被管控主机上安装任何客户端；

* no server：无服务器端，使用时直接运行命令即可；

* modules in any languages：基于模块工作，可使用任意语言开发模块；

* yaml，not code：使用yaml语言定制剧本playbook；

* ssh by default：基于SSH工作；

* strong multi-tier solution：可实现多级指挥。


## 优点
* 轻量级，无需在客户端安装agent，更新时，只需在操作机上进行一次更新即可；
* 批量任务执行可以写成脚本，而且不用分发到远程就可以执行；
* 使用python编写，维护更简单，ruby语法过于复杂；
* 支持sudo。

## 任务执行流程
![](http://githubforericwang.qiniudn.com/hexo/eric/wKiom1Rsx2uQYJZ5AAJplY08vOQ976.jpg)

## 使用
### 搭建多个可操作的host
没有机器，暂时用docker替代了
一个简单的sshd的Dockerfile
```
FROM ubuntu:14.04
MAINTAINER ericwang 123048591@qq.com
RUN apt-get update && apt-get install -y openssh-server \
   && sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config  \
   && useradd ansible \
   && echo ansible:ansible |chpasswd \
   && echo "admin   ALL=(ALL)       ALL" >> /etc/sudoers  \
   && mkdir /var/run/sshd
RUN ssh-keygen  -t rsa -f /root/.ssh/id_rsa -P ""
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]

```
build脚本
```
#!/bin/bash
docker build  --tag sshd-server ./

```

run脚本

```
#!/bin/bash
set -x
container_id=$(docker run -itd -P sshd-server)
cat ~/.ssh/id_rsa.pub | docker exec --user=root -i $container_id  sh -c 'cat >> /root/.ssh/authorized_keys'
docker exec $container_id chmod 600 /root/.ssh/authorized_keys

```
练习的时候才发现docker exec使用流的时候有些问题，google看到[这篇文章](https://forums.docker.com/t/docker-exec-api-using-stdin-to-upload-a-file/748)，直接使用exec command >> file这种形式是不行的，会把流输出到宿主系统，完全就是错误的。期待那位linux　大神继续更新shell相关的博文，更深入地理解linux

./build.sh 创建image,然后,./run.sh　这就创建了一个容器,也就是用来测试的ansible host。多次run后，就有了多个host机器了

```
➜  ~ docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
a57b8c69e513        sshd-server         "/usr/sbin/sshd -D"      35 seconds ago      Up 34 seconds       0.0.0.0:32784->22/tcp      serene_saha
```
每一次run后都会产生一个容器,将端口22映射到主机的一个端口上，可以通过ssh登陆到docker容器中
如
```
➜  ~ ssh root@127.0.0.1 -p 32784
The authenticity of host '[127.0.0.1]:32784 ([127.0.0.1]:32784)' can't be established.
ECDSA key fingerprint is 6a:d1:27:e4:89:93:98:96:8f:02:f7:bf:6e:d7:eb:7b.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:32784' (ECDSA) to the list of known hosts.
root@a57b8c69e513:~# 
```
第一次登陆需要add到know hosts，以后就是免密码登陆了

启动了２个docker容器后,接下来是配置ansible了
默认hosts是在/etc/ansible/hosts,添加一组远程主机地址
```
➜  ~ cat /etc/ansible/hosts 
[docker]
127.0.0.1:32784
```







## 参考
[自动化运维工具Ansible详细部署](http://sofar.blog.51cto.com/353572/1579894/)
[ansible 官方文档](http://docs.ansible.com)
