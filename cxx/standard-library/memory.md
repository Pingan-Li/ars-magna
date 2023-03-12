# memory

C++标准库提供了动态内存分配管理机制，由`memory`头文件提供，其中最重要的就是智能指针（Smart Pointer），智能指针提供了自动化的，异常安全的对象生命周期管理。最先提供的智能指针实际上`auto_ptr`，然而由于存在一些重大缺陷，在C++11版本被废弃。取而代之的是第二代智能指针，由三种类型的值智能指针组成`shared_ptr`, `unique_ptr`和`weak_ptr`。

## Shared Pointer

`shared_ptr`是一种共享所有权的智能指针，多个`shared_ptr`可以共同管理一个对象。`shared_ptr`在以下两种情况下，会销毁对象以及释放动态分配的内存：

1. 最后一个`shared_ptr`被销毁。
2. 最后一个`shared_ptr`通过`operator=`或者`reset()`重新赋值。

对象的销毁默认使用delete表达式，或者可以自己传入一个deletor。一个`shared_ptr`也可以不拥有任何对象的所有权，也被称为空指针。`shared_ptr`在内部存储了指向对象的原始指针，通过`get()`方法可以拿到这个原始指针，如果没有特殊需求，一般不要把原始指针泄漏出来。`share_ptr`满足`CopyConstructible`和`CopyAssignable`以及`LessThanComparable`三个`Contepts`，并且可以在给定的上下文情况下转换成bool类型。

所有的成员函数（包括拷贝构造和拷贝赋值）都以可以由`shared_ptr`不同实例上的多个线程调用，而无需额外的同步，即便这些不同的`shared_ptr`实例共享了同一个对象的所有权。但是，如果多个线程访问同一个实例并且没有同步，那么就会发生数据竞争。如果理解这个现象？一言以蔽之，`shared_ptr`自身并不是线程安全的，但是，`shared_ptr`内部的control block是线程安全的。

### API

#### reset

```C++
void reset() noexcept;

template< class Y >
void reset( Y* ptr );

template< class Y, class Deleter >
void reset( Y* ptr, Deleter d );

template< class Y, class Deleter, class Alloc >
void reset( Y* ptr, Deleter d, Alloc alloc );
```

reset()函数用于替换当前的管理的对象，可以提供一个deletor。默认情况下，deletor就是delete表达式。

如果`*this`（也就是当前的`shared_ptr`实例）拥有一个对象的所有权，并且是最后一个存活的`shared_ptr`的实例。那么，这个被管理的对象在`reset()`之后会被释放掉。

1. 空参数重载版本，调用后表明当前的`shared_ptr`要释放这个对象，并且`*this`不再管理任何对象。
2. 用`Y* ptr`替换当前被`shared_ptr`管理的那个对象，千万注意，这里Y类型并不是任意的，而要是一个完整类型并且可以隐式转换成T类型。
3. 在2.的基础上，再传入一个deletor
4. 在3.的基础上，再传入一个Allocator

#### swap

```C++
void swap( shared_ptr& r ) noexcept;
```

将`*this`管理的对象和`r`管理的对象进行交换，引用计数在这个过程中不会被调整。

#### get

```C++
T* get() const noexcept;  //  (until C++17)

element_type* get() const noexcept;  // (since C++17)
```

返回存储的指针，注意区分managed pointer和stored pointer的区别。

#### operator* & operator->

```C++
T& operator*() const noexcept;

T* operator->() const noexcept;
```

对stored pointer进行解引用，如果stored pointer是nullptr，那么该行为是未定义的。

1. 等价于`*get()`
2. 等价于`get()`

#### operaotr[]

```C++
element_type& operator[]( std::ptrdiff_t idx ) const;
```

返回array的第idx个元素。

`operatorp[]` 操作符可以作用于`shared_ptr`内部stored pointer指向的array。
如果：stored pointer 和 index 为负数，那么行为都是未定义的/

#### use_count

```C++
long use_count() const noexcept;
```

返回管理当前对象的不同`shaerd_ptr`实例，如果不存在被管理的对象（也就是空指针），那么返回0。在多线程环境中，use_count()返回值是近似值。

#### operator bool

```C++
explicit operator bool() const noexcept;
```

