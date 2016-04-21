title: nginx安装
tags:
  - nginx
category: linux
date: 2015-08-10 15:44:00

---
## 下载
[下载地址](http://nginx.org/en/download.html)
选择stable版本,下载的是.tar文件,执行

```
tar xvf nginx*.tar
```

## 编译
```
$./configure --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/ac\
cess.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/\
nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --with-http_ssl_modu\
le --with-http_realip_module  --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_\
module --with-http_stub_status_module --with-http_perl_module --with-mail --with-mail_ssl_module --add-module=/root/app/nginx-1.6.2/modules/ngx_ht\
tp_substitutions_filter_module  --with-pcre

$make 

$sudo make install

```

## 一些问题


1.pcre问题
./configure: error: the HTTP rewrite module requires the PCRE library.
需要安装pcre

2.安装第三方模块http_substitutions_filter_module   
[github地址](https://github.com/yaoweibin/ngx_http_substitutions_filter_module),不需要编译，参数- --add-module=/root/app/nginx-1.6.2/modules/ngx_ht\
tp_substitutions_filter_module 路径指定好就可以了

3.lc_all设置问题
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = "en_US:",
	LC_ALL = (unset),
	LC_CTYPE = "zh_CN.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
    
需要设置LC_ALL=C
```
$echo 'export LC_ALL=C' >>/etc/bash.bashrc

$source /etc/bash.bashrc
```
4.perl问题
/usr/bin/ld: cannot find -lperl

Linux下编译应用程序常常会出现如下错误：

/usr/bin/ld: cannot find -lxxx
意思是编译过程找不到对应库文件。其中，-lxxx表示链接库文件 libxxx.so。有时候，由于库文件是编译过程临时生成的，如果前面出错也会导致出现这种情况，下面针对的是由于本机系统环境缺失而引起的。

一般出现这种错误有以下几种原因：

* 系统缺乏对应的库文件；

* 版本不对应；

* 库文件的链接错误；

* 库文件路径设置问题；


对应第一第二种情况，可以通过下载安装lib来解决，ubuntu大多数可以直接通过apt-get来安装：

`apt-get install libxxx-dev`

所以执行

`
$apt-get install libperl-dev
`

---
## 安装pcre
[http下载地址](http://sourceforge.net/projects/pcre/files/pcre/)
下载完成后

```
$tar xvf pcre-8.36.tar

$cd pcre-8.36/

$./configure 

```

安装完会打印pcre信息

```
pcre-8.36 configuration summary:

    Install prefix .................. : /usr/local
    C preprocessor .................. : gcc -E
    C compiler ...................... : gcc
    C++ preprocessor ................ : g++ -E
    C++ compiler .................... : g++
    Linker .......................... : /usr/bin/ld -m elf_x86_64
    C preprocessor flags ............ : 
    C compiler flags ................ : -g -O2 -fvisibility=hidden
    C++ compiler flags .............. : -O2 -fvisibility=hidden -fvisibility-inlines-hidden
    Linker flags .................... : 
    Extra libraries ................. : 

    Build 8 bit pcre library ........ : yes
    Build 16 bit pcre library ....... : no
    Build 32 bit pcre library ....... : no
    Build C++ library ............... : yes
    Enable JIT compiling support .... : no
    Enable UTF-8/16/32 support ...... : no
    Unicode properties .............. : no
    Newline char/sequence ........... : lf
    \R matches only ANYCRLF ......... : no
    EBCDIC coding ................... : no
    EBCDIC code for NL .............. : n/a
    Rebuild char tables ............. : no
    Use stack recursion ............. : yes
    POSIX mem threshold ............. : 10
    Internal link size .............. : 2
    Nested parentheses limit ........ : 250
    Match limit ..................... : 10000000
    Match limit recursion ........... : MATCH_LIMIT
    Build shared libs ............... : yes
    Build static libs ............... : yes
    Use JIT in pcregrep ............. : no
    Buffer size for pcregrep ........ : 20480
    Link pcregrep with libz ......... : no
    Link pcregrep with libbz2 ....... : no
    Link pcretest with libedit ...... : no
    Link pcretest with libreadline .. : no
    Valgrind support ................ : no
    Code coverage ................... : no
```

执行make操作

```
$make

$sudo make install
```
pcre安装完成

