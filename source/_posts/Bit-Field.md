---
title: Bit Field
date: 2017-04-08 18:42:51
tags:
---

## 1. 大/小尾端
数据在内存中的存放方式可以分为大尾端（Big-endian）、小尾端（Little-endian）：  
1）Little-Endian就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。
2）Big-Endian就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。

如何得知本机是大尾端还是小尾端呢？只需运行如下代码：  

```c++
#include <stdio.h>  
  
int main(void)  
{  
    union {  
        unsigned int  a;  
        unsigned char b;  
    }c;  
    c.a = 1;  
    if (c.b == 1)  
        printf("Little-endian\n");  
    else  
        printf("Big-endian\n");  
  
    return 0;  
}  
```
**原理**：union中变量都是从内存低地址开始存放的，因此变量b一定与a的低位字节重叠，如果是小尾端，那么a的低位字节值为1，如果是大尾端，则a的高位字节值为1.

--------  
参考：http://blog.csdn.net/songuooo/article/details/7837070



## 2. Bit Field
### 2.1 what is bit field?
位字段是在struct中声明数据占用的内存bit数量，可用于signed/unsigned char(int) 类型，例如：  
```c
struct Field {
unsigned int a : 1;  // 变量a占1 bit
unsigned int b : 2;  // 变量b占2 bit
unsigned int c : 1;  // 变量c占1 bit
unsigned int d : 3;  // 变量d占3 bit, 最大值为7
};

struct Field s;
s.a = 1;
s.b = 3;
s.c = 0;
s.d = 7; 
```

结构体Field的size并不是4*sizeof(unsigned int)，而是1个unsigned int，即当总的bit数量没有超过unsigned int大小时，该结构体的size就是一个unsigned int大小。

### 2.2 bit field 仅仅可以节省内存吗？
显然，使用bit field可以节省内存，因此，它通常用于存储flag标志（取值0/1仅需1-bit），当然它也可以用来存储多于1-bit的enum类型值，也可以节省空间，例如：  
```c++
enum direction {
    north = 0;
    south = 1;
    west = 2;
    east = 3;
};

struct my_direct {
    unsigned int a : 2;
    unsigned int b : 2;
};

struct my_direct s;
s.b = east;
```
bit field不止可以节省内存，它也提高了代码的可读性，试想一下，对于上面的程序，如果不直接赋值，而使用位运算的方式实现，那么代码会变成下面的样子：  
```c++
#define OFFSET 2
s &= ~(3<<OFFSET);   //clear bits of b
s |= east << OFFSET;
```
因此，使用bit field之后可以很方便地以非字节对齐的方式赋值。

### 2.3 bit field 内存占用字节数
对于上面代码中的结构体my_direct，占用空间为sizeof（unsigned int），即一整个unsigned int类型的大小，即使你只用了其中4个bit。如果想要更加节省，可以使用char类型。

