title: varnish 
tags:
  - varnish
category: http
date: 2015-07-17 11:45:00

---
## 关于Varnish
* 高性能的开源HTTP加速器
* 基于内存进行缓存，重启后数据将消失
* 利用虚拟内存方式，I/O性能好
* 配置语言VCL(Varnish Configuration Language) 灵活管理配置.
    在程序启动时，varnish就把VCL转换成二进制代码，因此性能非常高
    
---
## 安装
略。注意安装依赖，有些命令需要安装依赖   

---
## varnish 启动
* 一个启动的例子
```
         sudo /usr/local/varnish/sbin/varnishd \
         -u www -g www \
         -f /usr/local/varnish/etc/varnish/default.vcl \
         -a 127.0.0.1:8080 \
         -s file,/var/www/varnish_cache.data,1G \
         -w 1024,2048,10 \
         -t 3600 \
         -T 127.0.0.1:2000 
```
* 说明
```
         u 用户名
         g 所属组
         f 配置文件地址
         a 绑定地址及端口
         s 缓存文件地址及大小
         w 最大最小线程 及超时时间
         T 管理端口  主要用来清除缓存
         t 默认缓存时间
```
* 结束 
      `sudo pkill varnishd`
   
---
## 管理命令 varnishadm

```
     /usr/local/varnish/bin/varnishadm -T 127.0.0.1:2000 -S /etc/varnish/secret help 查看可用的命令列表 -S 参数为指定密码文件
     vcl.load default.vcl /etc/varnish/default.vcl 载入新vcl配置文件
     vcl.list 查看已有的配置列表
     vcl.use default.vcl 使用加载的配置文件
     ban.url ^hello.php$ 清除hello.php的缓存
     start 启动
     stop 关闭 
```
---

## 其他命令

```
     varnishlog 日志
     varnishstat 状态 ：命中等
          
```

---

## VCL  
1.三个重要的数据结构  
> * req 
    请求头信息  当varnish接收到请求的时候 req会被创建
> * beresp
    后端返回对象  包含从backend返回的头信息 
> * obj
    缓存了的对象。大多数是驻留在内存中的只读对象。obj.ttl是可以写的，剩下的都是只读的

更多变量点[这里][2]

2.子程序
VCL 文件被分为多个子程序 常用子程序
> * vcl_recv 当有访问时，varnish会执行该子程序，该程序通过对req的一些参数进行处理，决定接下来的动作 这里只能访问到req数据结构
     一般以以下3种动作结束
     * pass：执行pass动作,把请求交给vcl_pass模块处理。
     * pipe：执行pipe动作，把请求交给vcl_pipe模块处理。
     * error code [reason]：表示把错误标识返回给客户端，并放弃处理该请求。错误标识包括200、405等。"reason"是对错误的提示信息。
> * vcl_fetch  当varnish成功从后端返回数据时,执行该子程序 这里既能访问req 也能访问beresp
> * vcl_pipe 执行pipe动作时调用,用于将请求直接传递至后端主机，在请求和返回的内容没有改变的情况下，
    也就是在当前连接未关闭时，服务器将不变的内容返回给客户端，直到该连接被关闭
> * lookup 一个请求在vcl_recv中被lookup后，Varnish将在缓存中提取数据。
    如果缓存中有相应的数据，就把控制权交给vcl_hit子程序；
    如果缓存中没有相应的数据，请求将被设置为pass并将其交给vcl_miss子程序

varnish 将在不同阶段执行它的子程序代码，因为它的代码是一行一行执行的，不存在优先级问题。随时可以调用这个子程序中的功能并且当他执行完成后就退出。

