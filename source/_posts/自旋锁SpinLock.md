---
title: 自旋锁SpinLock
date: 2017-04-03 16:34:11
tags:
---

## 常见的锁机制
Posix Threads（Pthreads）是在多核平台中进行多线程并行编程常用的API。它也提供了多种锁机制来实现多个线程对临界区的互斥访问，锁机制是实现线程同步的一种手段，另外一种手段是barrier机制，可以实现内存区域的同步。  
Pthreads库提供乐如下几种锁：  

* mutex（互斥量）
* Spin lock（自旋锁）
* conditional variable（条件变量）
* read/write lock（读写锁）

### 1.1 mutex
mutex属于sleep-waiting类型的锁，也就是说，当线程1想要锁住mutex进入临界区，那么如果mutex当前被其他线程持有，线程1就会进入阻塞状态（blocking），CPU核心会进行上下文切换将该线程放入等待队列，并开始运行其他线程。  
Pthread提供的mutex操作API如下：  
```
pthread_mutex_lock (pthread_mutex_t *mutex);
pthread_mutex_trylock (pthread_mutex_t *mutex);
pthread_mutex_unlock (pthread_mutex_t *mutex);
```

### 1.2 spin lock
自旋锁是一种busy-waiting类型的锁，也就是说，当线程2想要获取某个自旋锁时，如果当前该锁被其他线程持有，那么线程2不会阻塞，而是一直在当前CPU核心上进行忙等待并不断询问该锁是否被释放，直到获取为止。  
由于自旋锁是忙等待，不会引起调用者被切换，这样一旦锁变为可用状态，可以立刻获取到，因此自旋锁的效率要比普通的互斥锁高。但它也有一些不足之处：  
* 自旋锁等待过程中会一直占用CPU，这样如果短时间内不能获取到锁，那么CPU就在空转，使得CPU利用率降低。
* 使用过程中也可能出现死锁。
因此，自旋锁的适用场景是：锁的使用者持有锁的时间比较短，很快就会释放。否则，不适合使用自旋锁。  
Pthread提供的spin lock操作API如下：  
```
pthread_spin_lock (pthread_spinlock_t *lock);
pthread_spin_trylock (pthread_spinlock_t *lock);
pthread_spin_unlock (pthread_spinlock_t *lock);
```

### 1.3 条件变量


### 1.4 临界区


参考：  
http://blog.csdn.net/hiflower/article/details/2195350  
http://blog.163.com/helloworld_zhouli/blog/static/2033711212012240502579/  


