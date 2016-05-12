title: git hook
tags:
  - git
categories:
  - git
date: 2016-04-20 16:13:00
---
## hook
使用了很久git了，但是hook用的很少，今天研究一下
git的hook放在GIT_DIR/.git/hooks目录下
```
➜  hooks git:(master) ls -al
总用量 48
drwxrwxr-x 2 eric eric 4096  4月 20 16:18 .
drwxrwxr-x 7 eric eric 4096  4月 20 16:18 ..
-rwxrwxr-x 1 eric eric  452  4月 20 16:18 applypatch-msg.sample
-rwxrwxr-x 1 eric eric  896  4月 20 16:18 commit-msg.sample
-rwxrwxr-x 1 eric eric  189  4月 20 16:18 post-update.sample
-rwxrwxr-x 1 eric eric  398  4月 20 16:18 pre-applypatch.sample
-rwxrwxr-x 1 eric eric 1642  4月 20 16:18 pre-commit.sample
-rwxrwxr-x 1 eric eric 1239  4月 20 16:18 prepare-commit-msg.sample
-rwxrwxr-x 1 eric eric 1352  4月 20 16:18 pre-push.sample
-rwxrwxr-x 1 eric eric 4898  4月 20 16:18 pre-rebase.sample
-rwxrwxr-x 1 eric eric 3611  4月 20 16:18 update.sample
```
可以看到该目录下有很多以sample结尾的文件
这里尝试几个简单的吧


## 客户端钩子
### commit-msg
将commit-msg.sample 重命名commit-msg
```
mv commit-msg.sample commit-msg
➜  hooks git:(master) cat commit-msg 
#!/bin/sh
#
# An example hook script to check the commit log message.
# Called by "git commit" with one argument, the name of the file
# that has the commit message.  The hook should exit with non-zero
# status after issuing an appropriate message if it wants to stop the
# commit.  The hook is allowed to edit the commit message file.
#
# To enable this hook, rename this file to "commit-msg".

# Uncomment the below to add a Signed-off-by line to the message.
# Doing this in a hook is a bad idea in general, but the prepare-commit-msg
# hook is more suited to it.
#
# SOB=$(git var GIT_AUTHOR_IDENT | sed -n 's/^\(.*>\).*$/Signed-off-by: \1/p')
# grep -qs "^$SOB" "$1" || echo "$SOB" >> "$1"

# This example catches duplicate Signed-off-by lines.

test "" = "$(grep '^Signed-off-by: ' "$1" |
	 sort | uniq -c | sed -e '/^[ 	]*1[ 	]/d')" || {
	echo >&2 Duplicate Signed-off-by lines.
	exit 1
}

```
看介绍，应该是在git commit 命令时检查log message的脚本，有一个参数，存放commit message的文件名
在最后加上判断log message低于3个时不得提交的脚本
```
words=$(cat $1 |wc -w)
if test $words -lt 3
 then  echo "commit message should larger than 3 words"
       exit 1
fi
```
然后添加一个文件,提交。无法提交，原因是message单词数没有超过3个，过于简单
```
➜  git-hooks git:(master) ✗ echo 1111>aaa
➜  git-hooks git:(master) ✗ git add .
➜  git-hooks git:(master) ✗ git commit -m 'add aaa'
commit message should larger than 3 words

```

### pre-commit
git commit 触发  先于commit-msg，不接收参数


### prepare-commit-msg
效果与pre-commit类似，先于pre-commit执行


## 服务器钩子
### pre-receive
在客户端推送时最先执行，可以用它来拒绝客户端的推送。
我能想到的，可以做语法检查，拼写等等
### update
与 pre-receive 类似，但会在每个分支都执行一次。

### post-receive
在客户端推送完成后执行

## 使用gitlab的webhook
现在很多版本控制都是使用git，gitlab来管理git项目仓库，其提供了方便的web管理，用户访问及权限，是搭建私人git仓库的推荐选择。
具体安装可以参考官方的wiki，因为gitlab比较吃内存，，所以我不想把它安装到机器上，所以就用docker来操作吧。
创建一个名为gitlab的容器，并将其80端口映射到苏主机的8088端口
```
docker run -itd -p 8088:80 --name gitlab gitlab/gitlab-ce
```
等待一段时间，就可以访问本机的8088端口，进入到gitlab的web管理界面了，初次使用要设置root的密码，然后就可以登录了，默认管理员帐号是root，密码是初次进入时设置的。
登录后点击 new project创建一个git项目,就叫做testHook把
接下来再启动一个docker 容器来作为git的客户端
```
docker run -it --link=gitlab ubuntu:14.04 /bin/bash
```
启动后安装git
```
root@ed6dd236d56b:~# apt-get update && apt-get install git
```
生成sshkey
```
ssh-keygen 
```
一直回车就可以了，在~/.ssh/下就生成了2个文件，私钥和公钥
```
cat ~/.ssh/id_rsa.pub
```
把公钥设置到gitlab里，搜索框里搜索sshkey，把公钥复制到key中，title可以随意设置，gitlab也会自动生成
接下来把git项目clone到客户端中
```
root@ed6dd236d56b:~#git clone git@b9a4402e85aa:root/testhook.git
```
home目录下就会有一个空的git项目testHook，

