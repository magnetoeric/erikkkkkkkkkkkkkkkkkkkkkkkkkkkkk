title: awk简单使用
tags:
  - linux
category: linux
date: 2015-08-10 16:05:00

---
## 简介
awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理.

----
## 使用方法
基本语法如下，[使用手册](http://www.chinaunix.net/old_jh/7/16985.html)

```
awk '{pattern + action}' {filenames}
```
尽管操作可能会很复杂，但语法总是这样，其中 pattern 表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。

---
## 工作流程
awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"。如果想使用其他分隔符，可以使用-F指定域分隔符为':'。

---
## awk内置变量
````
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数
NR                 已读的记录数
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
````

----
## 实例
*   显示最近登录的5个帐号

```
$last -n 5 |awk '{print $1}'
root     pts/0        101.229.11.88    Mon Dec  1 22:47   still logged in   

wtmp begins Mon Dec  1 22:47:35 2014
```
*   显示/etc/passwd的账户和账户对应的shell,而账户与shell之间以tab键分割

```
$cat /etc/passwd |awk  -F ':'  '{print $1"\t"$7}'
root	/bin/bash
daemon	/usr/sbin/nologin
bin	/usr/sbin/nologin
sys	/usr/sbin/nologin
....
```

*   搜索/etc/passwd有root关键字的所有行

```
$awk '/root/' /etc/passwd
$sed -n '/root/p' /etc/passwd #sed方式
```


