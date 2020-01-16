---
title: "C/C++读写文件"
date: 2020-01-10T14:31:16+08:00
draft: false
description: "C/c++"
tags: 
 - c/C++
 - linux
categories: 
 - C/C++
---





最近需要在arm_linux环境编译运行程序，许多c++特性用不了，重新回顾一下c/c++的知识。

### 读写文件

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <iostream>
using namespace std;

#define RFILENAME "1.txt"
#define WFILENAME "2.txt"

int main()
{
    	//读文件
        FILE *fd = fopen(RFILENAME,"r");   //返回FILE对象指针，指向RFILENAME
		//文件的开头:SEEK_SET 
    	//文件指针的当前位置:SEEK_CUR
    	//文件的末尾:SEEK_END
        fseek(fd,0L,SEEK_END);   //文件指针偏移到文件末尾
        int size = ftell(fd);    //返回当前文件位置
        rewind(fd);              //需要将文件指针指向文件开头

        char* buff = (char*)malloc(size);
        fread(buff,sizeof(char),size,fd);    //读文件，每次读sizeof(char)个，读size次
    	string str=buff;
        cout<<buff<<endl;
        fclose(fd);              //关闭文件
        free(buff);
    
    	//写文件
    	FILE *fw = fopen(WFILENAME,"a");   //打开或新建一个文件，在末尾追加写
    	fwrite(str.c_str(),str.length(),1,fw);   //str.c_str()传入字符串首地址
    	fclose(fw);
    
        return 0;
}

```