接下来设置webhook,可以为每个项目设置webhook，比如我创建的这个git项目，在http://localhost:8088/root/testhook/hooks 中设置
这里我只添加push event，这个钩子作用应该类似post-receive，url是钩子触发时，需要向“谁”报告,这里我只在本机上启动了一个swoole-server,url请求swoole来看看gitlab钩子触发时具体发了些什么信息
swoole的代码就直接粘贴到这里了
```
<?php
// Server
class Server
{
    private $serv;

    public function __construct() {
        $this->serv = new swoole_server("0.0.0.0", 9501);
        $this->serv->set(array(
            'worker_num' => 8,
            'daemonize' => false,
            'max_request' => 10000,
            'dispatch_mode' => 2,
            'debug_mode'=> 1
        ));

        $this->serv->on('Start', array($this, 'onStart'));
        $this->serv->on('Connect', array($this, 'onConnect'));
        $this->serv->on('Receive', array($this, 'onReceive'));
        $this->serv->on('Close', array($this, 'onClose'));

        $this->serv->start();
    }

    public function onStart( $serv ) {
        echo "Start\n";
    }

    public function onConnect( $serv, $fd, $from_id ) {
        $serv->send( $fd, "Hello {$fd}!,this is swoole server " );
    }

    public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
        echo "Get Message From Client {$fd}:{$data}\n";
    }

    public function onClose( $serv, $fd, $from_id ) {
        echo "Client {$fd} close connection\n";
    }
}
// 启动服务器
$server = new Server();
```

启动swooleserver

```
php swoole-server.php
```

然后来操作下客户端，需要设置git的user.name 和user.email不然不能commit
```
root@ed6dd236d56b:~/testhook# echo 1>test
root@ed6dd236d56b:~/testhook# git add test
root@ed6dd236d56b:~/testhook# git commit -m 'add test'
[master 7923a94] add test
1 file changed, 1 insertion(+)
create mode 100644 test
```

接下来push，让gitlab产生一个触发
```
root@ed6dd236d56b:~/testhook# git push origin master
Counting objects: 4, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 256 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@b9a4402e85aa:root/testhook.git
79a7567..7923a94  master -> master
```
可以看到swoole产生的日志
```
➜  php swoole-server.php
Start
Get Message From Client 1:POST /swoole/swoole-client.php HTTP/1.1
Content-Type: application/json
X-Gitlab-Event: Push Hook
Connection: close
Host: 10.207.26.234:9501
Content-Length: 1532


Get Message From Client 1:{"object_kind":"push","before":"79a7567c469738a689d38510f9cfc6b5132eda02","after":"7923a944b9c2451be90d8f219fba6f0732c958ba","ref":"refs/heads/master","checkout_sha":"7923a944b9c2451be90d8f219fba6f0732c958ba","message":null,"user_id":1,"user_name":"Administrator","user_email":"admin@example.com","user_avatar":"http://www.gravatar.com/avatar/e64c7d89f26bd1972efa854d13d7dd61?s=80\u0026d=identicon","project_id":1,"project":{"name":"testhook","description":"","web_url":"http://b9a4402e85aa/root/testhook","avatar_url":null,"git_ssh_url":"git@b9a4402e85aa:root/testhook.git","git_http_url":"http://b9a4402e85aa/root/testhook.git","namespace":"root","visibility_level":20,"path_with_namespace":"root/testhook","default_branch":"master","homepage":"http://b9a4402e85aa/root/testhook","url":"git@b9a4402e85aa:root/testhook.git","ssh_url":"git@b9a4402e85aa:root/testhook.git","http_url":"http://b9a4402e85aa/root/testhook.git"},"commits":[{"id":"7923a944b9c2451be90d8f219fba6f0732c958ba","message":"add test\n","timestamp":"2016-05-12T03:36:03+00:00","url":"http://b9a4402e85aa/root/testhook/commit/7923a944b9c2451be90d8f219fba6f0732c958ba","author":{"name":"ericwang","email":"ericwang@leju.com"},"added":["test"],"modified":[],"removed":[]}],"total_commits_count":1,"repository":{"name":"testhook","url":"git@b9a4402e85aa:root/testhook.git","description":"","homepage":"http://b9a4402e85aa/root/testhook","git_http_url":"http://b9a4402e85aa/root/testhook.git","git_ssh_url":"git@b9a4402e85aa:root/testhook.git","visibility_level":20}}
```
关于webhook具体的细节，在gitlab搜索框中搜索webhook也可以查阅


## 参考
   [1] https://www.kernel.org/pub/software/scm/git/docs/githooks.html     
   [2] https://www.atlassian.com/git/tutorials/git-hooks/local-hooks
   [3] http://www.ituring.com.cn/article/206985