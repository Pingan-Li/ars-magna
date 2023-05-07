---
title: C++ STL <thread>
date: 2021-12-05 22:30:41
tags:
- cpp
- stl
- concurrent
- thread
categories: cpp
---
# Thread

## std::thread

在C++11之前的标准库中，“线程”在C++标准中并未予以支持，需要程序员自行封装操作系统提供的API。这种情况对于并发编程越来越普及的现状来说非常不方便，所幸C++11中及时提供了支援，与线程相关的类一般都放在\<thread\>头文件中，其中的核心类就是std::thread类。  
一个std::thread对象代表了操作系统中的一个执行线程，与Java中的Thread不同的是，C++中的std::thread对象一经创建就立刻执行，无需像Java那样调用start()方法。  
std::thread可以接受任意数量，任意类型的参数，因此你还可以通过传入一个std::promise用于“返回”函数执行的结果。

### 构造

std::thread的构造函数非常关键。应该说，在C++中，几乎所有与资源管理相关的类都会体现RAII机制。因此，构造与析构过程的设计就是RAII类的灵魂所在，也是阅读源码的一个绝佳切入点。  

1. 默认构造  
   std::thread支持默认构造，RAII类具有默认构造是不奇怪的。默认构造的RAII对象表明它不与任何操作系统的中的线程关联。从更普遍的意义上讲，一个RAII对象如果是被默认构造出来，那么，这个对象就没有与任何资源(内存，线程，Socket等等)关联。

   ```C++
    thread() noexcept = default;
   ```

2. 移动与拷贝
   std::thread不支持拷贝，拷贝构造函数和拷贝赋值运算符号都被明确delete。

   ```C++
    thread(const thread&) = delete;
    thread& operator=(const thread&) = delete;
   ```

   std::thread支持移动，在GNU中的实现如下所示：

   ```C++
    thread(thread&& __t) noexcept{ swap(__t); }

     thread& operator=(thread&& __t) noexcept{
         if (joinable())
            std::terminate();
         swap(__t);
         return *this;
    }
   ```

   非常典型的移动构造函数的写法，内部调用了swap函数。

   ```C++
   void
    swap(thread& __t) noexcept{ 
        std::swap(_M_id, __t._M_id); 
    }
   ```

   swap函数其实只做了一件事情，就是将_M_id做了交换，是std::thread内部的一个成员变量，仅需交换_M_id就能完成资源的移动，可见_M_id就是实质上的资源句柄，这里先按下不表。
3. Callable构造
   首先，Callable是指可以被调用的对象，函数或者lambda表达式，Callable具有与operator()结合的能力。我们向std::thread传入的Callable就是将被线程执行的逻辑，由于C++具有强大的元编程能力，因此std::thread可以接受任意数量的任意类型的参数。

   ```C++
   #ifdef _GLIBCXX_HAS_GTHREADS
    template<typename _Callable, typename... _Args, typename = _Require<__not_same<_Callable>>>
    explicit thread(_Callable&& __f, _Args&&... __args) {
     static_assert( __is_invocable<typename decay<_Callable>::type, typename decay<_Args>::type...>::value, 
            "std::thread arguments must be invocable after conversion to rvalues");
    #ifdef GTHR_ACTIVE_PROXY

 // Create a reference to pthread_create, not just the gthr weak symbol.
     auto __depend = reinterpret_cast<void(*)()>(&pthread_create);
    #else
auto__depend = nullptr;
    #endif
     using _Wrapper =_Call_wrapper<_Callable,_Args...>;
 // Create a call wrapper with DECAY_COPY(__f) as its target object
 // and DECAY_COPY(__args)... as its bound argument entities.
     _M_start_thread(_State_ptr(new _State_impl<_Wrapper>( std::forward<_Callable>(__f), std::forward<_Args>(__args)...)),__depend);
    }
    #endif // _GLIBCXX_HAS_GTHREADS

   ```

   当然C++强大的元编程能力也造成了代码阅读的困难，就上面的Callable构造逻辑来说，想要的彻底理解仍需一些努力。首先，会对传进来的Callable对象做一个检查，如果它不能被invoke，那么直接就会在编译期报错。然后最关键的是，对于_M_start_thread的调用，这表明在std::thread线程的构造过程中，就已经开始了线程的执行。

### 析构

## std::this_thread

在编写线程相关的代码时，常常需要使用到“当前线程”这个语义。在C++中对当前线程进行操作的方法是调用std::this_thread中的成员。  

