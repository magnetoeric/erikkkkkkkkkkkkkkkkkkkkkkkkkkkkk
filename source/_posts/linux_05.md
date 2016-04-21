title: linux source
tags:
  - linux
category: linux
date: 2015-08-10 17:54:00

---
常常在修改环境变量后需要source file.今天看看这东西到底是什么
## shell与export命令
先说说shell和export
用户登录到Linux系统后，系统将启动一个用户shell。在这个shell中，可以使用shell命令或声明变量，也可以创建并运行shell脚本程序。运行shell脚本程序时，系统将创建一个子shell。
此时，系统中将有两个shell，一个是登录时系统启动的shell，另一个是系统为运行脚本程序创建的shell。当一个脚本程序运行完毕，脚本shell将终止，返回到执行该脚本之前的shell。
从这种意义上来说，用户可以有许多 shell，每个shell都是由某个shell（称为父shell）派生的。在子shell中定义的变量只在该子shell内有效。如果在一个shell脚本程序中定义了一个变量，
当该脚本程序运行时，这个定义的变量只是该脚本程序内的一个局部变量，其他的shell不能引用它，要使某个变量的值可以在其他shell中被改变，可以使用export命令对已定义的变量进行输出。
export命令将使系统在创建每一个新的shell时，定义这个变量的一个拷贝。这个过程称之为变量输出。[引自](http://www.cnblogs.com/zhangze/articles/1832542.html)

---
## 区别
source filename 与 sh filename 及./filename执行脚本的区别在那里呢？
* 当shell脚本具有可执行权限时，用sh filename与./filename执行脚本是没有区别得。./filename是因为当前目录没有在PATH中，所有"."是用来表示当前目录的。
* sh filename 重新建立一个子shell，在子shell中执行脚本里面的语句，该子shell继承父shell的环境变量，但子shell新建的、改变的变量不会被带回父shell，子shell结束时.变量消失。不会返回给父shell除非使用export。
* source filename：这个命令其实只是简单地读取脚本里面的语句依次在当前shell里面执行，没有建立新的子shell。那么脚本里面所有新建、改变变量的语句都会保存在当前shell里面。

---
## 测试
新建test.sh
```
#!/bin/sh
A=1;
echo $A;
```
* `sh test.sh`  终端输出1，在终端中执行`echo $A`,变量不存在父shell中 
* ```
chmod +x test.sh //赋予脚本可执行权限
./test.sh
```
终端输出1，在终端中执行`echo $A`,变量不存在父shell中 
* `source test.sh`
终端输出1，在终端中执行`echo $A`,终端输出1

原因很简单,source是在当前shell中执行的,而执行脚本是在子shell中执行的。所以在修改某些环境变量后,为了能让脚本读取到当前环境的变量,必须先source 变量文件，将环境变量立即导入进来,
然后脚本在执行时会在一个新建的子shell中执行,由于子shell继承父shell的环境变量,脚本才能正常运行

