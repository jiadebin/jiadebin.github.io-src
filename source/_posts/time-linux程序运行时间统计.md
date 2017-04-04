---
title: time - Linux下统计程序运行时间
date: 2017-04-03 09:45:54
tags: time
---


Linux下编程时，当我们需要统计一个程序的执行时间，可以使用系统的time程序。根据这篇[文章](http://soft.chinabyte.com/os/22/11698522.shtml)的描述，time命令获取一个程序的执行时间包括程序的实际运行时间(real time)，以及程序运行在用户态的时间(user time)和内核态的时间(sys time)。
下面是一段示例代码，向文件中写入几行数字：

```cpp
#include <iostream>
#include <cstdio>
#include <unistd.h>     //可以不加

using namespace std;

int main(){
    FILE *fp = fopen("tmp.out", "r");
    int i=0, x;
    for(i=0; i<5; i++){
        fprintf(fp, "%d\n", i);
    }
    fclose(fp);

    return 0;
}
```

运行：  
```
$g++ test.cc  
$time ./a.out
```

结果：
```
real    0m0.020s
user    0m0.001s
sys     0m0.003s
```

可以看出，该程序实际运行时间为0.02秒，用户态运行时间0.001s，内核态运行时间为0.003s。内核态运行时间较长是因为使用了文件相关的系统调用，主要工作是在写文件。你可能发现，real并不是user和sys的总和，这是因为real代表的是程序从开始到结束的全部时间，即使程序不占CPU也统计时间。而user+sys是程序占用CPU的总时间，因此real总是大于或者等于user+sys的。下面我们用sleep来验证一下：

```cpp
#include <iostream>
#include <cstdio>
#include <unistd.h>     //可以不加

using namespace std;

int main(){
    FILE *fp = fopen("tmp.out", "r");
    int i=0, x;
    for(i=0; i<5; i++){
        sleep(1);
        fscanf(fp, "%d\n", &x);
        cout<<"x = "<< x << endl;
    }
    fclose(fp);

    return 0;
}
```
运行结果：

```
x = 0
x = 1
x = 2
x = 3
x = 4

real    0m5.007s
user    0m0.001s
sys     0m0.004s
```
可以看出，睡眠时，程序并不占用user和sys的时间。
Over.


