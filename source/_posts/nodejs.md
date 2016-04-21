title: 一个快速的mock服务器
tags:
  - nodejs
category: nodejs
date: 2015-08-28 20:03:00

---

最近做代码重构迁移
在为app提供接口开发时，常常是定好接口，然后为其提供fake api。
然后在java开发时，写接口需要定义好dto再返回，在开发过程中可能还会改变，更重要的是，需要机器去为app提供接口访问。
显然浪费了很多时间和精力
github上一个很火的[项目](https://github.com/typicode/json-server)
Get a full fake REST API with zero coding in less than 30 seconds (seriously)
正如描述的，可以提供快速的fake api构建。
然而使用普通的`json-server --watch db.json`方式启动并不能满足需求，项目的api是动态增加的，不可能每次都去修改db.json然后再重启，格式乱掉也会是个问题。
然后想到的是使用`json-server index.js` 在启动时把接口文件载入到内存中，提供给app使用
约定加载api文件夹下所有的文件，每个文件使用json格式保存
因为访问路径时使用的是／，所以考虑把api文件命名规则定为xx.x.xxx代替xx/x/xxx，这样访问路径就可以确定下来了
下面的代码中，我把文件夹名称改掉了
```
module.exports=function(){
    var data = [];
    var dirName = "./xxx/"
    var rf=require("fs");
    var files = rf.readdirSync(dirName);
    files.forEach(function(file){
        var content = rf.readFileSync(dirName+file);
        var ob  = JSON.parse(content);
        var arr = file.split(".");
        var path = arr.join("/");
        data[path] = ob;
    });
    return data;
}
```

写段nodejs真是不容易。。。开始readfile使用的是异步方式，然后服务器启动成功，但是没有任何api被加载，以为是闭包中全局变量的问题，查下来才发现是异步执行的。
接下来的问题不能在后台挂起，问了作者才知道，需要使用[hotel](https://github.com/typicode/hotel)做damon process,问题总算是解决了
接下来写了个shell，执行shell可以从远程git上把api拉下来，然后在server.js中执行shell来更新文件。
然后发现nodejs execfile也是异步的，真是xxxx。换了种思路，直接在shell中更新api数据，然后启动json-server。
```
#!/bin/bash
git fetch origin
diff=$(git diff origin/master --shortstat);
if [ -n "$diff" ]
then
 git rebase origin master
fi
echo "starting json-server";
```
```
hotel add 'cmd -p ${port}'
```
就可以在web中启动和关闭fake server实现代码更新了
接下来的问题是这个fake server的问题是无法处理post请求，json－server原本的post请求实际上是为了向‘服务器’添加数据。看了下文档，好像也没办法解决，只能在nginx里处理了
