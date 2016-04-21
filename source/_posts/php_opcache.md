title: read-phps-opcode
tags:
  - php
category: php
date: 2015-08-10 17:32:00

---

## Vulcan Logic Disassembler
本文翻译自http://rancoud.com/read-phps-opcode/  
VLD(Vulcan Logic Disassembler)是一个向你解释zend引擎如何将你的脚本转化成opcode的扩展.  
[扩展下载地址](http://pecl.php.net/package/vld)  
我将解释如何安装，使用并阅读opcode

首先，你要安装PEAR  
`aptitude install php-pear`  
然后就可以添加vld  
`pecl install "channel://pecl.php.net/vld-0.12.0"`  
编辑php.ini文件使扩展生效  
注意，这个文件是cli模式的,跟php.ini不同   
`nano /etc/php5/cli/php.ini`  
将下面这行加到静态扩展区域  
`extension=vld.so`
## 如何使用
```
php -dvld.dump_paths=1 -dvld.verbosity=0 -dvld.save_paths=1 -dvld.active=1 yourscript.php
```

以下列出参数列表及默认值

```
# Activate extension: boolean 
vld.active	 0
# The caracter used for writing columns: carater
vld.col_sep	 "\t"
# Dumps branch and paths: boolean
vld.dump_paths	 1
# Execute code in the same time: boolean
vld.execute	 1
# Format output analyze: boolean
vld.format	 0
# Where to save the file for graphviz .dot file: string
vld.save_dir	    /tmp
# dump path information has a graphviz .dot file: boolean
vld.save_paths	 0
# suppresses output and loading of auto_append_file respectively if vld.active=1 and vld.execute=0: boolean
vld.skip_append	 0
# suppresses output and loading of auto_prepend_file respectively if vld.active=1 and vld.execute=0: boolean
vld.skip_prepend 0
# Verbosities' level (0:none, 1, 2, 3): integer
vld.verbosity	 1
```
现在我门可以看这个网站的结果  
(http://blog.pascal-martin.fr/post/php-obtenir-dump-opcodes)  
php代码  

```
<?php
for ($i=0 ; $i>10 ; $i++) { 
    if ($i % 2 === 0) {
        echo "Pair : ";
    }
    else {
        echo "Impair : ";
    }
    echo $i, "\n";
}
```

VLD 输出  

```
$ ~/bin/php-5.3/bin/php -dextension=vld.so -dvld.active=1 -dvld.verbosity=0 -dvld.execute=0 exemples/003-control.php 
filename:       /.../exemples/003-control.php
function name:  (null)
number of ops:  16
compiled vars:  !0 = $i
line     # *  op                           fetch          ext  return  operands
---------------------------------------------------------------------------------
   2     0  >   ASSIGN                                                   !0, 0
         1  >   IS_SMALLER                                       ~1      !0, 10
         2    > JMPZNZ                                        6          ~1, ->15
         3  >   POST_INC                                         ~2      !0
         4      FREE                                                     ~2
         5    > JMP                                                      ->1
   3     6  >   MOD                                              ~3      !0, 2
         7      IS_IDENTICAL                                     ~4      ~3, 0
         8    > JMPZ                                                     ~4, ->11
   4     9  >   ECHO                                                     'Pair+%3A+'
   5    10    > JMP                                                      ->12
   7    11  >   ECHO                                                     'Impair+%3A+'
   9    12  >   ECHO                                                     !0
        13      ECHO                                                     '%0A'
  10    14    > JMP                                                      ->3
  11    15  > > RETURN                                                   1
 
branch: #  0; line:     2-    2; sop:     0; eop:     0; out1:   1
branch: #  1; line:     2-    2; sop:     1; eop:     2; out1:  15; out2:   6
branch: #  3; line:     2-    2; sop:     3; eop:     5; out1:   1
branch: #  6; line:     3-    3; sop:     6; eop:     8; out1:   9; out2:  11
branch: #  9; line:     4-    5; sop:     9; eop:    10; out1:  12
branch: # 11; line:     7-    9; sop:    11; eop:    11; out1:  12
branch: # 12; line:     9-   10; sop:    12; eop:    14; out1:   3
branch: # 15; line:    11-   11; sop:    15; eop:    15
path #1: 0, 1, 15, 
path #2: 0, 1, 6, 9, 12, 3, 1, 15, 
path #3: 0, 1, 6, 11, 12, 3, 1, 15,
```

## 如何阅读
在表格上面有两行   
*    操作执行的数量:有多少opcode操作将会被执行，这里是16  
*    编译的变量:php变量和它的opcode version，这里$i被表示成!0

## 其他方式
只翻译了部分，其他的不翻译了，因为我喜欢编译安装，所以vld也是编译安装  
根据[下载地址](http://pecl.php.net/package/vld)中的version，安装适合自己php的vld版本
可以使用`php -v `查看自己php的版本号
下载好后

```
$tar zxvf vld-0.12.0.tgz #解压
$cd vld-0.12.0/
$/usr/local/php/bin/phpize  #根据自己的phpize实际位置使用
$./configure --with-php-config=/usr/local/php/bin/php-config --enable-vld
$sudo make 
$sudo make install
```
这样就安装完成了，然后在php.ini中添加  
`extension=vld.so`  
一般编译安装的php.ini 放在php目录下的lib目录中  
可以使用`php -i |grep vld`查看是否安装成功
