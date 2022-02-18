# pthread

## 1. 概述
在UNIX中，一个进程可以有一个或者多个线程，
这些进程共享同一个内存空间，文件描述符等等一系列的资源，这种设计适应了计算机硬件的
发展趋势——CPU核心的数量越来越多。因此，熟悉线程的概念并正确地应用是充分发挥现代处理器
的一个前提。

pthread是由posix标准定义的线程接口，这些接口规定了线程如何创建，执行任务和释放。如果想掌握在Unix类系统上的多线程编程，那么熟悉pthread中的概念和接口是必要的前提条件。

## 2. 线程ID
线程和进程一样，拥有自己的ID，但是线程的ID在它所属的进程之外并没任何意义。线程的ID类型为pthread_t, 是一个结构体。既然是结构体，那么就需要一个函数来比较不同pthread_t类型的数据。
```C++
#include <pthread.h>

int pthread_equal(pthread_t tid1, pthread_t tid2);
```
返回：0，表示两个线程ID不相等。

返回：非0，表示两个线程ID相等。

pthread_t是一个结构体而不是基本数据类型带了一些移植性问题，上面的判断相等只是一个方面，想要打印出线程ID的值同样也会面临困难，好在线程ID的作用不如进程ID那么大。

线程自身可以通过调用pthread_self()返回自己的线程ID

```C++
#include <pthread.h>
pthread_t pthread_self(void);
```
## 3. 线程的创建
在传统的Unix进程模型中，每个进程都只有一个控制线程。从概念上讲
```C++
#include <pthread.h>
int pthread_create(
    pthread_t *restrict tidp,
    const pthread_attr_t *restrict attr,
    void *(*start_rtn)(void *), 
    void *restrict arg);
```
返回值：0，表示成功；否则，返回errno。请注意线程的errno是私有的，每一个线程都有自己的errno。

线程的创建有四个参数：
1. 传入一个pthread_t对象的值，用于保存新创建的线程ID。
2. 用于控制pthread属性的结构体，一般传入空指针表示默认。
3. 线程需要执行的函数。
4. 函数所需要的参数，全部封装在一个结构体中。

线程一旦创建完成就会进入执行的状态，这一点与Java中的线程不同。

## 4. 线程终止
在Unix中，一个进程可以通过调exit, Exit, _exit退出进程，线程也一样，POSIX中提供了一些方法用于终止线程。具体来说，线程可以通过以下三种方法推出。

（1）执行完任务后正常推出。
（2）被同一个进程中的其他线程取消。
（3）线程自己调用pthread_exit。

```C++
#include <pthread.h>
void pthread_exit(void *rval_ptr);
```
rval_ptr中保存了线程的返回值，如果线程被取消，rval_ptr中就会设置线程的返回值。与rval_ptr相关的函数还有pthread_join
```C++
#include <pthread.h>
int pthread_join(pthread_t thread, void **rval_ptr);
```
返回值：0， 表示成功；非0，表示错误码。如果一个线程调用了pthread_join，那么它就会等待被传入pthread_join那个线程返回，取消，或者退出。join
join可以理解为线程的“汇合”.


```txt
// 控制流的汇合
thread-1
   |     thread-2
   |------->
   |       |
   |       |  
   |       |
   |<------+  
   |
   |
```
对于已经detach的线程，pthread_join的调用会失败，返回EINVAL错误码。熟悉C++的标准库的话，对此应该有非常深刻的记忆。

