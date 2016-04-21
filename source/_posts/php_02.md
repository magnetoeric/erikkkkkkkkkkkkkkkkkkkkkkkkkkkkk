title: php strtotime
tags:
  - php
category: php
date: 2015-08-10 17:41:53

---
## 2014的最后一天
2014最后一天,写点有意思的吧

## strtotime
Parse about any English textual datetime description into a Unix timestamp
它是将英文文本转换成时间戳，说一下以前在循环生成当前月份的前36个月均价时遇到的问题,刚好今天是31号，很容易就会发现
```
php -r 'echo date("Ym",strtotime("-1 month",time()));'
```
显示的是201412 而不是期望的201411

strtotime在月份为31天的月里，都会出现问题,所以在寻找当前月份的前几月时，需要指定当前月的第一天  
第一种是我当时的写法，今天发现第二种方式也可以

```
date('Ym',strtotime("$i months ".date('Y-m-01', time())));
date("Ym",strtotime("first day of -$i month",time()))
```

stackoverflow上也有同样的问题,解决方式是这样的:
```
strtotime(date('Y-m',time()) . '-01 00:00:01')

```