title: laravel 的一些吐槽
tags:
  - php
  - laravel
category: php
date: 2016-01-12 22:02:00

---
最近在做业务项目迁移，把ec的代码迁移到laravel中，
为什么选用laravel，我也不知道(也不是我做选型)。其他的框架也没太多接触过，都是跑跑hello world，看的多一点的算是yaf了，不过个人理解，yaf没有orm，完全就是个vc框架，没有model层。需要自己去扩展，但是眼前项目很紧，明显不能这么用。以前用的框架是自己公司内部使用的，虽然思想的都不错，但是随便使用。
因为是公司内部物流系统，所以ui直接都是后端人员开发，ui框架选用的semantic，对于我这种不熟悉css，js的人来说也还不错，毕竟有文档，复制粘贴，大体的页面也就出来了。
但是开发过程中，还是想吐槽一下laravel。
1.最想吐槽的就是它的配置，放到.env里，每行其实就是个键值对，通过config目录下的配置读取，把这些配置替换过去，
但是我的项目配置不可能放在项目里啊，这是最基本的需求！因为不能对代码做入侵，所以可以在app初始化的时候，去set到外围目录里。为什么要set到外围目录？很简单，因为配置本就应该和代码分离，这样开发环境就能和线上环境区分出来了。但是这样远远不够，作为配置，很多项目是重复使用同一配置的，那就公用起来，做为每个项目不同的部分，也要区分出来，所以配置就成为了一个配置中心。以前在使用java开发时，比较简单，用的是superdiamond作为配置中心，公用的部分完全可以共用，独立的部分也不会被干扰。但是php显然不行，作为以进程为模型的语言，不可能每处理一个请求的时候就去读一遍配置，而应该是读取本地的某个固定目录的php文件，将配置载入。一旦配置发生改变（比如 redis迁移，mysql迁移，域名变更等）不需要一台一台地去更新配置，而是直接更新发布机的配置文件，直接发布到各机器上，这种方式可以使用ansible去做，Jenkins也可以做。也有自己去实现的，比如在每台机器上安装一个客户端，然后定时去和中心机器上的文件做对比，有不同就更新。想到的也就推模式和拉模式两种方式。
2.接下来就是blade模板，关于这个模板，文档实在是太少了，在页面上做变量嵌套的时候总是出问题，也无从查起，只能换种方式去做
3.log 这个就不多说了，实用的log并不多,php中的exception和fatal error是俩概念，无从和java的exception比拟。需要一个日志收集系统，去收集log，做分析。elk在这方面应该很强大，zabbix应该也可以。
4.分页，虽然可以自己实现，但是想短时间内做一个美观的无错误逻辑的，还是要花时间的。虽然这个在文档上有，但是用下来才发现，是给bootstrap模板使用的！一万个草泥马奔腾而过，然后github上找到了一份，还挺好用，虽然是个简单的实现。代码不贴了，在[这里](https://github.com/Landish/Pagination),感谢开源。因为想尽量保证少量的依赖包，所以直接拷贝到文件里了。
先吐槽这些吧，吐槽归吐槽，laravel也有很多优点，比如依赖注入，validation，orm等等，很方便，不多说了

