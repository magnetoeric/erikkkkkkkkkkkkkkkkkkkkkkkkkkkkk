title: rabbitmq 环境搭建
date: 2016-04-20 14:44:36
tags:
---
## erlang 安装
关于erlang 的下载，可以到官网找最新版本
```
wget http://erlang.org/download/otp_src_18.3.tar.gz
tar zxvf otp_src_18.3.tar.gz
cd otp_src_18.3
./configure
sudo make
sudo make install 
```

erlang编译会依赖一些环境，这里就不一一列举了
缺省情况下需要libncurses5-dev  libssl-dev 以及java环境
因为本人用的zsh,在编译的时候提示/bin/sh javac not found，原因大概是javac并没有在/bin/sh的path的环境变量中，暴力了一点,直接在/usr/local/bin下做了javac,jar的软连接,解决了

## rabbitmq
下载地址:http://www.rabbitmq.com/download.html
选择binary安装包
```
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-generic-unix-3.6.1.tar.xz
xz -d rabbitmq-server-generic-unix-3.6.1.tar.xz
tar xvf rabbitmq-server-generic-unix-3.6.1.tar
mv rabbitmq_server-3.6.1 /usr
mv rabbitmq_server-3.6.1 /usr/local/rabbitmq
```
下载的是二进制包，所以可以直接使用
启动rabbitmq 
```
/usr/local/rabbitmq/sbin/rabbitmq-server
```	