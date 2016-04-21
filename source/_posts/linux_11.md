title: 探索linux系统下proc文件系统内容
tags:
  - linux
category: linux
date: 2016-02-15 14:46:00

---
以下部分参考自[深入理解linux系统下proc文件系统内容](http://www.cnblogs.com/cute/archive/2011/04/20/2022280.html)
Linux系统上的/proc目录是一种文件系统，即proc文件系统。与其它常见的文件系统不同的是，/proc是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。 

基于/proc文件系统如上所述的特殊性，其内的文件也常被称作虚拟文件，并具有一些独特的特点。例如，其中有些文件虽然使用查看命令查看时会返回大量信息，但文件本身的大小却会显示为0字节。此外，这些特殊文件中大多数文件的时间及日期属性通常为当前系统时间和日期，这跟它们随时会被刷新（存储于RAM中）有关。 

为了查看及使用上的方便，这些文件通常会按照相关性进行分类存储于不同的目录甚至子目录中，如/proc/scsi目录中存储的就是当前系统上所有SCSI设备的相关信息，/proc/N中存储的则是系统当前正在运行的进程的相关信息，其中N为正在运行的进程（可以想象得到，在某进程结束后其相关目录则会消失）。 

大多数虚拟文件可以使用文件查看命令如cat、more或者less进行查看，有些文件信息表述的内容可以一目了然，但也有文件的信息却不怎么具有可读性。不过，这些可读性较差的文件在使用一些命令如apm、free、lspci或top查看时却可以有着不错的表现。 

### 进程目录中的常见文件介绍 

/proc目录中包含许多以数字命名的子目录，这些数字表示系统当前正在运行进程的进程号，里面包含对应进程相关的多个信息文件。
以下实验都是在基于docker的nginx容器里进行的，部分内容可能会与真实系统有些区别（用docker的原因是进程比较少，可以直接使用`docker run -it nginx:1.7 /bin/bash`创建一个nginx的容器并进入容器内部)

```
root@cdb3d33f6cb4:/# ls /bin/
bash   cp    dir	    egrep    gunzip    ln     mknod	  mv		 pwd	   run-parts   ss     tar	uname	      zcat    zforce
cat    dash  dmesg	    false    gzexe     login  mktemp	  nisdomainname  rbash	   sed	       stty   tempfile	uncompress    zcmp    zgrep
chgrp  date  dnsdomainname  fgrep    gzip      ls     more	  pidof		 readlink  sh	       su     touch	vdir	      zdiff   zless
chmod  dd    domainname     findmnt  hostname  lsblk  mount	  ping		 rm	   sh.distrib  sync   true	which	      zegrep  zmore
chown  df    echo	    grep     ip        mkdir  mountpoint  ping6		 rmdir	   sleep       tailf  umount	ypdomainname  fgrep  znew

```

容器中可用的命令少之又少，都是些基本的命令，但是有这些命令，可以做很多事情了，虽然很多命令我也不懂。

```
root@c27f523688ea:/# cd proc/
root@c27f523688ea:/proc# ls
1	   bus	      consoles	diskstats    fb		  iomem     kcore      kpagecount  meminfo  mtrr	  scsi	    stat	   sysvipc	tty	     vmstat
15	   cgroups    cpuinfo	dma	     filesystems  ioports   key-users  kpageflags  misc     net		  self	    swaps	   thread-self	uptime	     zoneinfo
acpi	   cmdline    crypto	driver	     fs		  irq	    keys       loadavg	   modules  pagetypeinfo  slabinfo  sys		   timer_list	version
buddyinfo  config.gz  devices	execdomains  interrupts   kallsyms  kmsg       locks	   mounts   partitions	  softirqs  sysrq-trigger  timer_stats	vmallocinfo
```

上面列出的是/proc目录中一些进程相关的目录，每个目录中是当程本身相关信息的文件。
下面我们来启动一个nginx，看看会发生什么

```
root@c27f523688ea:/proc# nginx
root@c27f523688ea:/proc# ls
1   acpi       cmdline	  crypto     driver	  fs	      irq	 keys	     loadavg  modules  pagetypeinfo  slabinfo  sys	      timer_list   version
17  buddyinfo  config.gz  devices    execdomains  interrupts  kallsyms	 kmsg	     locks    mounts   partitions    softirqs  sysrq-trigger  timer_stats  vmallocinfo
18  bus        consoles   diskstats  fb		  iomem       kcore	 kpagecount  meminfo  mtrr     scsi	     stat      sysvipc	      tty	   vmstat
19  cgroups    cpuinfo	  dma	     filesystems  ioports     key-users  kpageflags  misc     net      self	     swaps     thread-self    uptime	   zoneinfo

```

