# Overview

在Chromium的base库中，实现了一套“侵入式”智能指针的机制，由RefCounted和RefCountedThreadSafe作为模板提供给使用者。

## RefCounted

### Usage

RefCounted模板的使用方法如下，例如现在有一个MyFoo，如果需要获得引用计数的支持，则需要继承自base::RefCounted\<MyFoo\>。这种模板的使用方式被称为CRTP(curiously recurring template pattern)。

```C++
class MyFoo : public base::RefCounted<MyFoo> {
//    ...
 private:
// non-public destructor.
  friend class base::RefCounted<MyFoo>;
  ~MyFoo();
};
```

这里例子中凸显了两个要点：

1. 声明析构函数为privateh或者protected：防止MyFoo对象实例被外部代码析构。
2. 声明friend base::Refcounted\<YourClass\>，RefCounted有能力调用MyFoo的析构函数。

另外还有一些要求，RefCounted修改引用计数不是线程安全的，如果需要线程安全需要使用RefCountedThreadSafe。

### Details

RefCounted模板继承自RefCountedBase，RefCountedBase的需要做的就是实现引用计数。

## RefCountedBase

### Details

RefCounted对象禁止拷贝，拷贝构造和拷贝赋值都被显式删除：

```C++
  RefCountedBase(const RefCountedBase&) = delete;
  RefCountedBase& operator=(const RefCountedBase&) = delete;
```

仔细思考一下这个是必要的吗？

唯二的public的方法如下：

```C++
  bool HasOneRef() const { return ref_count_ == 1; }
  bool HasAtLeastOneRef() const { return ref_count_ >= 1; }
```

这两个方法都是读取引用计数，不会修改引用计数的值。RefCountedBase中的关键方法在protected或者private权限中：

```C++
protected:
  explicit constexpr RefCountedThreadSafeBase(StartRefCountFromZeroTag) {}
  explicit constexpr RefCountedThreadSafeBase(StartRefCountFromOneTag)
      : ref_count_(1) {
#if DCHECK_IS_ON()
    needs_adopt_ref_ = true;
#endif
  }
```

基类的构造函数需要暴露给子类

## RefCountedThreadSafe

TODO

## RefCountedBaseThreadSafeBase

TODO
