title: rsa加密
tags:
  - linux
categories:
    - other
date: 2017-04-22 19:35:00
---
# rsa加密
rsa算法相关
[RSA算法原理（一）](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
[RSA算法原理（二）](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)

最近研究百度登陆,发现百度登陆使用到了rsa加密,通过异步获取pubkey后，对password进行了rsa加密
具体php实现如下

```

function rsaSign($password,$pubKey) {
    openssl_public_encrypt($password,$sign,$pubKey);
    return base64_encode($sign);
}

 ```