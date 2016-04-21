title: jdk linux环境
tags:
  - linux
category: linux
date: 2015-08-10 16:07:00

---
## 下载
虽然Ubuntu提供了openjdk的使用,但是有时候apt-get的安装软件的配置信息并不熟悉,所以个人还是习惯自主安装
jdk7 的[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
必须accept才能下载

---
## 安装
我是在远程机器上安装的,所以还使用了scp等工具，这里就忽略了
```
$ sudo mkdir /usr/local/java
$ sudo tar zxvf jdk-7u21-linux-i586.tar.gz -C /usr/local/java
$ cd /usr/local/java
$ sudo mv jdk1.7.0_67 jdk
```
-C表示解压到指定文件夹,zxvf不说了
这样就在 /usr/local/java目录下生成了jdk目录

---
## 添加环境变量
```
$ sudo vi /etc/bash.bashrc 
```
在最后一行添加如下代码
```
export JAVA_HOME=/usr/local/java/jdk
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH  
```
使修改生效
```
$ source /etc/bash.bashrc 
```
---
## 测试
最后 在终端中输入java -version
```
$ java -version
java version "1.7.0_67"
Java(TM) SE Runtime Environment (build 1.7.0_67-b01)
Java HotSpot(TM) 64-Bit Server VM (build 24.65-b04, mixed mode)
```
安装成功