嗯？ 多出来3个数字命名的目录，对就是pid 为17，18，19的进程。再次ls，发现19小时，没错19应该是运行ls产生的进程，运行完成就消失了，猜测17，18一个是nginx master，另一个应该是worker
查看以下pid为18的进程，属组和用户都是nginx，更加确定它就是nginx的worker

```
root@c27f523688ea:/proc# ls -al 18
ls: cannot read symbolic link 18/cwd: Permission denied
ls: cannot read symbolic link 18/root: Permission denied
ls: cannot read symbolic link 18/exe: Permission denied
total 0
dr-xr-xr-x   9 nginx nginx 0 Feb 15 02:13 .
dr-xr-xr-x 134 root  root  0 Feb 15 02:11 ..
dr-xr-xr-x   2 nginx nginx 0 Feb 15 02:17 attr
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 autogroup
-r--------   1 nginx nginx 0 Feb 15 02:17 auxv
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 cgroup
--w-------   1 nginx nginx 0 Feb 15 02:17 clear_refs
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 cmdline
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 comm
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 coredump_filter
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 cpuset
lrwxrwxrwx   1 nginx nginx 0 Feb 15 02:17 cwd
-r--------   1 nginx nginx 0 Feb 15 02:17 environ
lrwxrwxrwx   1 nginx nginx 0 Feb 15 02:17 exe
dr-x------   2 nginx nginx 0 Feb 15 02:17 fd
dr-x------   2 nginx nginx 0 Feb 15 02:17 fdinfo
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 gid_map
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 limits
dr-x------   2 nginx nginx 0 Feb 15 02:17 map_files
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 maps
-rw-------   1 nginx nginx 0 Feb 15 02:17 mem
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 mountinfo
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 mounts
-r--------   1 nginx nginx 0 Feb 15 02:17 mountstats
dr-xr-xr-x   7 nginx nginx 0 Feb 15 02:17 net
dr-x--x--x   2 nginx nginx 0 Feb 15 02:17 ns
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 oom_adj
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 oom_score
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 oom_score_adj
-r--------   1 nginx nginx 0 Feb 15 02:17 pagemap
-r--------   1 nginx nginx 0 Feb 15 02:17 personality
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 projid_map
lrwxrwxrwx   1 nginx nginx 0 Feb 15 02:17 root
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 setgroups
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 smaps
-r--------   1 nginx nginx 0 Feb 15 02:17 stack
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 stat
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 statm
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 status
-r--------   1 nginx nginx 0 Feb 15 02:17 syscall
dr-xr-xr-x   3 nginx nginx 0 Feb 15 02:17 task
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 timers
-rw-r--r--   1 nginx nginx 0 Feb 15 02:17 uid_map
-r--r--r--   1 nginx nginx 0 Feb 15 02:17 wchar

```

有些文件没有权限，还是看nginx的master吧

```
root@c27f523688ea:/proc# ls -al 17/
total 0
dr-xr-xr-x   9 root root 0 Feb 15 02:13 .
dr-xr-xr-x 134 root root 0 Feb 15 02:11 ..
dr-xr-xr-x   2 root root 0 Feb 15 02:19 attr
-rw-r--r--   1 root root 0 Feb 15 02:19 autogroup
-r--------   1 root root 0 Feb 15 02:19 auxv
-r--r--r--   1 root root 0 Feb 15 02:19 cgroup
--w-------   1 root root 0 Feb 15 02:19 clear_refs
-r--r--r--   1 root root 0 Feb 15 02:19 cmdline
-rw-r--r--   1 root root 0 Feb 15 02:19 comm
-rw-r--r--   1 root root 0 Feb 15 02:19 coredump_filter
-r--r--r--   1 root root 0 Feb 15 02:19 cpuset
lrwxrwxrwx   1 root root 0 Feb 15 02:19 cwd -> /proc
-r--------   1 root root 0 Feb 15 02:19 environ
lrwxrwxrwx   1 root root 0 Feb 15 02:19 exe -> /usr/sbin/nginx
dr-x------   2 root root 0 Feb 15 02:19 fd
dr-x------   2 root root 0 Feb 15 02:19 fdinfo
-rw-r--r--   1 root root 0 Feb 15 02:19 gid_map
-r--r--r--   1 root root 0 Feb 15 02:19 limits
dr-x------   2 root root 0 Feb 15 02:19 map_files
-r--r--r--   1 root root 0 Feb 15 02:19 maps
-rw-------   1 root root 0 Feb 15 02:19 mem
-r--r--r--   1 root root 0 Feb 15 02:19 mountinfo
-r--r--r--   1 root root 0 Feb 15 02:19 mounts
-r--------   1 root root 0 Feb 15 02:19 mountstats
dr-xr-xr-x   7 root root 0 Feb 15 02:19 net
dr-x--x--x   2 root root 0 Feb 15 02:19 ns
-rw-r--r--   1 root root 0 Feb 15 02:19 oom_adj
-r--r--r--   1 root root 0 Feb 15 02:19 oom_score
-rw-r--r--   1 root root 0 Feb 15 02:19 oom_score_adj
-r--------   1 root root 0 Feb 15 02:19 pagemap
-r--------   1 root root 0 Feb 15 02:19 personality
-rw-r--r--   1 root root 0 Feb 15 02:19 projid_map
lrwxrwxrwx   1 root root 0 Feb 15 02:19 root -> /
-rw-r--r--   1 root root 0 Feb 15 02:19 setgroups
-r--r--r--   1 root root 0 Feb 15 02:19 smaps
-r--------   1 root root 0 Feb 15 02:19 stack
-r--r--r--   1 root root 0 Feb 15 02:19 stat
-r--r--r--   1 root root 0 Feb 15 02:19 statm
-r--r--r--   1 root root 0 Feb 15 02:19 status
-r--------   1 root root 0 Feb 15 02:19 syscall
dr-xr-xr-x   3 root root 0 Feb 15 02:19 task
-r--r--r--   1 root root 0 Feb 15 02:19 timers
-rw-r--r--   1 root root 0 Feb 15 02:19 uid_map
-r--r--r--   1 root root 0 Feb 15 02:19 wchan
```

