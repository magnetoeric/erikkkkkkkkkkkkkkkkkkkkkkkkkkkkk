# 利用shell脚本对大文件进行分割 
日常应用产生的日志是需要做文件分割的,无论是按文件大小还是按时间

##　利用shell脚本
```
#!/bin/bash  
   
  lines=`wc   -l   xxx.log|   awk   '{print   $1}'`  
  i=1  
  file_suffix=1  
  while   [   $i   -lt   $lines   ]  
  do  
                  next=`expr   $i   +   999`  
                  sed   -n   "${i},   ${next}p"   xxx.log >   file_$file_suffix.log    
                  n1=`expr   $n2   +   1`  
                  file_suffix=`expr   $file_suffix   +   1`  
  done   
```

实现了每1000行来分割文件

## 使用工具split
split 参数：
-b  : 后面可接欲分割成的档案大小，可加单位，例如 b, k, m 等；
-l  : 以行数来进行分割；
-d  : 默认是以字符为后缀的,使用-d 或者--numeric-suffixes  可以改变后缀
查看参数更多可以　`man split`

 
1.按每个文件1000行来分割除
split -l 1000 access.log split 
会切割成splitaa splitab splitac...
 
2.按照每个文件100K来分割
split -l 1000 access.log split 
会切割成httpaa，httpab，httpac ........

3.流切割
对于流split一样可以做到
简单点举例
cat access.log | split -l 1000 split
这样有些守护进程就可以这样写
php xxx.php | split -l 10000 xxx.log.