std::this_thread是一个命名空间，里面定义的方法是自由函数，而不是隶属于某个类的成员函数。其中主要的方法有：
| 函数                                                                                                         | 作用          |
| ------------------------------------------------------------------------------------------------------------ | ------------- |
| thread::id get_id() noexcept;                                                                                | 获取线程id    |
| void yield() noexcept;                                                                                       | 让出CPU执行权 |
| template<class Clock, class Duration> void sleep_until(const chrono::time_point<Clock, Duration>& abs_time); | 控制线程sleep |
| template<class Rep, class Period> void sleep_for(const chrono::duration<Rep, Period>& rel_time);             | 控制线程sleep |

众所周知，C++中的线程库实际上是对系统提供的API进行统一抽象，很多的功能实际上依赖于操作系统的实现。这一点在上述的方法中也不例外，我们以GNU上的实现作为参照。

1. yeild()

这个方法非常简单，就是对底层的API的一层封装而已。因此，在不同操作系统上调用yeild的动作并不一致，取决于操作系统本身

```C++
    inline void
    yield() noexcept
    {
#if defined _GLIBCXX_HAS_GTHREADS && defined _GLIBCXX_USE_SCHED_YIELD
      __gthread_yield();
#endif
    }
```

2. get_id()  

get_id()方法返回的是线程的id，而且返回值是C++标准定义的一个std::thread::id的class。我们首先看std::thread::id的定义:

```C++
    class id
    {
      native_handle_type _M_thread;

    public:
      id() noexcept : _M_thread() { }

      explicit
      id(native_handle_type __id) : _M_thread(__id) { }

    private:
      friend class thread;
      friend struct hash<id>;

      friend bool
      operator==(id __x, id __y) noexcept;

#if __cpp_lib_three_way_comparison
      friend strong_ordering
      operator<=>(id __x, id __y) noexcept;
#else
      friend bool
      operator<(id __x, id __y) noexcept;
#endif

      template<class _CharT, class _Traits>
 friend basic_ostream<_CharT, _Traits>&
 operator<<(basic_ostream<_CharT, _Traits>& __out, id __id);
    };
```

不难看出，std::thread::id本质上也是对native_handle_type的一种封装而已，native_handle_type的具体类型也是因操作系统而异，在GNU中native_handle_type被定义为pthread_t，本质上是unsigned long int类型。  
那么，get_id()方法就很容易理解了，它就是返回一个std::thread::id类的对象，其中封装有native_handle_type.

```C++
namespace this_thread
  {
    /// this_thread::get_id
    inline thread::id
    get_id() noexcept
    {
#ifndef _GLIBCXX_HAS_GTHREADS
      return thread::id(1);
#elif defined _GLIBCXX_NATIVE_THREAD_ID
      return thread::id(_GLIBCXX_NATIVE_THREAD_ID);
#else
      return thread::id(__gthread_self());
#endif
    }

  } // namespace this_thread
```

3. sleep_for()
使当前的线程进入阻塞状态，阻塞的时间长度不短于传入的duration。之所以说是不短于而不是精确的等于，是因为在sleep_for()指定的时间到了之后，操作系统未必会立刻解除阻塞状态，这取决于操作系统自己的线程调度机制。我们看GNU中是如何处理的：

```C++
    /// this_thread::sleep_for
    template<typename _Rep, typename _Period>
      inline void
      sleep_for(const chrono::duration<_Rep, _Period>& __rtime)
      {
 if (__rtime <= __rtime.zero())
   return;
 auto __s = chrono::duration_cast<chrono::seconds>(__rtime);
 auto __ns = chrono::duration_cast<chrono::nanoseconds>(__rtime - __s);
#ifdef _GLIBCXX_USE_NANOSLEEP
 struct ::timespec __ts =
   {
     static_cast<std::time_t>(__s.count()),
     static_cast<long>(__ns.count())
   };
 while (::nanosleep(&__ts, &__ts) == -1 && errno == EINTR)
   { }
#else
 __sleep_for(__s, __ns);
#endif
```

显然，sleep_for()在其中会通过宏来选择调用::nanosleep()还是__sleep_for()。

4. sleep_until()
使线程阻塞，知道过了某个时间点之后再唤醒，与上面的方法一致，都是调用了操作系统的API。

```C++
    template<typename _Clock, typename _Duration>
      inline void
      sleep_until(const chrono::time_point<_Clock, _Duration>& __atime)
      {
#if __cplusplus > 201703L
 static_assert(chrono::is_clock_v<_Clock>);
#endif
 auto __now = _Clock::now();
 if (_Clock::is_steady)
   {
     if (__now < __atime)
       sleep_for(__atime - __now);
     return;
   }
 while (__now < __atime)
   {
     sleep_for(__atime - __now);
     __now = _Clock::now();
   }
      }
  } // namespace this_thread
```
