title: php curl file_get_contents比较
tags:
  - php
categories: []
date: 2016-05-09 20:32:00
---
## 问题
最近发布系统出现了一些速度问题，而且时好时坏，网络非常不稳定，查了一下，发现代码里出现很多file_get_contents函数

## 比较
写两个简单的例子吧
curl 

```
$i= 0;
do{
    $t1 = microtime();
    $url = "http://www.anjuke.com";
    file_get_contents($url);
    $t2 = microtime();
    echo (($t2-$t1)<0?($t2-$t1)+1:($t2-$t1)).PHP_EOL;
}while($i++<2);
```

file_get_contents

```
$i= 0;
do{
    $t1 = microtime();
    $url = "http://www.anjuke.com";
    file_get_contents($url);
    $t2 = microtime();
    echo (($t2-$t1)<0?($t2-$t1)+1:($t2-$t1)).PHP_EOL;
}while($i++<20);

```

测试了一下，速度相差一个数量级,为何相差如此悬殊，查了些资料，原来file_get_contents会刷新dns缓存。
使用strace 追踪,发现其会读取/etc/hosts 和/etc/resolv.conf 文件,找到dns server,而curl不会。

此外，curl相对自由一些，file_get_contents不支持post请求等等



相关参考 :

	[fsockopen/curl/file_get_contents效率比较](http://www.nowamagic.net/academy/detail/12220248)