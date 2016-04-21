title: php urlencode
date: 2015-07-31 18:49:12
tags:
    - php
    - urlencode
category: php

---
公司里使用dubbo做rpc，启动时dubbo的service会向zookeeper注册其自身，包含一些参数，如版本号，服务地址等等信息，但是zookeeper里的信息是urldecode的，非常不友好，又不想总去找个网站做urldecode，这时候脚本语言的优势就显现出来了

```
<?php
echo urldecode($argv[1]);
```
非常简单的命令，执行时直接把zookeeper里的内容复制下来
php xxx.php $url　　就可以在cli里看到实际内容了
