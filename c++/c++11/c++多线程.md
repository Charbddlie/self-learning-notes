# c++多线程

https://zhuanlan.zhihu.com/p/194198073

## 2

### 2.1创建线程

#### join detach

join()：主函数调用，等待子线程处理完成，获得了子线程结果后，主函数继续执行

### 2.2

#### 死锁条件

1. 互斥（资源同一时刻只能被一个进程使用）
2. 请求并保持（进程在请资源时，不释放自己已经占有的资源）
3. 不剥夺（进程已经获得的资源，在进程使用完前，不能强制剥夺）
4. 循环等待（进程间形成环状的资源循环等待关系）

#### 临界区、信号量、互斥量（锁）的区别与联系：

三者都可用来进行进程的同步与互斥；

**临界区**速度最快，但只能作用于同一进程下不同线程，不能作用于不同进程；临界区可确保某一代码段同一时刻只被一个线程执行；

EnterCriticalSection() 进入临界区

LeaveCriticalSection() 离开临界区

**信号量**多个线程同一时刻访问共享资源，进行线程的计数，确保同时访问资源的线程数目不超过上限，当访问数超过上限后，不发出信号量；

P操作 申请资源

V操作 释放资源

**互斥量**（锁）比临界区慢，但支持不同进程间的同步与互斥；

#### 锁的类型

+ 互斥锁：mutex
+ 读写锁：shared_mutex
+ 自旋锁

#### 常用函数

```c++
#include <mutex>
mutex m;//初始化mutex对象，不能理解为定义变量
1.
m.lock();
m.unlock;
2.lock_guard
{
    lock_guard<mutex> g1(m);//用此语句替换了m.lock()；lock_guard传入一个参数时，该参数为互斥量，此时调用了lock_guard的构造函数，申请锁定m
    //此时不需要写m.unlock(),g1出了作用域被释放，自动调用析构函数，于是m被解锁
}
{
    m.lock();//手动锁定
    lock_guard<mutex> g1(m,adopt_lock);
    //自动解锁
}
3.unique_lock:
//std::unique_lock类似于lock_guard,只是std::unique_lock用法更加丰富，同时支持std::lock_guard()的原有功能。

3.1//使用std::unique_lock后可以手动lock()与手动unlock();
3.2//std::unique_lock的第二个参数，除了可以是adopt_lock,还可以是try_to_lock与defer_lock;
3.3//std::unique_lock对象实例化后会直接接管std::mutex，std::mutex对象的所有权不需要手动转移给std::unique_lock
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;
void proc1(int a)
{
	unique_lock<mutex> g1(m, defer_lock);//始化了一个没有加锁的mutex
	cout << "xxxxxxxx" << endl;
	g1.lock();//手动加锁，注意，不是m.lock();注意，不是m.lock(),m已经被g1接管了;
	cout << "proc1函数正在改写a" << endl;
	cout << "原始a为" << a << endl;
	cout << "现在a为" << a + 2 << endl;
	g1.unlock();//临时解锁
	cout << "xxxxx" << endl;
	g1.lock();
	cout << "xxxxxx" << endl;
}//自动解锁

void proc2(int a)
{
	unique_lock<mutex> g2(m, try_to_lock);//尝试加锁一次，但如果没有锁定成功，会立即返回，不会阻塞在那里，且不会再次尝试锁操作。
	if (g2.owns_lock()) {//锁成功
		cout << "proc2函数正在改写a" << endl;
		cout << "原始a为" << a << endl;
		cout << "现在a为" << a + 1 << endl;
	}
	else {//锁失败则执行这段语句
		cout << "" << endl;
	}
}//自动解锁

int main()
{
	int a = 0;
	thread t1(proc1, a);
	t1.join();
	//thread t2(proc2, a);
	//t2.join();
	return 0;
}

4.condition_variable:
#include <condition_variable>
```

如何使用？std::condition_variable类搭配std::mutex类来使用，std::condition_variable对象(std::condition_variable  cond;)的作用不是用来管理互斥量的，它的作用是用来同步线程，它的用法相当于编程中常见的flag标志（A、B两个人约定flag=true为行动号角，默认flag为false,A不断的检查flag的值,只要B将flag修改为true，A就开始行动）。

类比到std::condition_variable，A、B两个人约定notify_one为行动号角，A就等着（调用wait(),阻塞）,只要B一调用notify_one，A就开始行动（不再阻塞）。

std::condition_variable的具体使用代码实例可以参见文章中“生产者与消费者问题”章节。

wait(locker) :

wait函数需要传入一个std::mutex（一般会传入std::unique_lock对象）,即上述的locker。wait函数会自动调用 locker.unlock()  释放锁（因为需要释放锁，所以要传入mutex）并阻塞当前线程，本线程释放锁使得其他的线程得以继续竞争锁。一旦当前线程获得notify(通常是另外某个线程调用 notify_* 唤醒了当前线程)，wait() 函数此时再自动调用 locker.lock()上锁。

cond.notify_one(): 随机唤醒一个等待的线程

