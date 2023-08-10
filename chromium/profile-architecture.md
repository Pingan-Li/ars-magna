# Profile Architectrue

Chromium中的许多特性和Profile绑定在一起，比如用户以及浏览器会话的关联数据，这些数据可以跨越多个浏览器窗口。在Chromium项目最开始的时候，Profile上的成员还不是很多，只有cookie jar，history database以及bookmark database等等。

随着Chromium的特性越来越多，Profile对象变得越来越重，依赖越来越多。这样会造成Profile对象初始化困难，Profile依赖的服务方面也存在相互依赖，因此初始化和析构需要小心的安排时序，否者状态就会出问题。为了解决这个问题，Profile对象开始遵循以下的原则重新设计：

1. Profile现在是一种具有最小状态的句柄对象。
2. Profile使用DependencyManager解决依赖问题。
3. Profile使用二段式shutdown模型。

## Design Goals

1. Profile类复杂度必须维持在一个可以维护的程度，避免将所有的“状态”放置在同一个地方。
2. Profile的启动和关闭必须是健壮的，必须实现一种机制去自动解决依赖问题。
3. 我们必须允许功能被编译进和编译出。 这对于任何多平台项目都很重要，但对于 iOS 来说尤其重要，因为我们使用 WebKit 而不是 Blink。 分层组件为编写同时在 iOS 和桌面上运行的代码提供了指导。 这种设计还使第三方供应商（例如 Opera、Edge、Brave...）可以轻松添加/删除他们认为合适的功能。

## 如何增加一个新的服务？

### KeyedService

KeyedService是所有Profile关联的Service的公共基类，定义在"components/keyed_service/core/keyed_service.h"中。

```C++
class KEYED_SERVICE_EXPORT KeyedService {
 public:
  KeyedService();

  KeyedService(const KeyedService&) = delete;
  KeyedService& operator=(const KeyedService&) = delete;

  // The second pass is the actual deletion of each object.
  virtual ~KeyedService();

  // The first pass is to call Shutdown on a KeyedService.
  // Shutdown will be called automatically for you. Don't directly invoke this
  // unless you have a specific reason and understand the implications.
  virtual void Shutdown();
};
```

可以发现，KeyedService总共以下的特点：

1. 显式定义无参构造。
1. 虚析构函数。
1. 不可拷贝，不可移动。
1. 定义了虚函数Shutdown。

假如现在我们有一个新增的服务Foo，那么FooService就需要继承自KeyedService基类。

```C++
class FooService : public KeyedService {
 public:
  explicit FooService(PrefService* profile_prefs, BarService* bar_service);
  virtual ~FooService() = default;

  FooService(FooService const&) = delete;
  FooService& operator=(FooService const&) = delete;
 private:
  PrefService* profile_prefs_;
  BarService* bar_service_;
};
```

注意：foo_service.h仅需依赖keyed_service.h即可，foo_service.cc文件需要依赖foo_service_factory.h

### KeyedServiceFactory

在定义完FooService之后我们需要实现FooKeyedServiceFactory，这个类可以从ProfileKeyedServiceFactory或者BrowserContextKeyedServiceFactory中派生。

```C++
class FooServiceFactory : public ProfileKeyedServiceFactory {
 public:
  static FooService* GetForProfile(Profile* profile);
  static FooServiceFactory* GetInstance();

 private:
  FooServiceFactory();
  virtual ~FooServiceFactory();

  // BrowserContextKeyedServiceFactory:
  virtual BrowserContextKeyedService* BuildServiceInstanceFor(
      content::BrowserContext* context) const override;
};
```

注意：foo_service_factory.h仅需依赖browser_context_keyed_service_factory.h，foo_service_factory.cc文件需要依赖foo_service.h(如果有impl类，则依赖foo_service_impl.h)以及browser_context_dependency_manager.h
