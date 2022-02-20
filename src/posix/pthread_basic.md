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
在Unix中，一个进程可以通过调exit, Exit, _exit退出进程，线程也一样，POSIX中提供了一些方法用于终止线程。具体来说，线程可以通过以下三种方法退出。

（1）执行完任务后正常退出。

（2）被同一个进程中的其他线程取消

（3）线程自己调用pthread_exit。
### 4.1 正常退出
线程的正常退出很容易理解，就是线程执行完分配的任务后，自然就退出了。但是情况其实没有这么简单。如果主线程先于子线程退出，那么子线程会处于何种状态呢？首先，线程之间的关系不像进程那样具有父子关系，线程之间更像是同事的关系。通常所说的主线程指生命周期最早开始的线程，通常由它创建其他的线程。其次，这里的“退出”是一个模糊不清的概念，我们至少可以列举出四种情况：

(1)return

retrun是我们最熟悉的一种情况，return表示一个函数的某条控制流走到了终点，如果我们在主线程中调用return，那么子线程会继续执行吗？答案是否定的，如果主线程retrun，那么就意味着，整个进程都退出了，因此子线程也会终止。在这种情况下，子线程就不会继续执行了。

(2)exit()

如果在主线程中调用exit家族的函数呢？这个当然也会造成进程的退出。而且，不仅主线程调用exit会终结整个进程，子线程调用也会终结整个进程。

(3)pthread_exit()

通过两种情况的分析，我们不难发现，子线程要继续执行的一个前提条件就是进程不能退出。皮之不存，毛将焉附？如果进程一旦退出，那么所有的资源都会被操作系统回收。于是，我们就需要一种机制，让主线程退出的同时保证进程不退出。于是pthread中提供了一个用于退出线程的方法：
```C++
#include <pthread.h>
void pthread_exit(void *rval_ptr);
```
其中rval_ptr中保存了线程的返回值，如果在主线程中执行这个函数，那么线程退出不会引起进程的退出，保证子线程能够继续运行。

(4)pthread_join()

pthread中还提供了一种更为灵活方式——pthread_join。
join可以理解为线程控制流的“汇合”。主线程可以根据线程ID选择与哪个线程汇合，这样可以更加精细的控制。
```C++
#include <pthread.h>
int pthread_join(pthread_t thread, void **rval_ptr);
```
返回值：0， 表示成功；非0，表示错误码。如果一个线程调用了pthread_join，那么它就会等待被传入pthread_join那个线程返回，取消，或者退出。一般在main函数中按照如下的形式调用。
```C++
int main(int argc, char **argv){
    //...
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    //...
    return 0;// or exit(0);
}
```

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

## 4.2 pthread_cancel
属于同一个进程中的线程可以通过pthread_cancel来取消其他线程，其函数声明如下所示:
```C++
#include <pthread.h>

int pthread_cancel(pthread_t id);
```
返回值:0，表示成功；非0，表示错误码。

这里需要注意的是，pthread_cancel的作用机制是向操作系统提出一个请求，然后操作系统会设置cancellation point。在某些system call中会检查这些cancellaiton point，然后取消进程。pthread_cancel不一定会取消线程，如果某个线程阻塞了或者陷入了死循环，那么cancellation point也不会生效，pthread_cancel不是一个可靠的线程控制函数。

## 4.3 设置线程退出回调函数
线程可以设置它退出时的一些回调函数，就像进程可以通过atexit()函数设置回调一样。
```C++
#include <pthread.h>

void pthread_cleanup_push(void (*rtn)(void *), void *args);

void pthread_cleanup_pop(int execute);
```
push和pop是栈这种数据结构的典型操作。这就意味着，这些回调函数会遵循FILO的顺序进行调用。压栈成功的函数会在以下三种情况被调用。

1. 调用pthread_exit();

2. 响应phtread_cancel();

3. 用非0execute参数调用pthread_cleanup_pop()；

这两个函数有一个限制，它们可以实现为宏，在这种情况下，必须配对使用，否则会编译失败。
## 5. 总结
线程和进程在操作上存在一定的相似性，我们可以按照如下的表哥进行对应：
| 进程原语 | 线程原语            | 描述                   |
| -------- | ------------------- | ---------------------- |
| fork     | pthread_create      | 创建新的控制流         |
| exit     | pthread_exit        | 从现在的控制流中退出   |
| waitpid  | pthread_join        | 从控制流中得到退出状态 |
| atexit   | pthread_cancel_push | 注册回调函数           |
| getpid   | pthread_self        | 获取控制流的ID         |
| abort    | pthread_cancel      | 请求控制流的非正常退出 |
