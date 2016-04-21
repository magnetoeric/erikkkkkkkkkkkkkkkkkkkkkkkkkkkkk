title: git
tags:
  - git
category: linux
date: 2015-08-10 15:44:00

---
买个kindle,下了点书,上下班路上就可以看书了.来到ajk后,版本控制一直使用的是git

## Pro Git
![image](http://githubforericwang.qiniudn.com//hexo/eric/pro_git.jpg)
强烈推荐这本书,解释的很详尽透彻。附上[亚马逊](http://www.amazon.cn/Pro-Git-Chacon-Scott/dp/1430218339/ref=sr_1_2?ie=UTF8&qid=1412999771&sr=8-2&keywords=pro+git)地址

---
## Git 内部原理
直接跳到这里吧,git的使用实在不想多写了。
从根本上来讲 Git 是一套内容寻址 (content-addressable) 文件系统，在此之上提供了一个 VCS 用户界面。
![image](http://githubforericwang.qiniudn.com//hexo/eric/objects.jpg)
每个目录都创建了 tree对象 (包括根目录), 每个文件都创建了一个对应的 blob对象 . 最后有一个 commit对象 来指向根tree对象(root of trees), 这样我们就可以追踪项目每一项提交内容.

* blob 

存储的并不是文件名而仅仅是文件内容。这种对象类型称为 blob 。（git会为每一个改动后的文件生成一个blob,也就是一些文章中所说的快照）

* tree 

tree对象可以存储文件名，同时也允许存储一组文件。Git 以一种类似 UNIX 文件系统但更简单的方式来存储内容。所有内容以 tree 或 blob
对象存储，其中 tree 对象对应于 UNIX 中的目录，blob 对象则大致对应于 inodes 或文件内容。一个单独的 tree 对象包含一条或多条
tree 记录，每一条记录含有一个指向 blob 或子 tree 对象的 SHA-1 指针，并附有该对象的权限模式 (mode)、类型和文件名信息

* commit 

commit对象格式很简单：指明了该时间点项目快照的顶层树对象、作者/提交者信息（从 Git 设理发店的 user.name 和user.email中获得)
以及当前时间戳、一个空行，以及提交注释信息。

这三类 Git 对象 ── blob，tree 以及 tree ── 都各自以文件的方式保存在.git/objects 目录下。
可以在 objects 目录下看到这些文件。这便是 Git 存储数据内容的方式──为每份内容生成一个文件，取得该内容与头信息的 SHA-1 校验和，创建以该校验和前2个字符为名称的子目录，并以 (校验和) 剩下 38 个字符为文件命名 (保存至子目录下)。

## 示例
下面来尝试解释一下git的运行吧
```
$ git init #初始化git
$ ls -al #可以看到当前目录下产生了一个.git目录
```
看一下.git的目录结构
git 的本地化配置(git config --local 产生的配置会写到.git/config 
.git/objects 里面会存储SHA-1码的一些东西
.git/HEAD 存储的是当前工作中的分支指针
```
$ find .git/ 
.git/
.git/hooks
.git/hooks/post-update.sample
.git/hooks/pre-rebase.sample
.git/hooks/pre-applypatch.sample
.git/hooks/commit-msg.sample
.git/hooks/applypatch-msg.sample
.git/hooks/pre-push.sample
.git/hooks/pre-commit.sample
.git/hooks/prepare-commit-msg.sample
.git/hooks/update.sample
.git/config 
.git/description
.git/objects 
.git/objects/info
.git/objects/pack 
.git/info
.git/info/exclude
.git/refs
.git/refs/heads
.git/refs/tags
.git/branches
.git/HEAD 
```
创建README并写入hello world
```
$echo 'hello word' > README.md
```
添加到暂存区
```
$ git add README.md
$ git commit -m 'first commit'
[master （根提交） d2a03d1] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

```
查看暂存区中的提交
```
$git log
commit d2a03d1cece536133c49d3ea8f6c503c778307f7
Author: ericwang <orange.king@qq.com>
Date:   Sat Oct 11 18:31:36 2014 +0800

    first commit
```
查看.git/objects
```
$ find .git/objects/
.git/objects/
.git/objects/73
.git/objects/73/928f2dd793cbc72fcc2b1ba7d0585bfa91886a
.git/objects/info
.git/objects/pack
.git/objects/7f
.git/objects/7f/d5222177e8ffadda6437dc9cfa0630a2777673
.git/objects/d2
.git/objects/d2/a03d1cece536133c49d3ea8f6c503c778307f7
```
生成了3个文件
* .git/objects/d2/a03d1cece536133c49d3ea8f6c503c778307f7 
* .git/objects/7f/d5222177e8ffadda6437dc9cfa0630a2777673
* .git/objects/73/928f2dd793cbc72fcc2b1ba7d0585bfa91886a 

可以看到 commit d2a03d1cece536133c49d3ea8f6c503c778307f7 对应的是.git/objects/d2/a03d1cece536133c49d3ea8f6c503c778307f7文件
其SHA-1码被拆分成2部分,第一部分是SHA-1码的前两位作为目录名,第二部分是剩下的38位作为文件名

查看commit对象
```
$ git cat-file  -p d2a03d1cece536133c49d3ea8f6c503c778307f7
tree 73928f2dd793cbc72fcc2b1ba7d0585bfa91886a
author ericwang <orange.king@qq.com> 1413023496 +0800
committer ericwang <orange.king@qq.com> 1413023496 +0800

first commit
```
tree对象的SHA-1码为 73928f2dd793cbc72fcc2b1ba7d0585bfa91886a
对应的是 .git/objects/73/928f2dd793cbc72fcc2b1ba7d0585bfa91886a 
继续查看tree对象
```
$ git cat-file -p 73928f2dd793cbc72fcc2b1ba7d0585bfa91886a
100644 blob 7fd5222177e8ffadda6437dc9cfa0630a2777673    README.md
```
blob对象的SHA-1码为7fd5222177e8ffadda6437dc9cfa0630a2777673
对应的是.git/objects/7f/d5222177e8ffadda6437dc9cfa0630a2777673文件
查看blob对象
```
$ git cat-file -p 7fd5222177e8ffadda6437dc9cfa0630a2777673
hello word
```
其存储的是就是我们最初写入的文本内容
下面继续向README.md中写入内容
```
$ echo 2 >>README.md
$ git add .
$ git commit -m 'second commit'
[master 897276c] second commit
 1 file changed, 1 insertion(+)
```
生成了新的commit 对象 897276c61b6e00005f66a6b16477e872835b5408
查看新的commit对象
```
$ git cat-file -p 897276c61b6e00005f66a6b16477e872835b5408
tree 6f44b87201208be6247b73708470d2b6266b7063
parent d2a03d1cece536133c49d3ea8f6c503c778307f7
author ericwang <orange.king@qq.com> 1413027193 +0800
committer ericwang <orange.king@qq.com> 1413027193 +0800

second commit
```
其父commit是我们第一次的commit,tree对象是6f44b87201208be6247b73708470d2b6266b7063,
查看tree对象
```
$ git cat-file -p 6f44b87201208be6247b73708470d2b6266b7063
100644 blob d0a2ec18682f4d54a2507c0d0275248dc2d8dad2    README.md
```
查看blob对象
```
$ git cat-file -p d0a2ec18682f4d54a2507c0d0275248dc2d8dad2
hello word
2
```
可以看到,git为改动后的README.md生成了新的blob,区别于第一次提交产生的blob
查看.git/objects/下也可以看到，git为改动后的该文件新生成了1个commit,1个tree,1个blob
```
$ find .git/objects/
.git/objects/
.git/objects/73
.git/objects/73/928f2dd793cbc72fcc2b1ba7d0585bfa91886a
.git/objects/6f
.git/objects/6f/44b87201208be6247b73708470d2b6266b7063
.git/objects/89
.git/objects/89/7276c61b6e00005f66a6b16477e872835b5408
.git/objects/info
.git/objects/pack
.git/objects/d0
.git/objects/d0/a2ec18682f4d54a2507c0d0275248dc2d8dad2
.git/objects/7f
.git/objects/7f/d5222177e8ffadda6437dc9cfa0630a2777673
.git/objects/d2
.git/objects/d2/a03d1cece536133c49d3ea8f6c503c778307f7
```
接着 新建文件并写入内容
```
$ echo 'new file content' >newfile
$ git add .
$ git commit -m 'third commit'
[master e1107b4] third commit
 1 file changed, 1 insertion(+)
 create mode 100644 newfile
$ git cat-file -p e1107b460be89af8d508ac972270237d2f1c530f
tree 4c4d45c5a7485c475fd8fd1b53d26fddcf18f0cd
parent 897276c61b6e00005f66a6b16477e872835b5408
author ericwang <orange.king@qq.com> 1413028137 +0800
committer ericwang <orange.king@qq.com> 1413028137 +0800

third commit
$ git cat-file -p 4c4d45c5a7485c475fd8fd1b53d26fddcf18f0cd
100644 blob d0a2ec18682f4d54a2507c0d0275248dc2d8dad2    README.md
100644 blob 8e66654a5477b1bf4765946147c49509a431f963    newfile
```
第三次提交中对应的tree对象有两个blob,我们没有对README.md进行修改,所以git并没有为它新生成一个blob,而是继续使用原有的blob
画个图比较直观一些，画风不怎么样,哈哈
![image](http://githubforericwang.qiniudn.com//hexo/eric/commits.png)
实际上,项目中的git对象关系远比这复杂的多,很多tree对象有很多子tree,子tree中又包含很多blob,blob存储的是某次文件修改生成的快照


  [1]: http://magnetoeric-typechoupload.stor.sinaapp.com/2779866217.jpg
  [2]: http://magnetoeric-typechoupload.stor.sinaapp.com/3900410026.jpg
  [3]: http://magnetoeric-typechoupload.stor.sinaapp.com/562266015.png