title: 	SHELL编程之特殊符号
date: 2016-06-08 18:36:50
tags: shell
---
文章

微博ID：**orroz**

微信公众号：**Linux系统技术**

## 前言

本文是shell编程系列的第四篇，集中介绍了bash编程可能涉及到的特殊符号的使用。学会本文内容可以帮助你写出天书一样的bash脚本，并且顺便解决以下问题：

* 输入输出重定向是什么原理？
* exec 3<> /tmp/filename是什么鬼？
* 你玩过bash的关联数组吗？
* 如何不用if判断变量是否被定义？
* 脚本中字符串替换和删除操作不用sed怎么做？
* ” “和’ ‘有什么不同？
* 正则表达式和bash通配符是一回事么？

这里需要额外注意的是，相同的符号出现在不同的上下文中可能会有不同的含义。我们会在后续的讲解中突出它们的区别。

## 重定向(REDIRECTION)
重定向也叫输入输出重定向。我们先通过基本的使用对这个概念有个感性认识。

### 输入重定向

大家应该都用过cat命令，可以输出一个文件的内容。如：cat /etc/passwd。如果不给cat任何参数，那么cat将从键盘（标准输入）读取用户的输入，直接将内容显示到屏幕上，就像这样：

```
[zorro@zorrozou-pc0 bash]$ cat
hello 
hello
I am zorro!
I am zorro!
```

可以通过输入重定向让cat命令从别的地方读取输入，显示到当前屏幕上。最简单的方式是输入重定向一个文件，不过这不够“神奇”，我们让cat从别的终端读取输入试试。我当前使用桌面的终端terminal开了多个bash，使用ps命令可以看到这些终端所占用的输入文件是哪个：

```
[zorro@zorrozou-pc0 bash]$ ps ax|grep bash
 4632 pts/0    Ss     0:00 -bash
 5087 pts/2    S+     0:00 man bash
 5897 pts/1    Ss     0:00 -bash
 5911 pts/2    Ss     0:00 -bash
 9071 pts/4    Ss     0:00 -bash
11667 pts/3    Ss+    0:00 -bash
16309 pts/4    S+     0:00 grep --color=auto bash
19465 pts/2    S      0:00 sudo bash
19466 pts/2    S      0:00 bash
```

通过第二列可以看到，不同的bash所在的终端文件是哪个，这里的pts/3就意味着这个文件放在/dev/pts/3。我们来试一下，在pts/2对应的bash中输入：

```
[zorro@zorrozou-pc0 bash]$ cat < /dev/pts/3 
```

然后切换到pts/3所在的bash上敲入字符串，在pts/2的bash中能看见相关字符：

```
[zorro@zorrozou-pc0 bash]$ cat < /dev/pts/3 
safsdfsfsfadsdsasdfsafadsadfd
```

这只是个输入重定向的例子，一般我们也可以直接cat < /etc/passwd，表示让cat命令不是从默认输入读取，而是从/etc/passwd读取，这就是输入重定向，使用”<“。

### 输出重定向

绝大多数命令都有输出，用来显示给人看，所以输出基本都显示在屏幕（终端）上。有时候我们不想看到，就可以把输出重定向到别的地方：

[zorro@zorrozou-pc0 bash]$ ls /
bin  boot  cgroup  data  dev  etc  home  lib  lib64  lost+found  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[zorro@zorrozou-pc0 bash]$ ls / > /tmp/out
[zorro@zorrozou-pc0 bash]$ cat /tmp/out
bin
boot
cgroup
data
dev
......

使用一个”>”，将原本显示在屏幕上的内容给输出到了/tmp/out文件中。这个功能就是输出重定向。

### 报错重定向

命令执行都会遇到错误，一般也都是给人看的，所以默认还是显示在屏幕上。这些输出使用”>”是不能进行重定向的：

[zorro@zorrozou-pc0 bash]$ ls /1234 > /tmp/err
ls: cannot access '/1234': No such file or directory

可以看到，报错还是显示在了屏幕上。如果想要重定向这样的内容，可以使用”2>”：

[zorro@zorrozou-pc0 bash]$ ls /1234 2> /tmp/err
[zorro@zorrozou-pc0 bash]$ cat /tmp/err
ls: cannot access '/1234': No such file or directory

以上就是常见的输入输出重定向。在进行其它技巧讲解之前，我们有必要理解一下重定向的本质，所以要先从文件描述符说起。

## 文件描述符(file descriptor)

文件描述符简称fd，它是一个抽象概念，在很多其它体系下，它可能有其它名字，比如在C库编程中可以叫做文件流或文件流指针，在其它语言中也可以叫做文件句柄（handler），而且这些不同名词的隐含意义可能是不完全相同的。不过在系统层，还是应该使用系统调用中规定的名词，我们统一把它叫做文件描述符。

文件描述符本质上是一个数组下标（C语言数组）。在内核中，这个数组是用来管理一个进程打开的文件的对应关系的数组。就是说，对于任何一个进程来说，都有这样一个数组来管理它打开的文件，数组中的每一个元素和文件是映射关系，即：一个数组元素只能映射一个文件，而一个文件可以被多个数组元素所映射。

```
其实上面的描述并不完全准确，在内核中，文件描述符的数组所直接映射的实际上是文件表，文件表再索引到相关文件的v_node。具体可以参见《UNIX系统高级编程》。
```

shell在产生一个新进程后，新进程的前三个文件描述符都默认指向三个相关文件。这三个文件描述符对应的数组下标分别为0，1，2。0对应的文件叫做标准输入（stdin），1对应的文件叫做标准输出（stdout），2对应的文件叫做标准报错(stderr)。但是实际上，默认跟人交互的输入是键盘、鼠标，输出是显示器屏幕，这些硬件设备对于程序来说都是不认识的，所以操作系统借用了原来“终端”的概念，将键盘鼠标显示器都表现成一个终端文件。于是stdin、stdout和stderr就最重都指向了这所谓的终端文件上。于是，从键盘输入的内容，进程可以从标准输入的0号文件描述符读取，正常的输出内容从1号描述符写出，报错信息被定义为从2号描述符写出。这就是标准输入、标准输出和标准报错对应的描述符编号是0、1、2的原因。这也是为什么对报错进行重定向要使用2>的原因(其实1>也是可以用的)。

明白了以上内容之后，很多重定向的数字魔法就好理解了，比如：