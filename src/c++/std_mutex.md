---
title: C++ STL <mutex>
date: 2021-12-04 22:47:07
tags:
 - cpp
 - stl
 - concurrent
categories: cpp
---

# mutex
## 引言
mutex(mutual exclusion, 互斥元)是并发编程中一个非常重要的概念，它的保证了并发的情况下，多个线程对于收mutex保护的数据访问是排他性的。通俗地说，互斥元就是给数据加锁，这是解决数据竞争(data race)的一种基本理念。从C++11开始，标准库中也对mutex提供了支持，mutex相关的一系列类与函数被定义在头文件\<mutex>中。下面列出的类和函数仅仅是一部分，从中可以看出，C++标准提供的互斥元机制与其他的语言大同小异，最大的区别可能就来源于C++自身的特性——RAII机制。

| 类                        | 用途                             |
| ------------------------- | -------------------------------- |
| mutex                     | 基础的互斥元                     |
| timed_mutex               | 附带计时的互斥元                 |
| recursive_mutex           | 可递归(可重入)的互斥元           |
| recursive_mutex           | 可递归(可重入)的附带计时的互斥元 |
| lock_guard                | 互斥元的RAII简单封装类           |
| unique_lock               | 互斥元的RAII封装类,支持移动语义  |
| scoped_lock(Since C++ 17) | 互斥元的的RAII封装类，可避免死锁 |
| once_flag                 | call_once的辅助类                |

| 函数      | 用途                                    |
| --------- | --------------------------------------- |
| try_lock  | 尝试获取给定mutex的所有权               |
| lock      | 可以同时给多个mutex上锁，并且能防止死锁 |
| call_once | 保证函数在并发条件下仅被调用一次        |
| std::swap | unique_lock的特化版本                   |
## std::mutex
在C++中，std::mutex提供了最基本的互斥机制，它提供了两个语义：
* exclusive
* non-recursive ownership

它的对外接口如下所示：
```C++
namespace std {
  class mutex {
  public:
    constexpr mutex() noexcept;
    ~mutex();
 
    mutex(const mutex&) = delete;
    mutex& operator=(const mutex&) = delete;
    
    // locks the mutex, blocks if the mutex is not available
    void lock(); 
    // tries to lock the mutex, returns if the mutex is not available
    bool try_lock(); 
    // unlocks the mutex
    void unlock();
 
    using native_handle_type = /* implementation-defined */;
    native_handle_type native_handle();
  };
}
```
我们以发现，std::mutex提供了三个与加锁解锁相关的方法:
* lock()   
* unlock()
* try_lock()
  