接下来看看各个文件都是干吗用的吧

 - cmdline — 启动当前进程的完整命令，但僵尸进程目录中的此文件不包含任何信息;
 
```
root@c27f523688ea:/proc# cd 17
root@c27f523688ea:/proc/17# more cmdline
nginx: master process nginx
```

 - cwd 指向当前进程运行目录的一个符号链接； 

 - environ 当前进程的环境变量列表，彼此间用空字符（NULL）隔开；变量用大写字母表示，其值用小写字母表示；
 
```
root@c27f523688ea:/proc/17# more environ
 master process nginx
```

 - exe 指向启动当前进程的可执行文件（完整路径）的符号链接，通过/proc/N/exe可以启动当前进程的一个拷贝； 
 
```
root@c27f523688ea:/proc/17# ls -al exe
lrwxrwxrwx 1 root root 0 Feb 15 02:19 exe -> /usr/sbin/nginx
```
实际的可执行文件就是/usr/sbin/nginx

 - fd 这是个目录，包含当前进程打开的每一个文件的文件描述符（file descriptor），这些文件描述符是指向实际文件的一个符号链接;
 
 ```
 root@c27f523688ea:/proc/17# ls -al fd
total 0
dr-x------ 2 root root  0 Feb 15 02:19 .
dr-xr-xr-x 9 root root  0 Feb 15 02:13 ..
lrwx------ 1 root root 64 Feb 15 02:35 0 -> /dev/null
lrwx------ 1 root root 64 Feb 15 02:35 1 -> /dev/null
l-wx------ 1 root root 64 Feb 15 02:35 2 -> /1
lrwx------ 1 root root 64 Feb 15 02:35 3 -> socket:[63800]
l-wx------ 1 root root 64 Feb 15 02:35 4 -> /1
l-wx------ 1 root root 64 Feb 15 02:35 5 -> /1
lrwx------ 1 root root 64 Feb 15 02:35 6 -> socket:[63798]
lrwx------ 1 root root 64 Feb 15 02:35 7 -> socket:[63801]
 ```
 
 - limits 当前进程所使用的每一个受限资源的软限制、硬限制和管理单元；此文件仅可由实际启动当前进程的UID用户读取
 
 ```
 root@c27f523688ea:/proc/17# cat limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             1048576              1048576              processes
Max open files            1048576              1048576              files
Max locked memory         65536                65536                bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       3867                 3867                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     0                    0
Max realtime timeout      unlimited            unlimited            us
```
再运行 ulimit ，因为用户都是root，所以 ulimit应该和这个limits表示的内容是相同的

```
root@c27f523688ea:/proc/17# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3867
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 1048576
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
 - maps — 当前进程关联到的每个可执行文件和库文件在内存中的映射区域及其访问权限所组成的列表;
 看不懂 先过了

 
 ```
 root@c27f523688ea:/proc/17# cat maps
