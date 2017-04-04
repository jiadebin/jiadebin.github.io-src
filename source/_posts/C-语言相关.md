---
title: C++语言相关
date: 2017-04-03 16:21:09
tags:
---

1、volatile：修饰符表示每次读取变量的值都从内存地址读取，不读寄存器。对于多线程访问的值来说，如果不加volatile，编译器可能进行优化，直接从寄存器读取该值，这样的话某个线程修改了该变量，寄存器中的值可能是脏数据，其他线程读取时就会读到旧值。  
2、mutable：修饰的变量在const函数中也可以被修改。  
3、C++ 枚举类型与int相互转换问题，编译器支持自动将枚举转换为int，但不支持int自动转枚举，需要强制转换。参考。  
4、namespace使用时直接以 :: 开头，如 ::val = x;   表示此处的变量val是全局作用域中的val，如果定义了一个全局变量val，函数内部也定义了一个局部变量val，那么要想使用这个全局的val，就需要加上全局作用域标识符。  
5、宏USED_PARA的[作用](http://www.cnblogs.com/verygis/archive/2012/05/03/2480130.html)：  
```
#ifndef   UNUSED_PARAM
#define   UNUSED_PARAM(v)   (void)(v)
#endif
```
假如一个有返回值的函数，如调用时是没有使用它的返回值，编译器会给出一个警告，用void强制转换一下，则明确告诉编译器不使用返回值，这样可以消除编译警告。因为如果采用严格的编译模式，未使用的变量编译时会报错。  
6、REACH_TIME_INTERVAL宏的作用是判断是否达到指定的时间间隔：  
```
#define REACH_TIME_INTERVAL(i) \
  ({ \
    bool ret = false; \
    static volatile int64_t last_time = 0; \
    int64_t cur_time = ::common::current_time(); \
    int64_t old_time = last_time; \
    if ((i + last_time) < cur_time \
        && old_time == ATOMIC_CAS(&last_time, old_time, cur_time)) \
    { \
      ret = true; \
    } \
    ret; \
  })
```
该宏返回bool值，参数i为微秒数，比如1000*1000表示1秒，last_time为static且volatile的，这样多个线程将访问到相同的值，if判断中第一个条件表示cur_time是否已经超出了last_time指定的间隔i，如果不是则跳出if并返回false，否则判断第二个条件，第二个条件为了控制并发访问，原子地更新last_time的值，如果自从old_time被赋值之后没有其他线程修改last_time的值，那么就返回true，说明超过了间隔i。比如last_time=10000，i=200，cur_time=10222，那么就说明超过了间隔。该宏的应用场景是避免一些频繁的操作，如多线程打印日志，可以限制至少经过10秒才打印一次，避免日志过多。  
7、自旋锁特点：busy-waiting，普通读写锁和mutex则是sleep-waiting。例如在一个双核的机器上有两个线程(A和B)，分别运行在核心和 核心1上。假设线程A想要通过pthread_mutex_lock对临界区加锁，而此时这个锁正被线程B所持有，那么A就会阻塞 (blocking)，Core0会在此时进行上下文切换(Context Switch)将线程A置于等待队列中，此时Core0就可以运行其他的任务(例如另一个线程C)而不必进行忙等待。而Spin lock则不然，如果使用的是自旋锁，那么线程A就会一直在Core0上进行忙等待并不停的进行锁请求，直到得到这个锁为止。自旋锁的效率比普通锁要高，但是代价是更高的CPU占用。
try_lock和lock的区别就是try_lock获取不到锁时不会等待，避免死等。  
8、explicit关键字的作用：只能用于修饰类的构造函数（通常用于修饰带参数的构造函数），这样的类不会发生隐式类型转换。  
9、LIKELY、UNLIKELY 可以给编译器提示，减少分支预测错误的出现。  
10、在.cpp文件中定义成员函数，函数中使用类的成员变量时编译报错：xxx was not declared in this scope.  
原因：低级错误，该成员函数名字前面没有加类名作用域。  
11、C++ 枚举类型与int相互转换：编译器支持自动将枚举转换为int，但不支持int自动转枚举，需要强制转换。  
但是在g++编译器中，即使用强制转换运行时也会报拒绝访问。  
```
enum Day {
  Monday,
  Tuesday,
  Wednesday
};
int main()
{
  Day d = Tuesday;
  cout << d << endl;  //输出1
  d = Day(0);         //编译通过，运行时报拒绝访问
  cout << d << endl;
  return 0;
}
```
总之，在使用枚举类型时，记得按照枚举变量名方式赋值才算正确使用方式。
参考自[博客](http://blog.csdn.net/lihao21/article/details/6825722)。
12、LIKELY/UNLIKELY宏：在代码中经常可以看到如下代码：  
```
#define likely(x)       __builtin_expect(!!(x), 1)  //用!!的目的是对于大于1的x取非一次得到0，再取非则得到1
#define unlikely(x)     __builtin_expect(!!(x), 0)
if(LIKELY(x > 0)) {
  y = 1;
} else {
  y = -1;
}
```
对于这两个宏的意义，我们首先从CPU的指令流水线谈起，CPU为了提高指令执行的效率，采用流水线技术来预取下一条指令，如下图：  
+————————————————————  
|取指令1|执行指令|输出结果|  
+————————————————————  
|空空空空|取指令2|执行指令|输出结果  
+————————————————————  
|空空空空空空空空|取指令3 …  
+————————————————————  
通常如果指令1、2、3是程序的真实执行顺序的话这样做没有问题，但是如果指令1是一个if判断，判断条件失败时就会发生分支跳转，指令2不再是下一条要执行的指令，这时CPU就需要重新从内存中读取目标指令替换指令2，这样执行效率就比较低了。分支预测失败就会降低指令预取的效果，因此GCC编译器为程序员提供了帮助处理分支预测的函数，可以提高分支预测的准确性，优化程序性能。  
比如上面的示例代码，由于LIKELY表明 x>0 的概率比较高，这样gcc编译器编译出来的指令会预取 y=1 这条指令，如果 x>0 的概率很小，则应该将LIKELY替换为UNLIKELY，这样预取的指令会变为 y=-1。
也就是说，gcc编译器编译过程中总是会预取出更可能发生的指令，而LIKELY/UNLIKELY正是程序员给编译器的暗示，让程序性能优化得更高。  
参考[文章](https://my.oschina.net/moooofly/blog/175019)  


