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
```

```




## 参考
	[自动化运维工具Ansible详细部署](http://sofar.blog.51cto.com/353572/1579894/)
