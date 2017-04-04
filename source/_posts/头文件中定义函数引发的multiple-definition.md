---
title: 头文件中定义函数引发的multiple definition
date: 2017-04-03 16:08:14
tags: C++
---

> 本文引用了[博文](https://zybuluo.com/uuprince/note/81709)，感谢原文作者。

**问题**：某个头文件中声明并定义了一个函数，然后在多个源码文件中调用该函数，编译链接时出现了该函数multiple definition问题，在头文件中添加了 #ifndef 头也不行，经过尝试发现如果将该函数的声明和定义分开到.h和.cpp文件之后问题消失，为什么不能将函数直接定义在.h文件中呢？


针对该问题，抽象出如下几个问题：  

### 1. 头文件中只可以放置函数声明，不可以放置函数定义吗？
以下面的程序为例：   
```c++
// a.h
#ifndef __a_h__
#define __a_h__
void funcA(void);   // 声明
void funcA(void)    // 定义
{}
#endif

// b.h
#ifndef __b_h__
#define __b_h__
void funcB(void);
#endif

// b.cpp
#include "b.h"
#include "a.h"
void funcB(void)
{
    funcA();
}

//c.h
#ifndef __c_h__
#define __c_h__
void funcC(void);
#endif

//c.cpp
#include "c.h"
#include "a.h"
void funcC(void)
{
    funcA();
}

//main.cpp
#include "b.h"
#include "c.h"
int main(int argc, char* argv[])
{
    funcB();
    funcC();
    return 0;
}
```

上述代码编译链接的时候编译器（g++）会报如下错误：  
```
c.o: In function `funcA()':
c.cpp:(.text+0x0): multiple definition of `funcA()'
b.o:b.cpp:(.text+0x0): first defined here
collect2: ld returned 1 exit status
```

> 为什么编译器在链接的时候会抱怨“funcA()重复定义”？
> 其实本质问题就是funcA的定义被放在了a.h中，如果写在a.cpp中，就不会有重复定义的问题。下面分析一下编译过程都发生了什么，这样更容易从编译器的角度理解此问题。

编译器处理include指令很简单粗暴，就是直接把头文件中的内容包含进来。所以b.cpp、c.cpp和main.cpp代码展开后可以简化为：  

```c++
// b.cpp
void funcA(void);   // 声明
void funcA(void)    // 定义
{}
void funcB(void);
void funcB(void)
{
    funcA();
}

// c.cpp
void funcA(void);   // 声明
void funcA(void)    // 定义
{}
void funcC(void);
void funcC(void)
{
    funcA();
}

// main.cpp
void funcB(void);
void funcC(void);
int main(int argc, char* argv[])
{
    funcB();
    funcC();
    return 0;
}
```
编译的时候，C++是采用独立编译，就是每个cpp单独编译成对应的.o文件，最后链接器再将多个.o文件链接成可执行程序。所以从编译的时候，从各个cpp文件看，编译没有任何问题。但是能发现一个问题，b.o中声明和定义了一次funcA()，c.o中也声明和定义funcA()，这就是编译器报重复定义的原因。有人可能会问，既然是从同一份文件include过来的函数funcA，那么定义都是同一份，为什么编译器不会智能的处理一下，让链接时候不报错呢？  
其实编译器链接的时候，并不知道b.cpp中定义的funcA与c.cpp中定义的funcA是同一个文件include过来的，它只会认为如果有两份定义，而且这两份定义如果实现不同，那么到底以哪个为准呢？既然决定不了，那就干脆报错好了。  

### 2. 为什么有些头文件中直接把函数定义都写进去了？
刚才的分析，可以得出结论：头文件中只做变量和函数的声明，而不要定义，否则就会有重复定义的错误。但是有几种情况是例外的。  

* 内联函数的定义
* 类（class）的定义
* const 和 static 变量

以上几种可以在头文件中定义，下面逐个进行解释。  
内联的目的就是在编译期让编译器把使用函数的地方直接替换掉，而不是像普通函数一样通过链接器把地址链接上。这种情况，如果定义没有在头文件的话，编译器是无法进行函数替换的。所以C++规定，内联函数可以在程序中定义多次，只要内联函数定义在同一个cpp中只出现一次就行。  
按照这个理论，上述a.h简单修改一下就可以避免重复定义了。  

```c++
// a.h
#ifndef __a_h__
#define __a_h__
inline void funcA(void);   // 内联声明
void funcA(void)    // 定义
{}
#endif
```
此外，类（class）的定义，可以放在头文件中。  
用类创建对象的时候，编译器要知道对象如何布局才能分配内存，因此类的定义需要在头文件中。一般情况下，我们把类内成员函数的定义放在cpp文件中，但是如果直接在class中完成函数声明+定义的话，这种函数会被编译器当作inline的，因此满足上面inline函数可以放在头文件的规则。但是如果声明和定义分开实现，但是都放在头文件中，那就会报重复定义了！！  
const 和 static 变量，可以放在头文件中。  
const对象默认是static的，而不是extern的，所以即使放在头文件中声明和定义。多个cpp引用同一个头文件，互相也没有感知，所以不会导致重复定义。  

### 3. 模板函数/类中要求头文件中必须包含定义才能进行模板实例化，这种定义放在头文件的情况会不会有问题？

前面分析可知，头文件中要么只有函数声明，要么是含有inline函数的定义。但是模板的定义(包括非inline函数/成员函数)要求声明和实现都必须放在头文件中，难道没有“重复定义”的问题？？？  
答案当然是不会有问题（要不template早就被抱怨死了）。其实编译器也考虑到会遇到类似的问题，在编译器或连接器的某处已经有防止重定义的处理了。这里参考stackflow中的答案：http://stackoverflow.com/questions/235616/multiple-definitions-of-a-function-template













