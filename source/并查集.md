# 数据结构之并查集
文章转自[数据结构之并查集](http://dongxicheng.org/structure/union-find-set/);

## 概述
并查集（Disjoint set或者Union-find set）是一种树型的数据结构，常用于处理一些不相交集合（Disjoint Sets）的合并及查询问题。

##  基本操作
并查集是一种非常简单的数据结构，它主要涉及两个基本操作，分别为：

* 合并两个不相交集合

* 判断两个元素是否属于同一个集合

1.合并两个不相交集合（Union(x,y)）

合并操作很简单：先设置一个数组Father[x]，表示x的“父亲”的编号。那么，合并两个不相交集合的方法就是，找到其中一个集合最父亲的父亲（也就是最久远的祖先），将另外一个集合的最久远的祖先的父亲指向它。
![union](http://githubforericwang.qiniudn.com/hexo/eric/union.jpg)

上图为两个不相交集合，b图为合并后Father(b):=Father(g)

2.判断两个元素是否属于同一集合（Find_Set(x)）

本操作可转换为寻找两个元素的最久远祖先是否相同。可以采用递归实现。

## 优化

1.Find_Set(x)时，路径压缩

寻找祖先时，我们一般采用递归查找，但是当元素很多亦或是整棵树变为一条链时，每次Find_Set(x)都是O(n)的复杂度。为了避免这种情况，我们需对路径进行压缩，即当我们经过”递推”找到祖先节点后，”回溯”的时候顺便将它的子孙节点都直接指向祖先，这样以后再次Find_Set(x)时复杂度就变成O(1)了，如下图所示。可见，路径压缩方便了以后的查找。

![](http://githubforericwang.qiniudn.com/hexo/eric/optimization.jpg)

2.Union(x,y)时，按秩合并

即合并的时候将元素少的集合合并到元素多的集合中，这样合并之后树的高度会相对较小。

```
int father[MAX];   /* father[x]表示x的父节点*/
 
int rank[MAX];     /*rank[x]表示x的秩*/
 
&nbsp;
 
void Make_Set(int x)
 
{
 
father[x] = x; //根据实际情况指定的父节点可变化
 
rank[x] = 0;   //根据实际情况初始化秩也有所变化
 
}
 
/* 查找x元素所在的集合,回溯时压缩路径*/
 
int Find_Set(int x)
 
{
 
if (x != father[x])
 
{
 
father[x] = Find_Set(father[x]); //这个回溯时的压缩路径是精华
 
}
 
return father[x];
 
}
 
/*
 
按秩合并x,y所在的集合
 
下面的那个if else结构不是绝对的，具体根据情况变化
 
但是，宗旨是不变的即，按秩合并，实时更新秩。
 
*/
 
void Union(int x, int y)
 
{
 
x = Find_Set(x);
 
y = Find_Set(y);
 
if (x == y) return;
 
if (rank[x] > rank[y])
 
{
 
father[y] = x;
 
}
 
else
 
{
 
if (rank[x] == rank[y])
 
{
 
rank[y]++;
 
}
 
father[x] = y;
 
}
 
}
```

## 复杂度分析

空间复杂度为O(N)，建立一个集合的时间复杂度为O(1)，N次合并M查找的时间复杂度为O(M Alpha(N))，这里Alpha是Ackerman函数的某个反函数，在很大的范围内（人类目前观测到的宇宙范围估算有10的80次方个原子，这小于前面所说的范围）这个函数的值可以看成是不大于4的，所以并查集的操作可以看作是线性的。具体复杂度分析过程见参考资料（3）。

## 应用

并查集常作为另一种复杂的数据结构或者算法的存储结构。常见的应用有：求无向图的连通分量个数，最近公共祖先（LCA），带限制的作业排序，实现Kruskar算法求最小生成树等。

## 参考资料

* 并查集：http://www.nocow.cn/index.php/%E5%B9%B6%E6%9F%A5%E9%9B%86

* 博文《并查集详解》：http://www.cnblogs.com/cherish_yimi/

* Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, and Clifford Stein. Introduction to Algorithms, Second Edition. MIT Press and McGraw-Hill, 2001. ISBN 0-262-03293-7. Chapter 21: Data structures for Disjoint Sets, pp. 498–524.