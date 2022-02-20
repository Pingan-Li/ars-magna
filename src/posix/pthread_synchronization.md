# pthread sychronization
## 1. 概述
多线程并发编程模型的一个基础就是，属于同一个进程的线程之间可以共享资源（主要指内存）。线程之间的交互要比进程知见的交互要简单高效地多。但是多线程并发编程模型也存在一定的问题。首先就是数据同步，多线程共享了同一个内存空间，因此需要保证各个线程之间数据是一致的；其次，多线程架构稳定性不如多进程架构来得安全，因为一个线程的崩溃会导致整个进程退出。而多进程中的任意一个进程崩溃都不影响其他的进程，进程之间的堆区和内存空间是相互独立的。因此多进程并行模型要比多进程并发要安全很多。不过随着技术的不断演进，多线程模型越来越成为主流。

多进程并发时，为了保证各个线程之间看到的内存视图是一致的，需要采取同步机制。但其实，并不是所有的内存区域都需要同步，如果整个进程的内存空间都需要同步，那么多进程并发就变得毫无意义。我们可以将内存中的变量分为一下的三种类型：

（1）共享不可变

如果变量是不可变的，那么任何线程在任何时刻看到的数据都是一个常量，所以不需要同步机制。

（3）可变不共享
如果变量是不共享的，那么其他的线程都不会访问到它，因此也不要同步机制。

（3）可变且共享
如果一个变量是可变的，同时又需要在线程之间共享，这种情况下就必须要同步机制了。
同步机制是一个笼统的概念，并没有限定具体的方案。pthread中提供的主要有以下的几种类型：

（1）mutex

（2）condition_variable

（3）barrier

....

## 2. mutex
mutex是mutual exclusive的缩写，其含义为互斥元，它的作用就是保证线程对在它作用范围内的变量的访问是排他性的。互斥元所保护的区域也被称为临界区（Critical Zone），当一个线程进入临界区后，其他想进入临界区的线程会被阻塞，直到当前的线程离开临界区后。后续的线程才会继续竞争临界区的访问权，这样一来临界区的可变变量即便是共享的，也可以保证多个线程看到的变量是一致的。
### 2.1 创建和销毁
```C++
#include <pthread.h>
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);

int phtread_mutex_destroy(pthread_mutex_t *mutex);s
```
返回值：0，表示成功；非0，返回错误码。
还有一种少见的方式就是通过PTHREAD_MUTEX_INITIALIZER进行初始化(静态分配)。
### 2.2 加锁解锁
pthread提供了两种加锁的方式
```C++
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_trylock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
返回值；0，表示成功；非0，返回错误码。

此外，有些操作系统还提供了pthread_mutex_timedlock
```C++
#include <pthread.h>

int pthread_mutex_timedlock(pthread_mutex_t restrict *mutex,
const struct timespec restrict * tsptr
);
```
令人困惑的是macOS里面没有提供这个函数。

### 2.3 死锁问题
互斥元的使用当然远远不止这么简单，死锁就是一个臭名昭著的顽疾。随着软件项目越来越复杂，不经意间出发死锁的可能性越来越大，而且越来越难以发现，排除死锁问题是极其困难的。因此最好的方法就是一开始就将代码写对。

## 3. 读写锁
读写锁与互斥元类似，不过读写锁允许更加细致的同步。互斥元的一个问题在于，临界区内即便没有发生对内存的修改，也会产生阻塞。而读写锁可以有三种状态：读锁，写锁，解锁。这里的读锁和写锁的使用是由程序员决定的，而不是操作系统决定的。
### 3.1 创建和销毁
```C++
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t restrict *rwlock,
const pthread_rwlockattr_t *restrict arrt
);

int pthrdda_rwlock_destory(pthread_rwlock_t restrict *rwlock);
```
返回值：0，表示成功；非0，返回错误码。
此外，读写锁同样可以使用PTHREAD_RWLOCK_INITIALIZER进行静态分配。

### 3.2 加锁解锁
读写锁提供了更为复杂的加锁机制：
```C++
#include <pthread.h>
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```
返回值：0，表示成功；非0，返回错误码。
trylock函数
```C++
#include <pthread.h>
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);