3.动作(action)：
 > *  pass： 当一个请求被 pass 后，这个请求将通过 varnish 转发到后端服务器，但是它不会被缓存。pass 可以放在 vcl_recv 和 vcl_fetch 中。
 > *  lookup： 当一个请求在 vcl_recv 中被 lookup 后，varnish 将从缓存中提取数据，如果缓存中没有数据，将被设置为 pass，不能在 vcl_fetch 中设置 lookup。
 > *  pipe： pipe 和 pass 相似，都要访问后端服务器，不过当进入 pipe     模式后，在此连接未关闭前，后续的所有请求都发到后端服务器，而pass只对当前的传输有效，后续的请求会根据策略决定是否发送到客户端
 > *  deliver： 请求的目标被缓存，然后发送给客户端。
 > *  esi： ESI-process the fetched document (Edge Side Includes ，使用varnish+esi 
     可以对用户个性化相关的进行缓存，而我们通常的做法是使用ajax+memcache对用户个性化的内容进行缓存 [官方文档][3]）

---

## 处理流程
  ![image](http://githubforericwang.qiniudn.com/hexo/eric/process.jpg)

---
## http头信息
HTTP协议定义了四个可以用来控制浏览器缓存的HTTP头，它们是：
> * Last-Modified
> * Expires
> * Pragma: no-cache
> * Cache-Control

在HTTP/1.0协议中:
> * Last-Modified是控制缓存的一个非常重要的HTTP头。
    如果需要控制浏览器的缓存，服务器首先必须发送一个 以UTC时间为值的Last-Modifeid头，
    当第二次访问这个页面时，浏览器会发送一个If-Modified-Since头给服务器，让服务器判断是否有必要更新内容，
    这个If-Modified-Since头的值就是上次访问页面时，浏览器发送的Last-Modifeid头的值。
> * Expires是HTTP/1.0中另外一个很重要的HTTP头，它表示缓存的存在时间，
    告诉客户端浏览器在这个时间之前不对服务器发送请求，而直接使用浏览器的缓存。
    在HTTP/1.0中，可以使用Pragma: no-cache头来告诉浏览器不要缓存内容，
    它相当于HTTP/1.1中的Cache-Control:no-cache

HTTP/1.0协议的这种实现方式的缺点是，服务器和客户端的时间有可能是不同步的，
这样会造成缓存的实现达不到预期效果。HTTP/1.1协议用Cache-Control头解决了这个问题

在http 1.1中
Cache-Control响应头的语法为：Cache-Control = “Cache-Control” “:”{缓存响应指令}
缓存响应指令为一下几个中的任意一个：
 * public 指示响应数据可以被任何客户端缓存

 * private 指示响应数据可以被非共享缓存所缓存。这表明响应的数据可以被发送请求的浏览器缓存，而不能被中介所缓存

 * no-cache  指示响应数据不能被任何接受响应的客户端所缓存

 * no-store 指示所传送的响应数据除了不能被缓存，也不能存入磁盘。一般用于敏感数据，以免数据被复制。

 * must-revalidate 指示所有的缓存都必须重新验证，
   在这个过程中，浏览器会发送一个If-Modified-Since头。 
   如果服务器程序验证得出当前的响应数据为最新的数 据，那么服务器应当返回一个304

 * proxy-revalidate 与must-revalidate相似，不同的是用来指示共享缓存

 * max-age=时间  数据经过max-age设置的秒数后就会失效，
    相当于HTTP/1.0中的Expires头。
    如果在一次响应中同时设置了max-age和Expires，那么max-age将具有较高的优先级

 * s-maxage=时间 与max-age相似，不同的是用来指示共享缓存。
   注：通过POST方法发送的请求不能以如上所述的方式缓存

可以在使用浏览器发送如下信息头
> * 当我们在浏览器中输入url网址时，如果有过该网址的访问记录，浏览器会判断 max-age，如果没有会发送If-Modified-Since头信息，服务器去处理是否有更新
    有些内容有max-age，浏览器不会发送请求，使用浏览器缓存,Chrome返回的状态码是200 OK (from cache)，firefox没有请求。

> * 当我们按下F5时,浏览器会发送If-Modified-Since，max-age=0 头信息 这样服务器就会返回其最新内容(如果有varnish缓存的话,只返回varnish缓存)

> * 当我们按下Control + F5，强刷，浏览器会发送 Pragma: no-cache，Cache-Control: no-cache信息头 如果在vcl中对这类信息头做处理，就可以刷新varnish缓存
    这在生产环境中很适用，但是在线上环境中所有人都可以通过强制刷新varnish.所以需要使用白名单允许一些ip可以清除缓存,通常Cache-Control:nocache 是被varnish忽略的,根据自身需要处理

当然也可以写代码发送请求，或是使用浏览器插件比如livehttpheader或者使用curl命令

---

## 清除缓存
 1.这两个规则定义了允许哪些主机通过HTTP来执行PURGE进行缓存删除。如果不是指定的IP，就会出现HTTP 405错误，提示Not allowed错误字样
```
     acl purge {  
           "localhost";  
           "127.0.0.1";  
        }  
    if (req.request == "PURGE") {  
                   if (!client.ip ～ purge) {  
                           error 405 "Not allowed.";  
                   }  
                   return(lookup);  
     }  
```
 2.varnishadm ban.url [regexp] 不需要加域名

## 注意事项
* 如果backend返回的头信息中包含setcookie,varnish不会对此缓存，一般会在vcl中unset掉backend header产生的cookie [更多请看这里][4]
    
    php中有两种set cookie方法：
    一种是header(string,replace,http_response_code),
    一种是setCookie(name,value,expire,path,domain,secure)
    setCookie设置的cookie,Varnish会缓存吗?（其实是一样的）

* 如果varnish看到授权头信息(Authorization)时，它会pass该请求。可以unset这个头信息

* 当几个客户端正访问相同页面时，varnish会发送一个请求到后端，然后让那个其他几个请求挂起等待返回结果，
    返回结果后，复制请求的结果发送给客户端并且让其他请求等待,  
    但是如果，每秒上千的请求同时呢?（这个队列很庞大），这时候我们期待的应该是，返回过期的cache给用户
    为了使用过期的cache给用户提供服务，我们需要增加他们的TTL，保存所有cache中的内容在TTL过期以后30分钟不删除，
    使用以下VCL：
    ```
    sub vcl_fetch {
      set beresp.grace = 30m;
     }
    ```
    Varnish还不会使用过期的目标给用户提供服务，所以我们需要配置以下代码，在cache过期后的15秒内，使用旧的内容提供服务：
    ```
    sub vcl_recv {
        set req.grace = 15s;
    }
    ```

* 如果后端出了问题，是不是不应该返回问题的页面，而应该返回就版本的正常页面，可以开启健康检查，如果后端出问题了，可以长时间提供旧版本的内容
    ```
    if (! req.backend.healthy) {
       set req.grace = 5m;
    } else {
       set req.grace = 15s;
    }
    ```
---

## 总结
说一下我遇到的一些坑吧,毕竟不是运维人员,很多监控还不是很懂，所以只是说一些代码层面上的。
* 一般来说varnish是要将后端返回的header头的cookie信息重置掉,这样可以高效地利用缓存。
 这种方式带来的后果就是：
 由于Varnish 是一个HTTP缓存服务器,也就是说它是直接和浏览器打交道的,更通俗点,Varnish缓存的内容是直接被浏览器解析的!
 所以 不要在后端服务器中返回带有状态的代码！Varnish对于所有用户访问的同一链接,返回的都是同一个内容。
 如果你在后端返回的信息存在cookie,并且在Varnish中将cookie干掉了,就会导致,一个用户的行为被Varnish缓存住了,其他用户也被动地进行了这个行为。
 通常解决这种方式的做法是使用ajax。

* varnish 配置文件是允许重复配置同一个动作的,但是只是先加载的有效= =#。所以注意这个坑(已测试)

* 由于我们的业务需要,测试环境是穿透varnish的，也就是白名单,所以很多时候,内网会泄漏掉varnish的很多东西
  这时候需要使用一些外网IP，挂代理取检测网站内容了（不过要小心,外网代理的策略有可能是把整个网页缓存了）
* 由于Varnish会对每一个url地址做缓存,为了不必要的浪费,一般是对某个url做截断处理
(比如:http:www.baidu.com/?a=1&b=1 和http:www.baidu.com/ 缓存的应该是同一个副本)

---

## 参考文献
> * http://anykoro.sinaapp.com/2012/01/31/varnish3-0中文入门教程/
> * https://www.varnish-cache.org/docs/3.0/
> * https://www.varnish-software.com/static/book/HTTP.html


以前分享的文章， 放到博客里吧


  [1]: http://magnetoeric-typechoupload.stor.sinaapp.com/540940703.jpg
  [2]: https://www.varnish-cache.org/docs/3.0/reference/vcl.html#variables
  [3]: https://www.varnish-cache.org/trac/wiki/VCLExampleCachingLoggedInUsers
  [4]: https://www.varnish-cache.org/docs/3.0/tutorial/increasing_your_hitrate.html#tutorial-increasing-your-hitrate