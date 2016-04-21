title: php如何获取pdo执行的sql语句
tags:
  - mysql
category: php
date: 2015-07-18 19:15:23

---

## 起因
这大概是很多人都想知道的吧,开发的时候,能看到程序真正执行的是什么可以减少很多时间去调试。
尤其是在当今IOC,MVC框架中,很多只懂得复制粘贴的程序员们遇到问题就只能问,根本不去深究为什么可以这样
废话真多,不说了,先来看这段代码
```
<?php 
class PDO_TEST{
    public function test($config,$id){
        $pdo = new PDO($config['host'],$config['user'],$config['pass'],$config['init_statements']);
        $sql = 'SELECT  * FROM book WHERE `id` = :id';
        $stmt = $pdo->prepare($sql);
        $stmt->bindParam(':id',$id);
        $stmt->execute();
        return $stmt->fetchAll();
    }
}
$config = array(
    'host'=>'mysql:host=192.168.188.48;dbname=test',
    'user' => 'admin',
    'pass' => 'admin',
    'init_statements' => array(
        'SET CHARACTER SET utf8',
        'SET NAMES utf8'
    ),
    'default_fetch_mode' => PDO::FETCH_ASSOC
);
$id =1;
$test = new PDO_TEST();
$result =$test->test($config,$id);
var_dump($result);exit;
?>
```
---
## tcpdump
如果直接var_dump($sql);返回的只是绑定的sql,即一个不完整的sql,还要去var_dump($id)，才能完整地显示出来sql语句，而有时候,这种调试显得异常笨拙。
我们急需一种方法来快速定位并解决sql问题

```
sudo tcpdump -n -X -S -i eth0 port 3306 and host 192.168.188.48
```
使用man tcpdump可以查看tcpdump工具的具体使用
```
-n Don't convert addresses (i.e., host addresses, port numbers, etc.) to names. #不要对地址进行转换
-X  When parsing and printing, in addition to printing the headers of each packet, print the data of each packet (minus its link level  
    header) in hex and ASCII.  This  is  very handy for analysing new protocols.
    #-x/-xx/-X/-XX：以十六进制显示包内容，几个选项只有细微的差别，详见man手册。
-S  Print absolute, rather than relative, TCP sequence numbers. #显示绝对的Sequence Number
-i  Listen on interface.  #我本地的网卡是eth0
```

其他的就不多解释了,也不会，跟着干货师傅学到的~
这样就能在本地看到有一部分包中包含如下内容

```
    0x0000:  4500 005d af32 4000 4006 8d5a c0a8 c08c  E..].2@.@..Z....
    0x0010:  c0a8 bc30 e48e 0cea 7929 223b 4f09 64ca  ...0....y)";O.d.
    0x0020:  8018 00e5 fe5d 0000 0101 080a 005f 8ee6  .....]......._..
    0x0030:  0069 7c2c 2500 0000 0353 454c 4543 5420  .i|,%....SELECT.
    0x0040:  202a 2046 524f 4d20 626f 6f6b 2057 4845  .*.FROM.book.WHE
    0x0050:  5245 2060 6964 6020 3d20 2731 27         RE.`id`.=.'1'
```
实际上能看到发送的包是因为mysql的预处理机制
[这里](http://www.oschina.net/translate/php-pdo-how-to)有一些关于预处理的翻译
这里的note也不错
With PDO_MYSQL you need to remember about the PDO::ATTR_EMULATE_PREPARES option.

The default value is TRUE, like
$dbh->setAttribute(PDO::ATTR_EMULATE_PREPARES,true); 

This means that no prepared statement is created with $dbh->prepare() call. With exec() call PDO replaces the placeholders with values itself and sends MySQL a generic query string.

The first consequence is that the call  $dbh->prepare('garbage');
reports no error. You will get an SQL error during the $dbh->exec() call.
The second one is the SQL injection risk in special cases, like using a placeholder for the table name.

The reason for emulation is a poor performance of MySQL with prepared statements. Emulation works significantly faster.


## 小结
如果添加如下改动,这种方式放弃了预编译
```
         $pdo = new PDO($config['dsn'],$config['username'],$config['password'],$config['init_statements']);
+        $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
         $sql = 'SELECT  * FROM book WHERE `id` = :id';
```
两个包的接收片段
```
    0x0000:  4500 005b a449 4000 4006 9845 c0a8 c08c  E..[.I@.@..E....
    0x0010:  c0a8 bc30 e73c 0cea a759 8d95 4b69 5fc7  ...0.<...Y..Ki_.
    0x0020:  8018 00e5 fe5b 0000 0101 080a 0064 2962  .....[.......d)b
    0x0030:  006e 16a6 2300 0000 1653 454c 4543 5420  .n..#....SELECT.
    0x0040:  202a 2046 524f 4d20 626f 6f6b 2057 4845  .*.FROM.book.WHE
    0x0050:  5245 2060 6964 6020 3d20 3f              RE.`id`.=.?

```
```
    0x0000:  4500 0048 a44a 4000 4006 9857 c0a8 c08c  E..H.J@.@..W....
    0x0010:  c0a8 bc30 e73c 0cea a759 8dbc 4b69 608e  ...0.<...Y..Ki`.
    0x0020:  8018 00ed fe48 0000 0101 080a 0064 2962  .....H.......d)b
    0x0030:  006e 16a6 1000 0000 1701 0000 0000 0100  .n..............
    0x0040:  0000 0001 fd00 0131                      .......1
```
 尽管exec方法和查询在PHP中仍然被大量使用和支持，但是PHP官网上还是要求大家用预处理语句的方式来替代。为什么呢？主要是因为：它更安全。预处理语句不会直接在实际查询中插入参数，这就避免了许多潜在的SQL注入。

然而出于某种原因，PDO实际上并没有真正的使用预处理，它是在模拟预处理方式，在将语句传给SQL服务器之前会把参数数据插入到语句中，这使得某些系统容易受到SQL注入。
实际上$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES,true); 使得pdo将语句进行了本地处理,并直接向mysql提交了sql语句。
而$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES,false);则是先向sql服务器发送prepare中的语句,接受到服务器返回的一些数据(meta data)，再发送参数
这样做安全,但是速度变慢了
