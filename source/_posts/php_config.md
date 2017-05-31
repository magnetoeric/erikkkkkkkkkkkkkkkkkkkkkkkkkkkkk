title: 读取config的几种方式
tags:
  - php
categories:
    - php
date: 2017-04-02 22:15:00
---
## 几种方式
### .ini 
parse_ini_file() 载入一个由 filename 指定的 ini 文件，并将其中的设置作为一个联合数组返回。

ini 文件的结构和 php.ini 的相似。
一下是一个名为 db.ini的配置文件

```
[db]
host=0.0.0.0
port=1234
username=username
password=pass
```

那么使用parse_ini_file读取也很简单

```
$config = parse_ini_file('db.ini',true);
print_r($config);exit;
```

会返回一个key为db的二维数组
### .env
利用php的环境变量 $_ENV

```
通过环境方式传递给当前脚本的变量的数组。

这些变量被从 PHP 解析器的运行环境导入到 PHP 的全局命名空间。很多是由支持 PHP 运行的 Shell 提供的，并且不同的系统很可能运行着不同种类的 Shell，所以不可能有一份确定的列表。请查看你的 Shell 文档来获取定义的环境变量列表。

其他环境变量包含了 CGI 变量，而不管 PHP 是以服务器模块还是 CGI 处理器的方式运行。

$HTTP_ENV_VARS 包含相同的信息，但它不是一个超全局变量。 (注意 $HTTP_ENV_VARS 和 $_ENV 是不同的变量，PHP 处理它们的方式不同)
```

推荐使用php的[phpdotenv](https://github.com/vlucas/phpdotenv)
使用起来非常简单，laravel里就是使用dotenv作为配置的

### yaml
这个没怎么用过，不太喜欢靠缩进风格的代码。可读性太差
这里有很详尽的说明[PHP下处理YAML](https://segmentfault.com/a/1190000000409556)以及
[symfony/yaml](https://packagist.org/packages/symfony/yaml)

### xml
php中很少见到有使用xml的，不过spring基本上是以这种方式加载大多数配置的(后续版本中也有大量使用annotition来声明配置)，一个主要原因是可读性很强，另一个原因也是规范化

### array & json
很多公司自己实现的框架中都是依靠数组去实现配置的。这就导致配置文件的解析耗费了很大的性能


## yaconf
最后推荐鸟哥的[yaconf](http://www.laruence.com/2015/06/12/3051.html),由于php的生命周期,导致每次读取配置时都需要解析配置文件,而yaconf是常驻内存的，而且支持动态更新等等，优势非常明显，缺点也很突出,仅支持php7,但是php7的项目推荐使用yaconf
