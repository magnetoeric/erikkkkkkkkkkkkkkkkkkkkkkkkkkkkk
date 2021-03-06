title: malloc
tags:
  - C
category: C
date: 2017-05-23 12:08:00

-----
# malloc函数
## 函数原型

```
void* malloc (size_t size);
```

## 头文件

```
#include <stdlib.h>
```

##作用:
函数用来动态地分配内存空间

##参数说明 
size 为需要分配的内存空间的大小

##函数说明
malloc() 在堆区分配一块指定大小的内存空间，用来存放数据。这块内存空间在函数执行完成后不会被初始化，它们的值是未知的。如果希望在分配内存的同时进行初始化，请使用 calloc() 函数。

##注意
函数的返回值类型是 void *，void 并不是说没有返回值或者返回空指针，而是返回的指针类型未知。所以在使用 malloc() 时通常需要进行强制类型转换，将 void 指针转换成我们希望的类型，
例如：
```
int  *ptr = (int *)malloc(10*sizeof(int));  // 分配10个int的的内存空间，用来存放int数组
```
申请了内存空间后，必须检查是否分配成功。
当不需要再使用申请的内存时，记得释放.释放后应该把指向这块内存的指针指向NULL，防止程序后面不小心使用了它.这个函数和free应该配对使用。如果申请后不释放就是内存泄露；如果无故释放那就是什么也没有做。释放只能一次，如果释放两次及两次以上会
出现错误（释放空指针例外，释放空指针其实也等于啥也没做，所以释放空指针释放多少次都没有问题）。
