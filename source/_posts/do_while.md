title: do{...}while(0)的意义和用法
tags:
  - C
category: C
date: 2016-01-04 21:23:35

---
在尝试写扩展时，阅读到了这样类似的代码
```
#define ZVAL_BOOL(z, b) do {		\
		zval *__z = (z);			\
		Z_LVAL_P(__z) = ((b) != 0);	\
		Z_TYPE_P(__z) = IS_BOOL;	\
	} while (0)
```
好奇怪，明明是只做一次运算，为何要使用do{}while(0)的方式
google一下，涨了很多姿势
接下来的大部分部分参考[do{...}while(0)的意义和用法](http://www.spongeliu.com/415.html)

linux内核和其他一些开源的代码中，经常会遇到这样的代码：
```
do{
 ...
}while(0)
```
总结起来这样写主要有以下几点好处：

1、辅助定义复杂的宏，避免引用的时候出错：

举例来说，假设你需要定义这样一个宏：
```
#define DOSOMETHING()\
               foo1();\
               foo2()
```
这个宏的本意是，当调用DOSOMETHING()时，函数foo1()和foo2()都会被调用。但是如果你在调用的时候这么写：
```
if(a>0)
    DOSOMETHING();
```
因为宏在预处理的时候会直接被展开，你实际上写的代码是这个样子的：
```
if(a>0)
    foo1();
foo2();
```
这就出现了问题，因为无论a是否大于0，foo2()都会被执行，导致程序出错。

那么仅仅使用{}将foo1()和foo2()包起来行么？

我们在写代码的时候都习惯在语句右面加上分号，如果在宏中使用{}，代码里就相当于这样写了：“{...};”，展开后就是这个样子：
```
if(a>0)
{
    foo1();
    foo2();
};
```
所以碰到这种调用宏:
```
if (a>0)
     DOSOMETHING();
else
     DOOTHERTHING();
```
会被扩展成
```
if (a>0) {
    foo1();
    foo2();
};
else
    DOOTHERTHING(wolf);
```
这样不会编译通过。所以，很多人才采用了do{...}while(0);

```
#define DOSOMETHING() \
        do{ \
          foo1();\
          foo2();\
        }while(0)\
    
...
 
if(a>0)
    DOSOMETHING();
 
...
```

这样，宏被展开后，才会保留初始的语义。GCC提供了Statement-Expressions用以替代do{...}while(0); 所以你也可以这样定义宏：

```
#define DOSOMETHING() ({\
        foo1(); \
        foo2(); \
})
```

2、避免使用goto对程序流进行统一的控制：

有些函数中，在函数return之前我们经常会进行一些收尾的工作，比如free掉一块函数开始malloc的内存，goto一直都是一个比较简便的方法：

```
int foo()
{
    somestruct* ptr = malloc(...);
 
    dosomething...;
    if(error)
    {
        goto END;
    }
 
    dosomething...;
    if(error)
    {
        goto END;
    }
    dosomething...;
 
END:
    free(ptr);
    return 0;
 
}
```

由于goto不符合软件工程的结构化，而且有可能使得代码难懂，所以很多人都不倡导使用，那这个时候就可以用do{}while(0)来进行统一的管理：

```
int foo()
{
 
    somestruct* ptr = malloc(...);
 
    do{
        dosomething...;
        if(error)
        {
            break;
        }
 
        dosomething...;
        if(error)
        {
            break;
        }
        dosomething...;
    }while(0);
 
    free(ptr);
    return 0;
 
}
```

这里将函数主体使用do()while(0)包含起来，使用break来代替goto，后续的处理工作在while之后，就能够达到同样的效果。


3、避免空宏引起的warning

内核中由于不同架构的限制，很多时候会用到空宏，在编译的时候，空宏会给出warning，为了避免这样的warning，就可以使用do{}while(0)来定义空宏：

```
#define EMPTYMICRO do{}while(0)
```

4、定义一个单独的函数块来实现复杂的操作：

当你的功能很复杂，变量很多你又不愿意增加一个函数的时候，使用do{}while(0);，将你的代码写在里面，里面可以定义变量而不用考虑变量名会同函数之前或者之后的重复。

理解了这个do{...}while(0),一切豁然开朗。
发现ZVAL_RESOURCE,ZVAL_STRING,ZVAL_ZVAL等宏也都是使用的do{}while(0)的形式,ZVAL_DOUBLE,ZVAL_LONG等并没有使用,个人理解ZVAL_BOOL这样定义宏属于第一种情况。