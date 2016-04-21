title: ssh免密码登录 
tags:
  - linux
category: linux
date: 2015-08-10 15:28:00

---
测试环境机器的密码实在是太复杂了（没有跳板机），每次登录都要去找密码，所以干脆弄个免密码登录吧
首先是本地机器，因为装有git，所以已经生成好rsa公钥私钥了
如果没有生成过，可以使用 
``` 
ssh-keygen -t rsa
```
生成，连续回车就可以
```
cat ~/.ssh/id_rsa.pub
```
会有输出，这就是生成的公钥
接下来登录到目标机器
执行如下操作
```
sudo vi ~/.ssh/authorized_keys
```
将生成的公钥内容粘贴到最下面即可

或者 
```
echo 公钥内容 >> ~/.ssh/authorized_keys
```
修改文件权限
```
chmod 600 .ssh/authorized_keys
```
接下来在本地上就可以ssh免密码登录了