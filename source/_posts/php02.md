title: php object2array array2object
tags:
  - php
category: php
date: 2015-11-29 20:02:00

---
今天看到有人问php中对象如何转化成数组，想到的第一个方式便是json encode和decode，不过这种方式在性能上会有些问题,因为序列化反序列化始终是个耗时的操作
```
json_decode(json_encode($object),true)
```
然后有人说
```
(array)$object
```
就可以转化了，试了一下很神奇，竟然可以。但是当一个对象属性是对象时，没法将该对象的属性也变成一个数组
所以还是需要写一个递归函数
```
function objectToArray($stobject){
    $_array = is_object($stobject) ? get_object_vars($stobject) : $stobject;
    if(is_array($_array)){
    foreach ($_array as $key => $value) {
        $value = (is_array($value) || is_object($value)) ? objectToArray($value) : $value;
        $array[$key] = $value;
    }}
    return $array;
}
```
array2object也是同样的道理，如果比较懒，可以直接使用json的方式