00400000-004c1000 r-xp 00000000 00:20 92                                 /usr/sbin/nginx
006c0000-006c1000 r--p 000c0000 00:20 92                                 /usr/sbin/nginx
006c1000-006d7000 rw-p 000c1000 00:20 92                                 /usr/sbin/nginx
006d7000-006e6000 rw-p 00000000 00:00 0
01c28000-01c85000 rw-p 00000000 00:00 0                                  [heap]
7fb1c3d4f000-7fb1c3d5a000 r-xp 00000000 00:20 47                         /lib/x86_64-linux-gnu/libnss_files-2.13.so
7fb1c3d5a000-7fb1c3f59000 ---p 0000b000 00:20 47                         /lib/x86_64-linux-gnu/libnss_files-2.13.so
7fb1c3f59000-7fb1c3f5a000 r--p 0000a000 00:20 47                         /lib/x86_64-linux-gnu/libnss_files-2.13.so
7fb1c3f5a000-7fb1c3f5b000 rw-p 0000b000 00:20 47                         /lib/x86_64-linux-gnu/libnss_files-2.13.so
7fb1c3f5b000-7fb1c3f65000 r-xp 00000000 00:20 45                         /lib/x86_64-linux-gnu/libnss_nis-2.13.so
7fb1c3f65000-7fb1c4164000 ---p 0000a000 00:20 45                         /lib/x86_64-linux-gnu/libnss_nis-2.13.so
7fb1c4164000-7fb1c4165000 r--p 00009000 00:20 45                         /lib/x86_64-linux-gnu/libnss_nis-2.13.so
7fb1c4165000-7fb1c4166000 rw-p 0000a000 00:20 45                         /lib/x86_64-linux-gnu/libnss_nis-2.13.so
7fb1c4166000-7fb1c417b000 r-xp 00000000 00:20 43                         /lib/x86_64-linux-gnu/libnsl-2.13.so
7fb1c417b000-7fb1c437a000 ---p 00015000 00:20 43                         /lib/x86_64-linux-gnu/libnsl-2.13.so
7fb1c437a000-7fb1c437b000 r--p 00014000 00:20 43                         /lib/x86_64-linux-gnu/libnsl-2.13.so
7fb1c437b000-7fb1c437c000 rw-p 00015000 00:20 43                         /lib/x86_64-linux-gnu/libnsl-2.13.so
7fb1c437c000-7fb1c437e000 rw-p 00000000 00:00 0
7fb1c437e000-7fb1c4385000 r-xp 00000000 00:20 41                         /lib/x86_64-linux-gnu/libnss_compat-2.13.so
7fb1c4385000-7fb1c4584000 ---p 00007000 00:20 41                         /lib/x86_64-linux-gnu/libnss_compat-2.13.so
7fb1c4584000-7fb1c4585000 r--p 00006000 00:20 41                         /lib/x86_64-linux-gnu/libnss_compat-2.13.so
7fb1c4585000-7fb1c4586000 rw-p 00007000 00:20 41                         /lib/x86_64-linux-gnu/libnss_compat-2.13.so
7fb1c4586000-7fb1c4588000 r-xp 00000000 00:20 34                         /lib/x86_64-linux-gnu/libdl-2.13.so
7fb1c4588000-7fb1c4788000 ---p 00002000 00:20 34                         /lib/x86_64-linux-gnu/libdl-2.13.so
7fb1c4788000-7fb1c4789000 r--p 00002000 00:20 34                         /lib/x86_64-linux-gnu/libdl-2.13.so
7fb1c4789000-7fb1c478a000 rw-p 00003000 00:20 34                         /lib/x86_64-linux-gnu/libdl-2.13.so
7fb1c478a000-7fb1c490b000 r-xp 00000000 00:20 36                         /lib/x86_64-linux-gnu/libc-2.13.so
7fb1c490b000-7fb1c4b0b000 ---p 00181000 00:20 36                         /lib/x86_64-linux-gnu/libc-2.13.so
7fb1c4b0b000-7fb1c4b0f000 r--p 00181000 00:20 36                         /lib/x86_64-linux-gnu/libc-2.13.so
7fb1c4b0f000-7fb1c4b10000 rw-p 00185000 00:20 36                         /lib/x86_64-linux-gnu/libc-2.13.so
7fb1c4b10000-7fb1c4b15000 rw-p 00000000 00:00 0
7fb1c4b15000-7fb1c4b2b000 r-xp 00000000 00:20 519                        /lib/x86_64-linux-gnu/libz.so.1.2.7
7fb1c4b2b000-7fb1c4d2a000 ---p 00016000 00:20 519                        /lib/x86_64-linux-gnu/libz.so.1.2.7
7fb1c4d2a000-7fb1c4d2b000 r--p 00015000 00:20 519                        /lib/x86_64-linux-gnu/libz.so.1.2.7
7fb1c4d2b000-7fb1c4d2c000 rw-p 00016000 00:20 519                        /lib/x86_64-linux-gnu/libz.so.1.2.7
7fb1c4d2c000-7fb1c4ef6000 r-xp 00000000 00:20 517                        /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0
7fb1c4ef6000-7fb1c50f6000 ---p 001ca000 00:20 517                        /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0
7fb1c50f6000-7fb1c5111000 r--p 001ca000 00:20 517                        /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0
7fb1c5111000-7fb1c5120000 rw-p 001e5000 00:20 517                        /usr/lib/x86_64-linux-gnu/libcrypto.so.1.0.0
7fb1c5120000-7fb1c5124000 rw-p 00000000 00:00 0
7fb1c5124000-7fb1c517a000 r-xp 00000000 00:20 516                        /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0
7fb1c517a000-7fb1c537a000 ---p 00056000 00:20 516                        /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0
7fb1c537a000-7fb1c537d000 r--p 00056000 00:20 516                        /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0
7fb1c537d000-7fb1c5384000 rw-p 00059000 00:20 516                        /usr/lib/x86_64-linux-gnu/libssl.so.1.0.0
7fb1c5384000-7fb1c53c0000 r-xp 00000000 00:20 514                        /lib/x86_64-linux-gnu/libpcre.so.3.13.1
7fb1c53c0000-7fb1c55c0000 ---p 0003c000 00:20 514                        /lib/x86_64-linux-gnu/libpcre.so.3.13.1
7fb1c55c0000-7fb1c55c1000 rw-p 0003c000 00:20 514                        /lib/x86_64-linux-gnu/libpcre.so.3.13.1
7fb1c55c1000-7fb1c55c9000 r-xp 00000000 00:20 512                        /lib/x86_64-linux-gnu/libcrypt-2.13.so
7fb1c55c9000-7fb1c57c8000 ---p 00008000 00:20 512                        /lib/x86_64-linux-gnu/libcrypt-2.13.so
7fb1c57c8000-7fb1c57c9000 r--p 00007000 00:20 512                        /lib/x86_64-linux-gnu/libcrypt-2.13.so
7fb1c57c9000-7fb1c57ca000 rw-p 00008000 00:20 512                        /lib/x86_64-linux-gnu/libcrypt-2.13.so
7fb1c57ca000-7fb1c57f8000 rw-p 00000000 00:00 0
7fb1c57f8000-7fb1c580f000 r-xp 00000000 00:20 73                         /lib/x86_64-linux-gnu/libpthread-2.13.so
7fb1c580f000-7fb1c5a0e000 ---p 00017000 00:20 73                         /lib/x86_64-linux-gnu/libpthread-2.13.so
7fb1c5a0e000-7fb1c5a0f000 r--p 00016000 00:20 73                         /lib/x86_64-linux-gnu/libpthread-2.13.so
7fb1c5a0f000-7fb1c5a10000 rw-p 00017000 00:20 73                         /lib/x86_64-linux-gnu/libpthread-2.13.so
7fb1c5a10000-7fb1c5a14000 rw-p 00000000 00:00 0
7fb1c5a14000-7fb1c5a34000 r-xp 00000000 00:20 29                         /lib/x86_64-linux-gnu/ld-2.13.so
7fb1c5c2a000-7fb1c5c2f000 rw-p 00000000 00:00 0
7fb1c5c30000-7fb1c5c31000 rw-s 00000000 00:01 63799                      /dev/zero (deleted)
7fb1c5c31000-7fb1c5c33000 rw-p 00000000 00:00 0
7fb1c5c33000-7fb1c5c34000 r--p 0001f000 00:20 29                         /lib/x86_64-linux-gnu/ld-2.13.so
7fb1c5c34000-7fb1c5c35000 rw-p 00020000 00:20 29                         /lib/x86_64-linux-gnu/ld-2.13.so
7fb1c5c35000-7fb1c5c36000 rw-p 00000000 00:00 0
7ffe5a572000-7ffe5a593000 rw-p 00000000 00:00 0                          [stack]
7ffe5a5e3000-7ffe5a5e5000 r--p 00000000 00:00 0                          [vvar]
7ffe5a5e5000-7ffe5a5e7000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
```

 - mem 当前进程所占用的内存空间，由open、read和lseek等系统调用使用，不能被用户读取；
 - root 指向当前进程运行根目录的符号链接；在Unix和Linux系统上，通常采用chroot命令使每个进程运行于独立的根目录；
 - stat 当前进程的状态信息，包含一系统格式化后的数据列，可读性差，通常由ps命令使用； 
 
 ```
 root@c27f523688ea:/proc/17# more stat
