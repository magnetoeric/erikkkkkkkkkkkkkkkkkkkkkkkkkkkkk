title: vim sudo权限更新文件
tags:
  - linux
category: linux
date: 2015-11-13 19:11:00

---
工作时候常需要修改某些root权限的文件（如/etc/hosts），然而经常忘记打sudo命令，直接更改文件后，保存修改时才发现，白改了
退出后执行

```
sudo !!
```
可以用root权限执行上一次执行的命令，然后重新修改文件

最近发现了一个比较好的方法

```
w !sudo tee %
```
w的意思是把当前文件的所有行转到标准输入  ！表示要执行cmd命令，tee是一个小工具,读取标准输入的数据，并将其内容输出成文件。％表示当前文件，所以很好理解了，把当前文件传入到标准输入流，并使用root权限读取标准输入的数据输出到当前文件。

当然并没有完，tee也会将缓存内容输出到标准输出，所以我们要忽略它，命令进化为

```
w !sudo tee > /dev/null %
```

但是这个在vim里使用  要敲打着么多  费劲不？
当然费劲，而且每次都是敲同样的  
所以把它放到vimrc里
```
cmap w!! w !sudo tee > /dev/null %
```

这样再有什么需要root权限的文件，确定需要修改的时候 就可以 使用:w!!（很像 sudo !!）就完成了  省了很多时间

参考
[How does the vim “write with sudo” trick work?](http://stackoverflow.com/questions/2600783/how-does-the-vim-write-with-sudo-trick-work?answertab=votes#tab-top)