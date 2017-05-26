title: C perror函数
tags:
  - C
category: C
date: 2017-05-25 19:59:00

----

# perror函数

## 头文件

```
include<stdlib.h>
```

## 函数原型

```
void perror(const char *str)
```

## 函数说明
perror()用来将上一个函数发生错误的原因输出到标准设备(stderr)。参数所指的字符串会先打印出,后面再加上错误原因字符串。此错误原因依照全局变量error 的值来决定要输出的字符串。

在库函数中有个error变量，每个error值对应着以字符串表示的错误类型。当你调用"某些"函数出错时，该函数已经重新设置了error的值。perror函数只是将你输入的一些信息和现在的error所对应的错误一起输出

```
#include <stdio.h>

int main ()
{
   FILE *fp;

   rename("file.txt", "newfile.txt");

   fp = fopen("file.txt", "r");
   if( fp == NULL ) 
   {
      perror("Error: ");
      return -1;
   }
   fclose(fp);
      
   return 0;
}
```