int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
```
返回值：0，表示成功；非0，返回错误码。

## 4. 条件变量
条件变量是另外一种同步机制。条件变变量给多个线程提供了一个汇合的场合。当条件变量和互斥量一起使用时，允许线程以无竞争的方式等待特定的条件发生。条件变量中的条件是由互斥元保护的。线程在改变条件之前必须首先对互斥元加锁。
### 4.1 创建和销毁
在pthread中，以pthread_cond_t表示条件变量类型，通过以下的函数进行创建和销毁。
```C++
#include <pthread.h>

int pthread_cond_init(pthread_cond_t restrict *cond,
const pthread_condarrt_t *restrict attr);

int pthread_cond_destroy(pthread_cont_t restrict *cond);
```
返回值：0，表示成功；非0，返回错误码。
PTHREAD_COND_INITIALIZER就不再啰嗦了。

我们使用pthread_cond_wait等待条件变量为真，或者使用timed wait进行定时等待。
```C++
#include <pthread.h>
int pthread_cond_wait(
    pthread_cond_t *restrict cond,
    pthread_mutex_t *restrict attr,
)

int pthread_cond_timedwait(
    pthread_cond_t *restrict cond,
    pthread_mutex_t * restrict mutex,
    const struct timesepc * tsptr,
)
```
返回值：0，表示成功；非0，返回错误码。

达到条件后可以通过以下的函数唤醒其他线程：
```C++
#include <pthread.h>
int pthread_cond_signal(pthread_cond_t *cond);

int pthread_cond_broadcast(pthread_cond_t *cond);
```
返回值：0，表示成功；非0，返回错误码。
pthread_cond_singal的作用是至少唤醒一个线程；而pthread_cond_broadcast则是唤醒所有的线程。
## 5. 自旋锁
自旋锁与互斥元的作用类似，不过自旋锁并不会迫使线程进入休眠的状态，而是将线程转入自旋状态（死循环）。自旋锁可以用于以下的情况，线程不会长期持有锁。
通常情况下，自旋锁的性能要比互斥元更高，这是因为互斥元引发的上下文切换会造成性能损失，但是这种优势是不稳定的，任何因素都可能抵消这种优势。比如底层硬件的进步，使得上下文切换的开销越来越小；又如，在锁竞争非常激烈的情况下，线程自旋的时间可能会很长，这样导致性能开销会超过上下文切换。
```C++
#include <pthread.h>
int pthread_spin_init(pthread_spinlock_t *lock, int pshare);

int pthread_spin_destory(pthread_spinlock_t *lock);

int pthread_spin_lock(pthread_spinlock_t *lock);

int pthread_spin_trylock(pthread_spinlock_t *lock);

int pthread_spin_unlock(pthread_spinlokc_t *lock);
```
返回值：0，表示成功；非0，返回错误码。
## 6. 屏障
屏障(barrier)是用户协调多个线程并行工作的同步机制。屏障允许每个线程等待，直到所有的合作线程都到达某一点。然后该点继续执行。
```C++
#include <pthread.h>
int pthread_barrier_init(pthread_barrier_t *restrict barrier,
    const pthread_barrierattr_t *restrict attr,
    unsigned int count,
);

int pthread_barrier_destroy(pthread_barrier_t *barrier);
```
返回值：0，表示成功；非0，返回错误码。
在初始化的时候，count表示线程数量的临界值，当被屏障阻塞的线程数量达到count时，则会解除掉所有线程的阻塞状态。要使得屏障发挥作用就很简单，直接调用下面的函数就行:
```C++
#include <pthread.h>
int pthread_barrier_wait(pthread_barrier_t *barrier);
```
返回值：0或者PTHREAD_BARRIER_SERIAL_THREAD；否则，返回错误码。
只有第一个到达屏障的线程得到PTHREAD_BARRIER_SERIAL_THREAD，剩下的都是0。

## 7.总结
虽然POSIX中提供了很多的同步机制，但是真正需要深入理解的就是mutex和condition_variable。因为以这种同步机制，你可以实现任意的复杂同步机制。