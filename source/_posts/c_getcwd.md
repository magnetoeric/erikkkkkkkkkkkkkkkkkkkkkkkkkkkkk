title: C getcwd函数
tags:
  - C
category: C
date: 2017-05-26 19:59:00

----

#getcwd函数

## 函数原型

```
char *getcwd( char *buffer, int maxlen );
```

# 头文件

```
include <unistd.h>
```

## 功能
获取当前工作目录

## 参数说明
getcwd()会将当前工作目录的绝对路径复制到参数buffer所指的内存空间中,参数maxlen为buffer的空间大小。
## 返回值
成功则返回当前工作目录，失败返回 FALSE。

## 使用实例

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#define MAX_LEN 1024
int main(int argc,char* argv[]){
	char buff[MAX_LEN];
	char *cwd = getcwd(buff,MAX_LEN);
	if(NULL == cwd){
		perror("get current working director fail\n");
		exit(-1);
	}
	printf("%s\n",buff);
	return 0;
}
```