首先，线程可以通过调用lock()给std::mutex加锁，如果成功，则线程可以进入临界区(发生数据竞争的代码段)执行相关代码，如果失败，则线程会被阻塞。成功加锁的线程在离开临界区之后，可以调用unlock()进行解锁，后续的线程可以继续进行加锁。如果线程忘记调用unlock()，后果是十分严重的，因为后续的线程都会竞争失败。但是，unlock()也不是简单的调用一次就能万无一失，因为需要保证在任意一条控制流上都需要调用unlock()。对于复杂的代码逻辑来说，手动调用unlock()难免会出现纰漏。所幸，C++中的RAII机制给出了近乎完美的解决方法，这也是\<mutex\>提供了一系列RAII包装类的原因。try_lock()方法虽然也是给std::mutex加锁，但是它提供了一种更为灵活的方式。线程在调用try_lock()之后会得到一个bool类型的返回值，用于告诉线程是否加锁成功。这样一来，线程就可以根据结果来进入不同的控制流，而不是像lock()一样直接将竞争失败的线程阻塞。
```C++
if (mutex.try_lock()) {
    //加锁成功的代码路径
} else {
    //加锁失败的代码路径
}
```
除了三个与锁相关的方法之外，std::mutex的拷贝构造函数和拷贝赋值运算符都显式标记为delete，表明std::mutex对象是没法被复制的。
## std::timed_mutex
std::timed_mutex在std::mutex基础上提供了计时的功能，它的对外接口如下所示：
```C++
namespace std {
  class timed_mutex {
  public:
    timed_mutex();
    ~timed_mutex();
 
    timed_mutex(const timed_mutex&) = delete;
    timed_mutex& operator=(const timed_mutex&) = delete;
 
    void lock();
    bool try_lock();

    /**
     * tries to lock the mutex, returns if the mutex has been
     * unavailable for the specified timeout duration
     */
    template<class Rep, class Period>
      bool try_lock_for(const chrono::duration<Rep, Period>& rel_time);
    template<class Clock, class Duration>
    /**
     * tries to lock the mutex, returns if the mutex has been 
     * unavailable until specified time point has been reached
     */
      bool try_lock_until(const chrono::time_point<Clock, Duration>& abs_time);
    void unlock();
 
    using native_handle_type = /* implementation-defined */;
    native_handle_type native_handle();
  };
}
```
显然，最引人注目的就是try_lock_for()和try_lock_until()两个方法，它们看上去要比其他三个加锁方法要复杂许多，甚至还带了两个模板参数。不过，仔细一看其实是使用了标准库中的\<chrono\>而已(\<chrono\>是C++标准中提供计时功能的头文件)。  
如前所述，lock()方法是最为“粗暴”一种，一旦线程竞争失败就会被阻塞。而try_lock()提供了额外的灵活度，可以让线程自行决定竞争失败之后做什，不过try_lock()过于“悲观”，在加锁失败之后不作任何等待就立刻返回false，从而使得线程进入另外的控制流。  

try_lock_for()给予了线程更大的灵活度，它允许线程等待一定的时间，时间的长短由传入的参数决定。线程只有在这段内一直没有加锁成功才会返回false，相比于try_lock()而言要“乐观”许多，因为它预计在这段时间内有较大概率加锁成功，因此付出等待是值得的，这个方法比较适用于锁竞争压力较小的场景。如下代码所示：
两个线程在竞争一个std::timed_mutex，thread1需要占用2秒的时间，不过thread2愿意最多可以等待4秒，因此thread2也可以加锁成功。
```C++
   std::timed_mutex timedMutex;
   std::thread thread1{
            [&timedMutex]() -> void {
                timedMutex.lock();
                std::this_thread::sleep_for(std::chrono::seconds{2});
                timedMutex.unlock();
            }
    };

    std::thread thread2{
            [&timedMutex]() -> void {
                if (timedMutex.try_lock_for(std::chrono::seconds{4})) {
                    std::cout << "got you!";
                } else {
                    std::cout << "heck!";
                }
            }
    };

    thread1.join();
    thread2.join();
```

try_lock_until()的功能与try_lock_for()是非常类似的，仅仅是传入的参数不同而已。try_lock_until()的时间控制是通过时间点（time_point）而不是时间段（duration）。这两个概念的区别需要了解\<chrono\>库。

## std::recursive_mutex(timed_mutex)
递归互斥元(recursive mutex)又名可重入互斥元(reentrant mutex)可重入锁(reentrant lock)，指的是同一个线程可以对其进行重复加锁(可重入性)，而不造成死锁。总结起来就是说：递归互斥元必须提供以下两种语义： 

* exclusive
* recursive ownership

之前提到的std::mutex和std::timed_mutex都属于非递归互斥元（又名不可重入锁），这意味着即便是同一个线程对其重复加锁也会造成死锁。如：
```C++
  std::mutex mutex;
  mutex.lock();
  mutex.lock(); // dead lock
  mutex.unlock();
  mutex.unlock();
```
而对于std::recursive_mutex而言，上面的操作不会有死锁的问题
```C++
 std::recursive_mutex recursiveMutex;
 recursiveMutex.lock();
 recursiveMutex.lock(); // it's OK
 recursiveMutex.unlock();
 recursiveMutex.unlock();
```
可以说，可重入性就是std::recursive_mutex(timed_mutex)与std::mutex(timed_mutex)唯一区别，除此之外它们在使用上并无不同，对外的接口也完全一致。

