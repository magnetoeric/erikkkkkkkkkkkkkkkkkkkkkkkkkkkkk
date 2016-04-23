title: linux ps 的几个tips
tags: []
categories:
  - linux
date: 2016-04-23 23:47:00
---
1.有时候查看某些运行中的进程时常常这样(比如nginx)
```
➜  ~ ps aux |grep nginx
root            13617   0.0  0.0  2463900    448   ??  Ss   11:48下午   0:00.00 nginx: master process nginx
eric            13624   0.0  0.0  2434840    744 s001  R+   11:48下午   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn nginx
eric            13618   0.0  0.0  2464120   1000   ??  S    11:48下午   0:00.00 nginx: worker process
```
问题就是，我想对这些pid做些操作，但是会多出来一个ps进程产生的一行，一直以来也不知道如何去掉这行，反正也不碍事。前些天看到一篇微博里给出了方法
```
➜  ~ ps aux |grep \[n]ginx
eric            13618   0.0  0.0  2464120   1000   ??  S    11:48下午   0:00.00 nginx: worker process
root            13617   0.0  0.0  2463900    448   ??  Ss   11:48下午   0:00.00 nginx: master process nginx
➜  ~
```
2.有时候需要展示进程之间的父子关系，虽然有父进程的pid，但是并不直观，可以用`ps aux --forest`
3.批量杀掉进程,可以借助xargs
```
ps aux |grep \[n]ginx |awk '{print $2}' |xargs sudo kill -9
```