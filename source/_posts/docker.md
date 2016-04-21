title: docker 笔记
tags:
  - docker
category: docker
date: 2016-02-07 14:33:00

---
最近找了本docker的书[Using Docker](http://www.salttiger.com/using-docker/) 
读了大半部分了，整理了一下一些很有用的Docker相关笔记，顺便把以前做的一些问题修改了一下，整套脚本构建放到了[github](https://github.com/magnetoeric/docker-laravel)上。

 - docker rm $(docker ps -aq) 
可以清除所有停止的docker 容器，运行中的不会被清理，同理，docker rmi $(docker images -aq)也可以清理，但是请谨慎使用，因为会把很多有用的‘基础’容器删除，构建image时还要重新下载。所以这里可以使用--filter 和awk grep等命令协同处理（因为有时候docker产生的无用image太多了）
 - 尽量不要指定HOST_DIR 挂载，因为这会影响到宿主机的文件
如果开发dev环境，可以指定宿主机挂载，注意container的运行权限，尽量不要对挂载目录有写的权限，运行时会产生一些缓存文件，这些缓存目录可以设置成777 并在git config里忽略文件的权限
 - CTRL P +CTRL Q 
这个命令是attach到任何容器后不想退出容器，而是进行detach
 - 对于简单的复制，可以使用copy 替代 add，add可以对文件进行解压缩，实际上是对流进行操作
 - 不要使用root运行 因为有时候你可能会使用挂载，而root有权限对挂载目录进行写操作，从而造成一些不必要的问题
 - .DOCKERIGNORE 类似gitignore，你可以添加一个 .dockerignore 文件到你的 `Dockerfile` ， Docker 将会在发送构建上下文到守护进程时忽略在 .dockerignore 中指定的文件和目录
 - 数据尽量使用单独的image，不要混合使用
 - docker save 和 export的区别是save是多layer  export只有一个layer (个人使用较少,export 还会丢失一些环境变量)
 - RUN 使用&&减少layer的数量，这样在同一层使用某文件后并删除可以控制image的大小。这也是我的构建脚本从900MB减少的500MB的原因，而且可以继续减少（去除apt安装的一些编译头文件）
 - compose 的yml可以使用extends 子yml里的配置会覆盖父yml的配置，links 和 volumes-from 不会被继承(docker-compose 个人不是很喜欢，个人比较喜欢shell，方便可控)
 - CMD和ENTRYPOINT的区别
CMD和ENTRYPOINT是在运行container 时会运行的指令, 都只能写一条, 如果写了多条, 则最后一条生效.
CMD在运行时会被command覆盖, ENTRYPOINT不会被运行时的command覆盖, 但是也可以指定.[Docker RUN CMD 和ENTRYPOINT](http://blog.163.com/digoal@126/blog/static/163877040201410411715832/) 我对ENTRYPOINT的使用也不是很多
 - 不要把key放到docker里
安全原因,可以把它写到shell里，运行时通过shell添加；也可以通过配置中心获取，比如consul等
 - docker 日志输出
[logspout](https://github.com/gliderlabs/logspout) 可以将docker 产生的log 输出到任何地方，可以使用elk将这些收集整理 最近闲暇时间也学习了一下elk，也搭建了一套本地elk环境，以后该考虑一下应用到生产环境
 - 监控docker cpu等信息
```
docker stats $(docker inspect -f {{.Name}} $(docker ps -q)) 
``` 

继续说说docker laravel环境遇到的一些问题，因为需要将ecshop的代码迁移到laravel里，也就意味着 php版本也要做相应的升级，问题不少，所以搭建了一套laravel的也搭建了一套ecshop的，但是ecshop的nginx涉及到私有问题，就不传出来了。思想都是一样的。最后说说一个折腾了很久的问题吧，搭建docker的环境，主要是为了分离各服务，也就是所谓的微服务microservice，但是使用时，nginx对php-fpm进行link和volumn，发现端口并不能被转发到PHP－fpm的socket端口，nginx一直报502.由于nginx使用的是官方提供的版本，系统精简到ps，netstat，telnet等一些常用的命令都没有，加大了调试难度，在调试过程中个人也从中学到了很多linux系统的一些知识。最后在php-fpm容器中使用netstat发现Local Address是127.0.0.1:9000,也就是说只会监听本地的端口，遂查看php-fpm的config，将127.0.0.1:9000替换为9000,使其可以被外网访问到，当然容器中才可以这样使用，因为容器本身并没有把端口暴露给外部，而是暴露给使用了link该容器的容器。