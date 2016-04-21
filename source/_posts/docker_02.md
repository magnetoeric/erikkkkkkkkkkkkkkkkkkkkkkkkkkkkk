title: docker laravel dev环境搭建
tags:
  - docker
  - php
category: docker
date: 2016-01-16 14:28:00

---
mbp上搭建了docker环境，就完完整整的写个docker for laravel的dev环境吧
(鉴于问题很多  就不贴具体代码了)
有docker环境的直接安装应该就可以了。
需要配置下本地的laravel目录，这样就可以把laravel目录挂载到docker 里了，就可以在本机里修改代码，docker中也会更新。
使用过程也碰到一些坑：

 - docker-machine重启vm后，vm内所有的东西全都没了！！这是最坑的，本来已经写好了，结果重启了一下，什么都没了，2天白干了，索性又花了一天时间去重写了一遍，效率还可以。
 - docker的挂载有些问题，挂载后的目录在docker里看的怪怪的，比如我挂载了一个/data目录,ls -al /data data的用户竟然是10000，让我很迷茫，后来发现，实际上是docker－machine里docker用户的uid
 - 默认的ubuntu 14.04里包很少，而我使用的编译安装，所以很多时候会找不到依赖，然后就要重新build。以前写的时候并没有很在意，最近发现，其实是有技巧的，docker是有类似缓存的，只要在dockerfile最下新增内容，实际上之前build成功的地方是有commit的，直接使用cache

从前并不认为经验很重要，这几天感悟很深，2天写的东西丢失之后，用了仅仅半天就搞定了。由此可见，做过和没做过是有差别的，用心做和做过也是有差别的。
这个环境有很多需要改进的地方，比如脚本路径有些问题；docker的基础环境，个人觉得应该是一层一层build出来的，而不是简单的像我这样，一次性安装完；