## std::lock_guard
前面我们提到了手动调用unlock()的问题，如果在某条控制流上没有调用unlock()，那么mutex就一直被占用。解决这个问题的方案显然是C++的RAII机制。lock_guard是标准库提供的最简单的一种RAII类。
```C++
namespace std {
  template<class Mutex>
  class lock_guard {
  public:
    using mutex_type = Mutex;
 
    explicit lock_guard(mutex_type& m);
    lock_guard(mutex_type& m, adopt_lock_t);
    ~lock_guard();
 
    lock_guard(const lock_guard&) = delete;
    lock_guard& operator=(const lock_guard&) = delete;
 
  private:
    mutex_type& pm;             // exposition only
  };
}
```
下面的代码展示了std::lock_guard的使用方式：
```C++
   std::mutex mutex;
    {
        // lock_guard初始化会调用mutex.lock()
        std::lock_guard<std::mutex> lockGuard{mutex};
    }//离开作用域lock_guard的析构过程会调用mutex.unlock()
```
## std::unique_lock
std::lock_guard的功能还是过于简单，它仅仅支持了最简单的lock()和unlock()两种方法，对于更加灵活的加锁方式就显得无能为力了。因此，C++中还提供了更为强大的std::unique_lock，它可以包装上面提到的所有类型的mutex。  
std::unique_lock具有两个私有的成员变量，其中一个变量是mutex的指针类型，用于保存mutex的地址；另外还有一个bool类型的变量，指明当前的std::unique_lock是否获取了某个mutex对象的所有权。
```C++
namespace std {
  template<class Mutex>
  class unique_lock {
  public:
    using mutex_type = Mutex;
 
    // construct/copy/destroy
    unique_lock() noexcept;
    explicit unique_lock(mutex_type& m);
    unique_lock(mutex_type& m, defer_lock_t) noexcept;
    unique_lock(mutex_type& m, try_to_lock_t);
    unique_lock(mutex_type& m, adopt_lock_t);
    template<class Clock, class Duration>
      unique_lock(mutex_type& m, const chrono::time_point<Clock, Duration>& abs_time);
    template<class Rep, class Period>
      unique_lock(mutex_type& m, const chrono::duration<Rep, Period>& rel_time);
    ~unique_lock();
 
    unique_lock(const unique_lock&) = delete;
    unique_lock& operator=(const unique_lock&) = delete;
 
    unique_lock(unique_lock&& u) noexcept;
    unique_lock& operator=(unique_lock&& u);
 
    // locking
    void lock();
    bool try_lock();
 
    template<class Rep, class Period>
      bool try_lock_for(const chrono::duration<Rep, Period>& rel_time);
    template<class Clock, class Duration>
      bool try_lock_until(const chrono::time_point<Clock, Duration>& abs_time);
 
    void unlock();
 
    // modifiers
    void swap(unique_lock& u) noexcept;
    mutex_type* release() noexcept;
 
    // observers
    bool owns_lock() const noexcept;
    explicit operator bool () const noexcept;
    mutex_type* mutex() const noexcept;
 
  private:
    mutex_type* _M_device;             // exposition only
    bool _M_owns;                  // exposition only
  };
 
  template<class Mutex>
    void swap(unique_lock<Mutex>& x, unique_lock<Mutex>& y) noexcept;
}
```

unique_lock具有8个构造函数，理解它的构造函数十分重要，因为RAII类的最引人关心的部分就是它的构造和析构过程。  