17 (nginx) S 1 17 17 0 -1 4218944 49 0 0 0 0 0 0 0 20 0 1 0 5428469 31825920 195 18446744073709551615 4194304 4982492 140730414209264 140730414208128 140401482384618 0 0 1073745920
402745863 0 0 0 17 0 0 0 0 0 0 7081352 7168288 29523968 140730414214953 140730414214959 140730414214959 140730414215144 0
```
 - status 与stat所提供信息类似，但可读性较好，如下所示，每行表示一个属性信息；其详细介绍请参见 proc的man手册页;
 
 ```
 root@c27f523688ea:/proc/17# more status
Name:	nginx
State:	S (sleeping)
Tgid:	17
Ngid:	0
Pid:	17
PPid:	1
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	64
Groups:
NStgid:	17
NSpid:	17
NSpgid:	17
NSsid:	17
VmPeak:	   31080 kB
VmSize:	   31080 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	     780 kB
VmRSS:	     780 kB
VmData:	     724 kB
VmStk:	     136 kB
VmExe:	     772 kB
VmLib:	    4500 kB
VmPTE:	      76 kB
VmPMD:	      12 kB
VmSwap:	       0 kB
Threads:	1
SigQ:	0/3867
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000040001000
SigCgt:	0000000198016a07
CapInh:	00000000a80425fb
CapPrm:	00000000a80425fb
CapEff:	00000000a80425fb
CapBnd:	00000000a80425fb
Seccomp:	0
Cpus_allowed:	1
Cpus_allowed_list:	0
Mems_allowed:	1
Mems_allowed_list:	0
voluntary_ctxt_switches:	1
--More--(0%) 
```
 - task 目录文件，包含由当前进程所运行的每一个线程的相关信息，每个线程的相关信息文件均保存在一个由线程号（tid）命名的目录中，这类似于其内容类似于每个进程目录中的内容

### /proc目录下常见的文件介绍 

 - /proc/apm 高级电源管理（APM）版本信息及电池相关状态信息，通常由apm命令使用(容器中没有)
 - /proc/buddyinfo 用于诊断内存碎片问题的相关信息文件(容器中没有)
 - /proc/cmdline 在启动时传递至内核的相关参数信息，这些信息通常由lilo或grub等启动管理工具进行传递
 - /proc/cpuinfo 处理器的相关信息的文件
 - /proc/crypto 系统上已安装的内核使用的密码算法及每个算法的详细信息列表
 - /proc/devices 系统已经加载的所有块设备和字符设备的信息，包含主设备号和设备组（与主设备号对应的设备类型）名
 - /proc/diskstats 每块磁盘设备的磁盘I/O统计信息列表
 - /proc/dma 每个正在使用且注册的ISA DMA通道的信息列表
 - /proc/execdomains 内核当前支持的执行域（每种操作系统独特“个性”）信息列表
 - /proc/fb 帧缓冲设备列表文件，包含帧缓冲设备的设备号和相关驱动信息；
 - /proc/filesystems 当前被内核支持的文件系统类型列表文件，被标示为nodev的文件系统表示不需要块设备的支持；通常mount一个设备时，如果没有指定文件系统类型将通过此文件来决定其所需文件系统的类型
 - /proc/interrupts X86或X86_64体系架构系统上每个IRQ相关的中断号列表；多路处理器平台上每个CPU对于每个I/O设备均有自己的中断号
 - /proc/iomem 每个物理设备上的记忆体（RAM或者ROM）在系统内存中的映射信息
 - /proc/ioports 当前正在使用且已经注册过的与物理设备进行通讯的输入-输出端口范围信息列表；如下面所示，第一列表示注册的I/O端口范围，其后表示相关的设备
 - /proc/kallsyms 模块管理工具用来动态链接或绑定可装载模块的符号定义，由内核输出;通常这个文件中的信息量相当大；
 - /proc/kcore 系统使用的物理内存，以ELF核心文件（core file）格式存储，其文件大小为已使用的物理内存（RAM）加上4KB；这个文件用来检查内核数据结构的当前状态，因此，通常由GDB通常调试工具使用，但不能使用文件查看命令打开此文件
 - /proc/kmsg 此文件用来保存由内核输出的信息，通常由/sbin/klogd或/bin/dmsg等程序使用，不要试图使用查看命令打开此文件
 - /proc/locks 保存当前由内核锁定的文件的相关信息，包含内核内部的调试数据；每个锁定占据一行，且具有一个惟一的编号；如下输出信息中每行的第二列表示当前锁定使用的锁定类别，POSIX表示目前较新类型的文件锁，由lockf系统调用产生，FLOCK是传统的UNIX文件锁，由flock系统调用产生；第三列也通常由两种类型，ADVISORY表示不允许其他用户锁定此文件，但允许读取，MANDATORY表示此文件锁定期间不允许其他用户任何形式的访问；  
 - /proc/meminfo 系统中关于当前内存的利用状况等的信息，常由free命令使用；可以使用文件查看命令直接读取此文件，其内容显示为两列，前者为统计属性，后者为对应的值
 
 ```
 root@c27f523688ea:/proc# cat meminfo