判断当前的`shared_ptr`是否是空指针，等价于`get() != nulltpr`。如果`*this`拥有stored pointer，则返回`true`，否则返回`false`。

#### make_shared

```C++
template< class T, class... Args >
shared_ptr<T> make_shared( Args&&... args ); // T is not Array

template< class T >
shared_ptr<T> make_shared( std::size_t N ); // T is U[]


template< class T >
shared_ptr<T> make_shared(); // T is U[N]

template< class T >
shared_ptr<T> make_shared( std::size_t N, const std::remove_extent_t<T>& u ); // T is U[]

template< class T >
shared_ptr<T> make_shared( const std::remove_extent_t<T>& u ); // T is U[N]


template< class T >
shared_ptr<T> make_shared_for_overwrite(); // T is not U[]

template< class T >
shared_ptr<T> make_shared_for_overwrite( std::size_t N ); // T is U[]
```

### Implemetation Details

在典型的实现中，`shared_ptr`拥有两种个指针

1. stored pointer (也就是`get()`返回的那个值)
2. a pointer to `control block`

所谓的`control block`持有：

1. managed object或者指向managed object的指针。
2. Deletor
3. Allocator
4. 持有同一个managed object的shared_ptr的数量
5. 引用同一个managed object的weak_ptr的数量

## Unique Pointer

`unique_ptr`是一个智能指针，与`shared_ptr`不同的是，它对于被管理对象的所有权是独占的。当发生以下的情况时，会销毁被管理的对象：

1. `unique_ptr`被销毁。
2. `unique_ptr`通过`operator=`或者`reset()`重新赋值。

`unique_ptr`可以为空，`unique_ptr`是两种类型：

1. 管理一个单独的对象(allocated with new)
2. 管理一个数组(allocated with new[])

千万要注意，只有`non-const`的`unique_ptr`可以转移所有权。

### API

#### release

```C++
pointer release() noexcept;
```

释放`unique_ptr`所管理对象的所有权，并返回被管理对象的指针。之后再通过`unique_ptr`调用`get()`时，会返回`nullptr`。但是相应地，调用者需要负责原来的对象的生命周期。

#### reset

```C++
void reset( pointer ptr = pointer() ) noexcept;
```

假如`*this`当前持有的指针是`current_ptr`, 当调用reset()时会执行以下的逻辑：

1. 将current_ptr赋值给old_ptr，作为保存。
2. 将ptr赋值给currnet_ptr，更新current_ptr。
3. 如果old_ptr是非空的，那么就会销毁old_ptr指向的对象。

```C++
template< class U >
void reset( U ptr ) noexcept;
```

```C++
void reset( std::nullptr_t = nullptr ) noexcept;
```

#### swap

```C++
void swap( unique_ptr& other ) noexcept;
```

交换两个`unique_ptr`的管理对象和deleter

## Weak Pointer

`weak_ptr`一般作为`shared_ptr`的辅助类。`weak_ptr`持有`shared_ptr`管理对象的的一个"弱引用"，这里所谓的弱引用，是指该引用对管理对象不具有所有权，不改变引用计数。但是`weak_ptr`可以升级成`shared_ptr`，取得对象的临时所有权。此外，`weak_ptr`的另外一个重要用途就是，打破`shared_ptr`造成的循环引用，循环引用会导致`shared_ptr`永远不会销毁对象，造成内存泄漏。

### API

#### reset

```C++
void reset() noexcept;
```

释放掉引用

#### swap

```C++
void swap( weak_ptr& r ) noexcept;
```

交换两个`weak_ptr`中管理的对象

#### expired

```C++
bool expired() const noexcept;
```

等价于`use_count() == 0`，判断`weak_ptr`指向的对象是否已经被销毁。

此方法在多线程的情况下存在数据竞争的问题，当返回`false`的时候，可能调用者拿到的值是过时的，但是`true`结果是可信的。

#### lock

```C++
std::shared_ptr<T> lock() const noexcept;
```

创建一个`shared_ptr`对象以临时的获取对象的所有权，如果`weak_ptr`是空的，那么`shared_ptr`也是空的。

## 总结

| | shared_ptr | unique_ptr | weak_ptr|
| - | - | - | - |
| 可拷贝 | 是 | 否 | 是 |
| 可移动 | 是 | 否 | 是 |
| 所有权 | 共享 | 独占 | 无 |