| No  | declaration                                                                                                              |
| --- | ------------------------------------------------------------------------------------------------------------------------ |
| (1) | `unique_lock() noexcept;`                                                                                                |
| (2) | `explicit unique_lock(mutex_type& m);`                                                                                   |
| (3) | `unique_lock(mutex_type& m, defer_lock_t) noexcept;`                                                                     |
| (4) | `unique_lock(mutex_type& m, try_to_lock_t);`                                                                             |
| (5) | `unique_lock(mutex_type& m, adopt_lock_t);`                                                                              |
| (6) | `template<class Clock, class Duration> unique_lock(mutex_type& m, const chrono::time_point<Clock, Duration>& abs_time);` |
| (7) | `template<class Rep, class Period>unique_lock(mutex_type& m, const chrono::duration<Rep, Period>& rel_time);`            |
| (8) | `unique_lock(unique_lock&& u) noexcept;`                                                                                 |

首先第一个就是默认构造函数(1)，以下是它的定义。它的成员初始化列表将_M_device赋值为0，_M_owns赋值为false，而构造函数的函数体没有执行任何代码，这就意味着默认构造的std::unique_lock仅仅是一个空壳而已，没有任何mutex对象与之关联，因此不会发挥mutex的功能。
```C++
   unique_lock() noexcept: _M_device(0), _M_owns(false){}
```
构造函数(2)要求传入一个mutex的引用，这个是std::unique_lock最为普遍的用法。构造器的初始化列表将传入的mutex的地址保存在_M_device变量中，并且_M_owns被设置为false。
```C++
explicit unique_lock(mutex_type& __m): _M_device(std::__addressof(__m)), _M_owns(false){
  lock();
  _M_owns = true;
  }
```
在构造函数体内部调用了lock()，下面是lock()函数的一种实现，它的内部会作两重检查，一个是检查_M_device是否为空指针，如果为空则会抛出异常。紧接着下一步就是检查_M_owns，如果为true则抛出异常。只有两步检查都通过了才会最终调用mutex的lock()，并且将_M_owns置为true，表明当前的std::unique_lock成功获得了mutex的所有权（加锁成功）。
```C++  
void lock(){
  if (!_M_device)
  __throw_system_error(int(errc::operation_not_permitted));
  else if (_M_owns)
  __throw_system_error(int(errc::resource_deadlock_would_occur));
  else{
    _M_device->lock();
    _M_owns = true;
    }
  }
```
仔细分析，你会发现这个构造函数(2)如果正常执行不抛出异常，那么一定会执行`_M_device->lock()`。于是问题就来了，如果一个std::mutex已经调用lock()后，如果不慎调用了构造函数(2)，那么将会触发死锁，例如下面的代码:
```C++
    std::mutex mutex;
    mutex.lock(); // mutex is already locked.
    std::unique_lock<std::mutex> uniqueLock{mutex}; // dead lock
```
出现死锁的根源在于mutex的lock()调用了两次，因此最直观的解决方法就是保证传给unique_lock的mutex都是处于未加锁的状态，但是这种方法显然是选择了逃避问题。于是C++提供了额外的机制用于解决这个问题，\<mutex\>中定义了三种类型的lock。单纯从字面意义上，我们很难理解三种lock的差别，但是通过观察std::unique_lock的源码，就很容易理解了。  

| Types         | Effects                                                      |
| ------------- | ------------------------------------------------------------ |
| defer_lock_t  | do not acquire ownership of the mutex                        |
| try_to_lock_t | try to acquire ownership of the mutex without blocking       |
| adopt_lock_t  | assume the calling thread already has ownership of the mutex |
  
