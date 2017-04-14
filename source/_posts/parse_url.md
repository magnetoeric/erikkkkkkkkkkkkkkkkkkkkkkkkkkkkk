title: php 解析url
tags:
  - php
categories:
    - php
date: 2017-04-12 19:35:00
---
很多人喜欢用正则提取url中的domain,path及参数信息
实际上php已经提供了对url解析的方法:[parse_url](http://php.net/manual/en/function.parse-url.php)
一个url可以分解成如下几个部分

* scheme  协议
* host  主机
* port  端口
* user  用户名
* pass  密码
* path  路径
* query 参数
* fragment 

直接调用parse_url
可以将url解析成一个数组,可以通过索引名称获取到对应的值

```
<?php
$url = 'http://username:password@hostname:9090/path?arg=value#anchor';

var_dump(parse_url($url));
var_dump(parse_url($url, PHP_URL_SCHEME));
var_dump(parse_url($url, PHP_URL_USER));
var_dump(parse_url($url, PHP_URL_PASS));
var_dump(parse_url($url, PHP_URL_HOST));
var_dump(parse_url($url, PHP_URL_PORT));
var_dump(parse_url($url, PHP_URL_PATH));
var_dump(parse_url($url, PHP_URL_QUERY));
var_dump(parse_url($url, PHP_URL_FRAGMENT));
?>
```