MemTotal:        1020096 kB
MemFree:          746572 kB
MemAvailable:     802920 kB
Buffers:           41512 kB
Cached:           139888 kB
SwapCached:            0 kB
Active:           135888 kB
Inactive:          92792 kB
Active(anon):      83496 kB
Inactive(anon):    90648 kB
Active(file):      52392 kB
Inactive(file):     2144 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       1182172 kB
SwapFree:        1182172 kB
Dirty:                 8 kB
Writeback:             0 kB
AnonPages:         47276 kB
Mapped:            29256 kB
Shmem:            126868 kB
Slab:              28464 kB
SReclaimable:      16392 kB
SUnreclaim:        12072 kB
KernelStack:        2432 kB
PageTables:         1616 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     1692220 kB
Committed_AS:     371556 kB
VmallocTotal:   34359738367 kB
VmallocUsed:        8908 kB
VmallocChunk:   34359699820 kB
AnonHugePages:     32768 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:       47040 kB
DirectMap2M:     1001472 kB
```
 - /proc/mounts 在内核2.4.29版本以前，此文件的内容为系统当前挂载的所有文件系统，在2.4.19以后的内核中引进了每个进程使用独立挂载名称空间的方式，此文件则随之变成了指向/proc/self/mounts（每个进程自身挂载名称空间中的所有挂载点列表）文件的符号链接；/proc/self是一个独特的目录，后文中会对此目录进行介绍

```
root@c27f523688ea:/proc# ls -al mounts
lrwxrwxrwx 1 root root 11 Feb 15 06:18 mounts -> self/mounts
```
其中第一列表示挂载的设备，第二列表示在当前目录树中的挂载点，第三点表示当前文件系统的类型，第四列表示挂载属性（ro或者rw），第五列和第六列用来匹配/etc/mtab文件中的转储（dump）属性

```
root@c27f523688ea:/proc# more mounts
none / aufs rw,relatime,si=6bfa28fca2bd64f1,dio,dirperm1 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev tmpfs rw,nosuid,mode=755 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666 0 0
sysfs /sys sysfs ro,nosuid,nodev,noexec,relatime 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,relatime,mode=755 0 0
cgroup /sys/fs/cgroup/cpuset cgroup ro,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/cpu cgroup ro,nosuid,nodev,noexec,relatime,cpu 0 0
cgroup /sys/fs/cgroup/cpuacct cgroup ro,nosuid,nodev,noexec,relatime,cpuacct 0 0
cgroup /sys/fs/cgroup/blkio cgroup ro,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/memory cgroup ro,nosuid,nodev,noexec,relatime,memory 0 0
cgroup /sys/fs/cgroup/devices cgroup ro,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/freezer cgroup ro,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/net_cls cgroup ro,nosuid,nodev,noexec,relatime,net_cls 0 0
cgroup /sys/fs/cgroup/perf_event cgroup ro,nosuid,nodev,noexec,relatime,perf_event 0 0
cgroup /sys/fs/cgroup/net_prio cgroup ro,nosuid,nodev,noexec,relatime,net_prio 0 0
cgroup /sys/fs/cgroup/hugetlb cgroup ro,nosuid,nodev,noexec,relatime,hugetlb 0 0
```
可以看到/proc 使用的是proc文件系统,大多数的ubuntu镜像使用的是aufs文件系统，当然可以在启动镜像时的时候指定storage driver

 - /proc/modules 当前装入内核的所有模块名称列表，可以由lsmod命令使用，也可以直接查看；如下所示，其中第一列表示模块名，第二列表示此模块占用内存空间大小，第三列表示此模块有多少实例被装入，第四列表示此模块依赖于其它哪些模块，第五列表示此模块的装载状态（Live：已经装入；Loading：正在装入；Unloading：正在卸载），第六列表示此模块在内核内存（kernel memory）中的偏移量
  
 ```
 root@c27f523688ea:/proc# cat modules
