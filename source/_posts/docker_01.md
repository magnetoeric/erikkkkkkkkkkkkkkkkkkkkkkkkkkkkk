title: 初识docker 
tags:
  - docker
category: docker
date: 2015-07-20 10:35:00

---
## 什么是docker
docker是一个开源项目,其目的是实现轻量级的操作系统虚拟化解决方案.
Docker 的基础是linux的容器(LXC)等技术

---
## docker和vagrant的区别
stackoverflow上的一个[问题](http://stackoverflow.com/questions/16647069/should-i-use-vagrant-or-docker-io-for-creating-an-isolated-environment)，二楼是vagrant的作者，三楼是docker的作者

---
## Docker的三个概念
* 镜像(Image)
* 容器(Container)
* 仓库(Repository)
理解这三个概念,就理解了Docker的整个生命周期

### 镜像
Docker镜像就是一个只读的模板。例如一个镜像可以包含一个完整的Ubuntu镜像,里面仅安装了某些你需要的应用程序,比如Apache+Mysql+php等等
镜像可以用来创建Docker容器
Docker提供了一个简单的方式来创建镜像或是更新现有镜像DockerHub(有点像github),你可以直接从官方或是其他人那里获得镜像直接使用
### Docker 容器
Docker 利用容器来运行应用。
容器是镜像创建的运行实例,每个容器可以是隔离的,保证安全的平台。
镜像是只读的,容器在启动的时候创建一层可写层作为最上层。
### Docker仓库
仓库是集中存放镜像文件的场所,有公有仓库和私有仓库
公有仓库就是我们前面说的DockerHub，用户可以在本地网络中创建一个私有仓库
用户可以创建自己的镜像并push到共有或私有仓库,方便下次或另外机器上使用
和github类似,不多说了

---
## 安装
Ubuntu系统安装(我的是13.10系统 这种安装方式不是最新的Docker)
```
$ sudo apt-get update
$ sudo apt-get install docker.io
$ sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker
$ sudo sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker.io
```
---
## 运行一个Docker实例
首先要注册一个Docker账户[这里](https://hub.docker.com/account/signup/)，邮箱确认后就可以登录了
```
$ sudo docker login
```
登录后,可以输入`$ docker run`来查看可用指令
接下来 试一试`$ sudo docker run ubuntu:14.04 /bin/echo 'Hello world'`
`sudo docker run ubuntu:14.04`告诉 Docker加载 ubuntu 14.04。
Docker在运行时会判断当前运行的镜像是否加载到Docker主机上,如果没有,会到Docker下载指定的镜像
	```
Hello world
```

以上 就安装并运行了一个简单的Docker实例
