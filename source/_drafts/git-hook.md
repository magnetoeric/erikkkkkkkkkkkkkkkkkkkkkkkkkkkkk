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

### post-update 
在服务器执行

### prepare-commit-msg
效果与pre-commit类似，先于pre-commit执行

## 参考
   [1] https://www.kernel.org/pub/software/scm/git/docs/githooks.html     
   [2]https://www.atlassian.com/git/tutorials/git-hooks/local-hooks