title: 生成N个不重复的随机数
tags:
  - php
category: php
date: 2015-08-10 17:38:00

---
##生成N个不重复的随机数
转自[Veda 原型](http://www.nowamagic.net/librarys/veda/detail/2508)  
  
有25幅作品拿去投票，一次投票需要选16幅，单个作品一次投票只能选择一次。前面有个程序员捅了漏子，忘了把投票入库，有200个用户产生的投票序列为空。那么你会如何填补这个漏子？   

当然向上级反映情况。但是我们这里讨论的是技术，就是需要生成1-25之间的16个不重复的随机数，去填补。具体怎么设计函数呢？将随机数存入数组，再在数组中去除重复的值，即可生成一定数量的不重复随机数。

```
<?php
/*
* array unique_rand( int $min, int $max, int $num )
* 生成一定数量的不重复随机数
* $min 和 $max: 指定随机数的范围
* $num: 指定生成数量
*/
function unique_rand($min, $max, $num) {
    $count = 0;
    $return = array();
    while ($count < $num) {
        $return[] = mt_rand($min, $max);
        $return = array_flip(array_flip($return));
        $count = count($return);
    }
    shuffle($return);
    return $return;
}

$arr = unique_rand(1, 25, 16);
sort($arr);

$result = '';
$result = '';
for($i=0; $i < count($arr);$i++)
{
	$result .= $arr[$i].',';
}
$result = substr($result, 0, -1);
echo $result;
?>

```
补充几点说明：  
*    生成随机数时用了 mt_rand() 函数。这个函数生成随机数的平均速度要比 rand() 快四倍。 
去除数组中的重复值时用了“翻翻法”，就是用 array_flip() 把数组的 key 和 value 交换两次。这种做法比用 array_unique() 快得多。
*    返回数组前，先使用 shuffle() 为数组赋予新的键名，保证键名是 0-n 连续的数字。如果不进行此步骤，可能在删除重复值时造成键名不连续，给遍历带来麻烦。

##一些优化
为了尊重作者，我还是不在原文里对内容进行修改了   
*    for循环在使用时切记不要使用count(array)方式，php不像java会对这样的代码进行优化，这样的代码造成的结果是数组有多少个元素，就会执行多少次count(array)。  
*    把数组拼接成字符串可以使用implode函数

```
 string implode ( string $glue , array $pieces )
```