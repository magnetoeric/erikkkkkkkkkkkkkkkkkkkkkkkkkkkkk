title: 30天了解30种技术（一)
tags:
  - bower
category: nodejs
date: 2015-07-20 10:08:00

---
## 关于30种技术
这是[segmentfault](http://segmentfault.com/a/1190000000349384)上的一个系列,做业务做累了就该找点东西玩玩,不是么?
但是说实话,这仅仅是了解,熟悉真谈不上,业务中没有真正使用到,都是纸上谈兵

---
## 第一天 Bower
原文地址:[Day 1: Bower —— 管理你的客户端依赖关系](http://segmentfault.com/a/1190000000349555)
英文原文在[这里](https://www.openshift.com/blogs/day-1-bower-manage-your-client-side-dependencies)
由于环境的千差万别,遇到的问题肯定会不一样,所以我在跟着原文使用时也会遇到一些问题,我会将我遇到的问题整理出来
Bower是一个客户端技术的软件包管理器，它可用于搜索、安装和卸载如JavaScript、HTML、CSS之类的网络资源。其他一些建立在Bower基础之上的开发工具，如YeoMan和Grunt，这个会跟着写 :)

---
## 为什么我会在意Bower?
* 节省时间。为什么要学习Bower的第一个原因，就是它会为你节省寻找客户端的依赖关系的时间。每次我需要安装jQuery的时候，我都需要去jQuery网站下载包或使用CDN版本。但是有了Bower，你只需要输入一个命令，jquery就会安装在本地计算机上，你不需要去记版本号之类的东西，你也可以通过Bower的info命令去查看任意库的信息。
* 脱机工作。Bower会在用户主目录下创建一个。bower的文件夹，这个文件夹会下载所有的资源、并安装一个软件包使它们可以离线使用。如果你熟悉Java，Bower即是一个类似于现在流行的Maven构建系统的。m2仓库。每次你下载任何资源库都将被安装在两个文件夹中一个在的应用程序文件夹，另一个在用户主目录下的。bower文件夹。因此，下一次你需要这个仓库时，就会用那个用户主目录下.bower中的版本。
* 可以很容易地展现客户端的依赖关系。你可以创建一个名为bower。json的文件，在这个文件里你可以指定所有客户端的依赖关系，任何时候你需要弄清楚你正在使用哪些库，你可以参考这个文件。
* 让升级变得简单。假设某个库的新版本发布了一个重要的安全修补程序，为了安装新版本，你只需要运行一个命令，bower会自动更新所有有关新版本的依赖关系。

---
## 前提准备
为了安装bower，你首先需要安装如下文件：
* node.js + npm  安装步骤看[这里](http://magnetoeric.github.io/2014/09/24/second/)
* git 需要从github上下载代码包，github都不会用也不会找到我的博客 :)

---
## 安装bower
安装好准备的东西,就可以安装bower了
`
sudo npm install -g bower
`
-g 表示全局安装

---
## 运行bower
```
$ bower help
Usage:

    bower <command> [<args>] [<options>]
Commands:

    cache                   Manage bower cache
    help                    Display help information about Bower
    home                    Opens a package homepage into your favorite browser
    info                    Info of a particular package
    init                    Interactively create a bower.json file
    install                 Install a package locally
    link                    Symlink a package folder
    list                    List local packages - and possible updates
    lookup                  Look up a package URL by name
    prune                   Removes local extraneous packages
    register                Register a package
    search                  Search for a package by name
    update                  Update a local package
    uninstall               Remove a local package
    version                 Bump a package version
Options:

    -f, --force             Makes various commands more forceful
    -j, --json              Output consumable JSON
    -l, --log-level         What level of logs to report
    -o, --offline           Do not hit the network
    -q, --quiet             Only output important information
    -s, --silent            Do not output anything, besides errors
    -V, --verbose           Makes output more verbose
    --allow-root            Allows running commands as root
    --version               Output Bower version
See 'bower help <command>' for more information on a specific command.
```
运行bower help 时遇到了如下错误 
/usr/bin/env: node: 没有那个文件或目录

修改/usr/local/bin/bower中的第一行，
`#!/usr/bin/env node` 改成`#!/usr/bin/env nodejs` 再启动就好了 

---
## 包的安装
Bower是一个软件包管理器，所以你可以在应用程序中用它来安装新的软件包。举例来看一下来如何使用Bower安装JQuery，在你想要安装该包的地方创建一个新的文件夹，键入如下命令：
`bower install jquery`
由于没有权限 安装出现错误
```
$ bower install jquery
bower jquery#*              not-cached git://github.com/jquery/jquery.git#*
bower jquery#*                 resolve git://github.com/jquery/jquery.git#*
bower jquery#*                download https://github.com/jquery/jquery/archive/2.1.1.tar.gz
bower jquery#*                 extract archive.tar.gz
bower jquery#*                resolved git://github.com/jquery/jquery.git#2.1.1
bower                           EACCES EACCES, mkdir '/usr/bin/bower_components'

Stack trace:
Error: EACCES, mkdir '/usr/bin/bower_components'
....
```
然后使用sudo = =#
```
$ sudo bower install jquery
[sudo] password for ericwang: 
bower ESUDO         Cannot be run with sudo

Additional error details:
Since bower is a user command, there is no need to execute it with superuser permissions.
If you're having permission errors when using bower without sudo, please spend a few minutes learning more about how your system should work and make any necessary repairs.

http://www.joyent.com/blog/installing-node-and-npm
https://gist.github.com/isaacs/579814

You can however run a command with sudo using --allow-root option
```
提示就不翻译了,使用如下命令 安装成功
`$ bower --allow-root install jquery`
引起这个的原因是因为bower是在当前文件夹下创建bower_components文件夹,而bower是没有root权限的（我是因为改动/usr/local/bin/bower进入到/usr/local/bin/下的,所以运行命令时就会遇到权限问题），所以在root权限文件夹下使用该命令就会导致这种问题,如果你一定要在该文件夹下创建,可以使用--allow-root参数,或者修改文件夹的权限

---
## 包的使用
现在就可以在应用程序中使用jQuery包了，在jQuery里创建一个简单的html5文件：
```
<!doctype html>
<html>
<head>
    <title>Learning Bower</title>
</head>
<body>

<button>Animate Me!!</button>
<div style="background:red;height:100px;width:100px;position:absolute;">
</div>

<script type="text/javascript" src="bower_components/jquery/jquery.min.js"></script>
<script type="text/javascript">

    $(document).ready(function(){
        $("button").click(function(){
            $("div").animate({left:'250px'});
        });
    });
</script>
</body>
</html>
```
正如你所看到的，你刚刚引用jquery.min.js文件，现阶段完成。
## 其他命令
```
bower list //所有安装过的包的列表
      search bootstrap //假如你想在你的应用程序中使用twitter的bootstrap框架，但你不确定包的名字，这时你可以使用search 命令
      info bootstrap //如果你想看到关于特定的包的信息，可以使用info 命令来查看该包的所有信息 还可以这样
      uninstall jquery //卸载包可以使用uninstall 命令
      install //安装包  (bower install  bootstrap#2.1.1 --save 
                        安装指定版本,不过如果之前安装了某版本,会提示冲突,跟着提示走就可以了，bower.json也会被更新)
```
---
## bower.json文件的使用
bower.json文件的使用可以让包的安装更容易，你可以在应用程序的根目录下创建一个名为“bower.json”的文件，并定义它的依赖关系。使用bower init 命令来创建bower.json文件：
```
$ bower init
? name: bower-test
? version: 0.0.1
? description: Add bower.json
? main file: 
? what types of modules does this package expose?: 
? keywords: 
? authors: ericwang
? license: MIT
? homepage: 
? set currently installed components as dependencies?: Yes
? add commonly ignored files to ignore list?: Yes
? would you like to mark this package as private which prevents it from being accidentally published to the registry?: Yes

{
  name: 'bower-test',
  version: '0.0.1',
  description: 'Add bower.json',
  authors: [
    'ericwang'
  ],
  license: 'MIT',
  private: true,
  ignore: [
    '**/.*',
    'node_modules',
    'bower_components',
    'test',
    'tests'
  ],
  dependencies: {
    jquery: '~2.1.1'
  }
}

? Looks good?: Yes
```
可以查看该文件`view bower.json`
```
{
  "name": "bower-test",
  "version": "0.0.1",
  "description": "Add bower.json",
  "authors": [
    "ericwang"
  ],
  "license": "MIT",
  "private": true,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "jquery": "~2.1.1"
  }
}
```
已经加入了jQuery依赖关系
现在假设也想用twitter bootstrap，我们可以用下面的命令安装twitter bootstrap并更新bower.json文件：
`$ bower install bootstrap --save`
它会自动安装最新版本的bootstrap并更新bower.json文件
```
{
  "name": "bower-test",
  "version": "0.0.1",
  "description": "Add bower.json",
  "authors": [
    "ericwang"
  ],
  "license": "MIT",
  "private": true,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "jquery": "~2.1.1",
    "bootstrap": "~3.2.0"
  }
}
```
---
## 玩完了
原谅我这一生不羁放纵爱自由,上班时间写博客,罪过罪过。
用过java的maven管理,理解起来这个就相对简单了。
搭建依赖库时,最讨厌的就是到处找依赖包不是么，而有了这些工具之后,都省掉了。