构造函数(3),(4)和(5)分别对应了defer_lock_t, try_to_lock_t和adopt_lock_t。这三个构造函数的函数体都是空的，没有执行任何代码，唯一明显的区别就体现在成员初始化列表上。对于构造函数(3)，它的成员初始化列表将_M_device初始为mutex的地址，将_M_owns初始化为false，在整个构造过程中没有任何给mutex加锁的动作，这就体现了defer_lock_t的特点，不管传进来的mutex有没有加锁，它都绝不会给mutex加锁（获取所有权）。而对于try_to_lock_t而言，它的特点就是在成员初始化的过程中会调用mutex的`try_lock()`，并将返回值赋值为_M_owns，因此在构造函数(4)的过程中，会调用一次`try_lock()`。最后，对于adpot_lock_t而言，它与defer_lock_t非常类似，唯一的区别在于_M_owns被设置为了true。adpot_lock_t适用于这样的场景：传入的mutex的已经加锁了，并且mutex的所有权正好被当前的线程占有。
```C++
   unique_lock(mutex_type& __m, defer_lock_t) noexcept: _M_device(std::__addressof(__m)), _M_owns(false){}
   unique_lock(mutex_type& __m, try_to_lock_t): _M_device(std::__addressof(__m)), _M_owns(_M_device->try_lock()){}
   unique_lock(mutex_type& __m, adopt_lock_t) noexcept: _M_device(std::__addressof(__m)), _M_owns(true){}
```
构造函数(6)和构造函数(7)显然就是对应了timed_mutex，两者在构造函数体中同样是什么都没做，唯一需要关心的的就是在_M_owns的初始化过程中分别调用了try_lock_until和try_lock_for()。
```C++
  template<typename _Clock, typename _Duration>
  unique_lock(mutex_type& __m, const chrono::time_point<_Clock, _Duration>& __atime): 
  _M_device(std::__addressof(__m)),
  _M_owns(_M_device->try_lock_until(__atime)){}

  template<typename _Rep, typename _Period>
  unique_lock(mutex_type& __m, const chrono::duration<_Rep, _Period>& __rtime): 
  _M_device(std::__addressof(__m)),
  _M_owns(_M_device->try_lock_for(__rtime)){}

```
移动构造函数，定义非常trivial。
```C++
  unique_lock(unique_lock&& __u) noexcept: _M_device(__u._M_device), _M_owns(__u._M_owns){
    __u._M_device = 0;
    __u._M_owns = false;
  }
```

## call_once
std::call_once是一个函数模板，它保证了任何传入的Callable对象在任何情况下都只被调用一次。它的声明如下所示：
```C++
template< class Callable, class... Args >
void call_once( std::once_flag& flag, Callable&& f, Args&&... args );
```
这种只被调用一次的保证是通过once_flag实现的，在std::once_flag的定义中，once_flag起了重要的作用。以下是call_once的一种实现：
```C++
  template<typename _Callable, typename... _Args>
    void
    call_once(once_flag& __once, _Callable&& __f, _Args&&... __args)
    {
      // Closure type that runs the function
      auto __callable = [&] {
	  std::__invoke(std::forward<_Callable>(__f),
			std::forward<_Args>(__args)...);
      };

      once_flag::_Prepare_execution __exec(__callable);

      // XXX pthread_once does not reset the flag if an exception is thrown.
      if (int __e = __gthread_once(&__once._M_once, &__once_proxy))
	__throw_system_error(__e);
    }
```
前面的定义的__callable只是对传入的__f以及参数做一此包装而已。真正发生函数调用的位置在`once_flag::_Prepare_execution __exec(__callable);`于是我们再进一步查看once_flag中的_Prepare_execution的定义
```C++
  struct once_flag::_Prepare_execution
  {
    template<typename _Callable>
      explicit
      _Prepare_execution(_Callable& __c)
      {
	// Store address in thread-local pointer:
	__once_callable = std::__addressof(__c);
	// Trampoline function to invoke the closure via thread-local pointer:
	__once_call = [] { (*static_cast<_Callable*>(__once_callable))(); };
      }

    ~_Prepare_execution()
    {
      // PR libstdc++/82481
      __once_callable = nullptr;
      __once_call = nullptr;
    }

    _Prepare_execution(const _Prepare_execution&) = delete;
    _Prepare_execution& operator=(const _Prepare_execution&) = delete;
  };
```
因此函数的调用实际发生在_Prepare_execution构造函数中。

## Reference
[0] https://en.cppreference.com/w/cpp/header/mutex  
[1] http://en.wikipedia.org/wiki/Reentrant_mutex