veth 16384 0 - Live 0xffffffffa0173000
xt_conntrack 16384 1 - Live 0xffffffffa0152000
ipt_MASQUERADE 16384 1 - Live 0xffffffffa014d000
nf_nat_masquerade_ipv4 16384 1 ipt_MASQUERADE, Live 0xffffffffa01a5000
```
 - proc/partitions 块设备每个分区的主设备号（major）和次设备号（minor）等信息，同时包括每个分区所包含的块（block）数目（如下面输出中第三列所示)
 
 ```
 root@c27f523688ea:/proc# more partitions
major minor  #blocks  name

   1        0      65535 ram0
   1        1      65535 ram1
   1        2      65535 ram2
   1        3      65535 ram3
   1        4      65535 ram4
   1        5      65535 ram5
   1        6      65535 ram6
   1        7      65535 ram7
 250        0     194216 zram0
  11        0      30720 sr0
   8        0   20480000 sda
   8        1   19486845 sda1
   8        2     987966 sda2
 ```

 - /proc/pci 
内核初始化时发现的所有PCI设备及其配置信息列表，其配置信息多为某PCI设备相关IRQ信息，可读性不高，可以用“/sbin/lspci –vb”命令获得较易理解的相关信息；在2.6内核以后，此文件已为/proc/bus/pci目录及其下的文件代替
- /proc/slabinfo 在内核中频繁使用的对象（如inode、dentry等）都有自己的cache，即slab pool，而/proc/slabinfo文件列出了这些对象相关slap的信息；详情可以参见内核文档中slapinfo的手册页
- /proc/stat 
实时追踪自系统上次启动以来的多种统计信息；如下所示，其中

	“cpu”行后的八个值分别表示以1/100（jiffies）秒为单位的统计值（包括系统运行于用户模式、低优先级用户模式，运系统模式、空闲模式、I/O等待模式的时间等）
	
	“intr”行给出中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数； 
	
	“ctxt”给出了自系统启动以来CPU发生的上下文交换的次数。 

	“btime”给出了从系统启动到现在为止的时间，单位为秒； 

	“processes (total_forks) 自系统启动以来所创建的任务的个数目； 

	“procs_running”：当前运行队列的任务的数目； 

	“procs_blocked”：当前被阻塞的任务的数目； 
- /proc/swaps 当前系统上的交换分区及其空间利用信息，如果有多个交换分区的话，则会每个交换分区的信息分别存储于/proc/swap目录中的单独文件中，而其优先级数字越低，被使用到的可能性越大 



``` 
root@c27f523688ea:/proc# cat swaps
Filename				Type		Size	Used	Priority
/dev/zram0                              partition	194212	0	-1
/dev/sda2                               partition	987960	0	-2
```
 - /proc/uptime 系统上次启动以来的运行时间,如下所示，其第一个数字表示系统运行时间，第二个数字表示系统空闲时间，单位是秒；
 
```
 	root@c27f523688ea:/proc# cat uptime
	70344.11 70304.83