cond.notify_all(): 唤醒所有等待的线程

### 2.3 异步线程

```c++
#include<future>
```

#### **async与future：**

std::async创建子线程，返回std::future对象，主线程通过future对象的.get()方法取值

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include<future>
#include<Windows.h>
using namespace std;
double t1(const double a, const double b)
{
 double c = a + b;
 Sleep(3000);//假设t1函数是个复杂的计算过程，需要消耗3秒
 return c;
}

int main() 
{
 double a = 2.3;
 double b = 6.7;
 future<double> fu = async(t1, a, b);//创建异步线程线程，并将线程的执行结果用fu占位；
 cout << "正在进行计算" << endl;
 cout << "计算结果马上就准备好，请您耐心等待" << endl;
 cout << "计算结果：" << fu.get() << endl;//阻塞主线程，直至异步线程return
        //cout << "计算结果：" << fu.get() << endl;//取消该语句注释后运行会报错，因为future对象的get()方法只能调用一次。
 return 0;
}
```

#### **shared_future**

std::future与std::shard_future的用途都是为了**占位**，但是两者有些许差别。

std::future的get()成员函数是**转移**数据所有权

std::shared_future的get()成员函数是**复制**数据

因此： 

**future对象的get()只能调用一次**；无法实现多个线程等待同一个异步线程，一旦其中一个线程获取了异步线程的返回值，其他线程就无法再次获取。 

**std::shared_future对象的get()可以调用多次**；可以实现多个线程等待同一个异步线程，每个线程都可以获取异步线程的返回值。

### **2.4 原子类型atomic<>**

互斥量的加锁一般是针对一个代码段，而原子操作针对的一般都是一个变量(操作变量时加锁防止他人干扰)

std::atomic<>是一个模板类，使用该模板类实例化的对象，提供了一些保证原子性的成员函数来实现共享数据的常用操作。

（不准确地）理解为：std::atomic<>用来定义一个自动加锁解锁的共享变量

std::atomic<>对象提供了常见的原子操作（通过调用成员函数实现对数据的原子操作）： store是原子写操作，load是原子读操作。exchange是于两个数值进行交换的原子操作。 

**即使使用了std::atomic<>，也要注意执行的操作是否支持原子性**，也就是说，你不要觉得用的是具有原子性的变量（准确说是对象）就可以为所欲为了，你对它进行的运算不支持原子性的话，也不能实现其原子效果。

一般针对++，–，+=，-=，&=，|=，^=是支持的，这些原子操作是通过在std::atomic<>对象内部进行运算符重载实现的。

## 3 代码实例

### 3.1 生产者消费者问题

见原文

## 4 多线程并发高级知识

### 4.1 线程池

#### 4.1.1 线程池基础知识

为了减少创建与销毁线程所带来的时间消耗与资源消耗，因此采用线程池的策略

**线程池所解决的问题：**

(1) 需要频繁创建与销毁大量线程的情况下，由于线程预先就创建好了，接到任务就能马上从线程池中调用线程来处理任务，**减少了创建与销毁线程带来的时间开销和CPU资源占用**。

(2) 需要并发的任务很多时候，无法为每个任务指定一个线程（线程不够分），使用线程池可以将提交的任务挂在任务队列上，等到池中有空闲线程时就可以为该任务指定线程。

#### 4.1.2 线程池的实现

太难了他没更

可以通过阅读 《C++ Concurrency in Action, Second Edition》 9.1章节来学习

## 5 拓展延伸

### 5.1 并行与并发的区别

### 5.2 创建线程时的传参问题分析

如“std::thread th1(proc1)”,创建线程时需要传递函数名作为参数，提供的函数对象会复制到新的线程的内存空间中执行与调用。

如果用于创建线程的函数为含参函数，那么在创建线程时，要一并将函数的参数传入。常见的，传入的参数的形式有基本数据类型(int，char,string等)、引用、指针、对象这些

1. 当传入参数为**基本数据类型(int，char,string等)**时，**会拷贝**一份给创建的线程；
2. 当传入参数为**指针**时，**会浅拷贝**一份给创建的线程，也就是说，只会拷贝对象的指针，不会拷贝指针指向的对象本身。
3. 当传入的参数为**引用**时，实参必须**用ref()函数处理**后传递给形参，否则编译不通过，**此时不存在“拷贝”行为**。引用只是变量的别名，在线程中传递对象的引用，那么该对象始终只有一份，只是存在多个别名罢了
4. 当传入的参数为**类对象**时，**会拷贝**一份给创建的线程。此时会调用类对象的拷贝构造函数。

### 5.3 detach()

使用detach()时，可能存在**主线程比子线程先结束**的情况，主线程结束后会释放掉自身的内存空间；在创建线程时，如果std::thread类传入的参数含有**引用或指针**，则子线程中的数据依赖于主线程中的内存，主线程结束后会释放掉自身的内存空间，则子线程会出现错误。



stl容器不是线程安全的


# c++11 并发指南

https://www.cnblogs.com/haippy/p/3235560.html