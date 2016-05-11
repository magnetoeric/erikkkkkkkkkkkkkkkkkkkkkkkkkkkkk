title: php glob
tags:
  - php
categories:
  - php
date: 2016-05-11 14:13:00
---
## php查找符合某些规则的文件
通常实现查找目录下某些规则的文件，用shell实现起来很容易，但是php呢,个人想到的是遍历目录下所有文件并再用正则筛选.
然而php有现成的函数已经为我们做好了

```
说明
array glob ( string $pattern [, int $flags = 0 ] )

glob() 函数依照 libc glob() 函数使用的规则寻找所有与 pattern 匹配的文件路径，类似于一般 shells 所用的规则一样。不进行缩写扩展或参数替代。
参数

pattern

    The pattern. No tilde expansion or parameter substitution is done.
flags

    有效标记有：

        GLOB_MARK - 在每个返回的项目中加一个斜线
        GLOB_NOSORT - 按照文件在目录中出现的原始顺序返回（不排序）
        GLOB_NOCHECK - 如果没有文件匹配则返回用于搜索的模式
        GLOB_NOESCAPE - 反斜线不转义元字符
        GLOB_BRACE - 扩充 {a,b,c} 来匹配 'a'，'b' 或 'c'
        GLOB_ONLYDIR - 仅返回与模式匹配的目录项
        GLOB_ERR - 停止并读取错误信息（比如说不可读的目录），默认的情况下忽略所有错误

返回值

返回一个包含有匹配文件／目录的数组。如果出错返回 FALSE。

    Note:

    On some systems it is impossible to distinguish between empty match and an error. 
```

比如查找某目录下的txt文件并显示其大小
```

<?php
foreach (glob("*.txt") as $filename) {
    echo "$filename size " . filesize($filename) . "\n";
}
?>
```