```
 - /proc/version 
当前系统运行的内核版本号，还会显示系统安装的gcc版本，如下所示；

```
root@c27f523688ea:/proc# cat version
Linux version 4.1.13-boot2docker (root@11aafb97cfeb) (gcc version 4.9.2 (Debian 4.9.2-10) ) #1 SMP Fri Nov 20 19:05:50 UTC 2015
```
 - /proc/vmstat 
当前系统虚拟内存的多种统计数据，信息量可能会比较大，这因系统而有所不同，可读性较好
 - /proc/zoneinfo 内存区域（zone）的详细信息列表，信息量较大

### /proc/sys目录详解 

与 /proc下其它文件的“只读”属性不同的是，管理员可对/proc/sys子目录中的许多文件内容进行修改以更改内核的运行特性，事先可以使用“ls -l”命令查看某文件是否“可写入”。写入操作通常使用类似于“echo  DATA > /path/to/your/filename”的格式进行。需要注意的是，即使文件可写，其一般也不可以使用编辑器进行编辑。 

 - /proc/sys/debug 子目录 
此目录通常是一空目录； 

 - /proc/sys/dev 子目录 
为系统上特殊设备提供参数信息文件的目录，其不同设备的信息文件分别存储于不同的子目录中，如大多数系统上都会具有的/proc/sys/dev /cdrom和/proc/sys/dev/raid（如果内核编译时开启了支持raid的功能） 目录，其内存储的通常是系统上cdrom和raid的相关参数信息文件。

照着搬过来很多内容，了解了一些，一些还是不懂，写博客的好处，就是可以温故而知新

 
