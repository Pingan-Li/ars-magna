# Mojo Core Overview

## Overview

概念：

1. Node
2. Port
3. Message
4. MessagePipe

Mojo Core实现了在Nodes网络之上的消息路由IPC机制，一个Node典型的例子是对应一个操作系统中的Process。消息是在一对Ports之间交换，一对Ports就代表了一个MessagePipe。MessagePipe提供了可靠的有序消息派发服务。

Messages可以用于在Nodes之间传送Port和platform handles(file descriptors, Mach prots, Windows handles, etc.)

Nodes之间的通信依赖于平台的IPC机制，比如AF_UNIX sockets, pipes 或者 Fuchsia channels. 一个新的Node可以通过给它分配一个IPC channel使它加入到一个网络中，或者建立一个新的IPC channel。

Mojo Core 提供了共享内存缓存原语

一个单独的Node在网路中也被称为是Broker？

1. 在网络中引入一对Node，并且在他们之间建立一个IPC链接。
1. 帮助在Nodes之间拷贝handles，有些Nodes可能不具备权限去拷贝handles
1. 版主创建共享内存对象，有些Process可能无法完成对应的操作。

Mojo公布了两种API

1. C system API
1. embdder API

## Chromium的初始化过程

content::RunContentProcess(ContentMainParams params,
                  ContentMainRunner* content_main_runner)

content::InitializeMojo(mojo::core::Configuration* config)
首先这里会进行判断，决定这个进程是否是Broker进程：

1. 要求是Browser进程
  如何判断是否是Browser进程呢？代码中体现的是没有kProcessType这个switch。
1. 要求在CommandLine没有Mojo invitation。

然后其实Mojo这里会选择动态和静态两种初始化方式：
判断的依据就是这个函数：IsMojoCoreSharedLibraryEnabled()
如果返回结果是false，意味着Mojo将不会使用动态初始化。
使用静态初始化是非常简单的，不论是browser进程还是其他进程都会进入
`mojo::core::Init`这个方法

```C++
  if (!IsMojoCoreSharedLibraryEnabled()) {
    mojo::core::Init(*config);
    return;
  }
```

如果说配置了动态初始化，那么最终会调用`mojo::LoadAndInitializeCoreLibrary()`

```C++
MojoResult LoadAndInitializeCoreLibrary(absl::optional<base::FilePath> path,
                                        MojoInitializeFlags flags) {
  DCHECK_EQ(flags & MOJO_INITIALIZE_FLAG_LOAD_ONLY, 0u);
  InitializationState state(path, flags);
  return MojoInitialize(&state.options);
}
```

```C++
MojoResult MojoInitialize(const struct MojoInitializeOptions* options) {
  LOG(ERROR) << "lipingan: " << __func__ << " called!";
  static base::NoDestructor<mojo::CoreLibraryInitializer,
                            base::AllowForTriviallyDestructibleType>
      initializer;

  base::StringPiece library_path_utf8;
  if (options) {
    if (!MOJO_IS_STRUCT_FIELD_PRESENT(options, mojo_core_path_length))
      return MOJO_RESULT_INVALID_ARGUMENT;
    library_path_utf8 = base::StringPiece(options->mojo_core_path,
                                          options->mojo_core_path_length);
  }

  MojoResult load_result = initializer->LoadLibrary(
      base::FilePath::FromUTF8Unsafe(library_path_utf8));
  if (load_result != MOJO_RESULT_OK)
    return load_result;

  DCHECK(g_thunks.Initialize);
  return INVOKE_THUNK(Initialize, options);
}
```

在MojoInitialize会创建一个`CoreLibraryInitializer`用于动态加载mojo的动态库，其实在底层是调用了dlopen
然而，如果是动态初始化，那么Browser和其他的进程会在不同地方完成初始化
Browser进程的mojo初始化会继续在这个方法中完成。

然而子进程的话，会在`ContentMainRunnerImpl::Run()`中进行初始化，注意这里是`LoadCoreLibrary`，而不是
`LoadAndInitializeCoreLibrary`

```C++
// If dynamic Mojo Core is being used, ensure that it's loaded very early in
// the child/zygote process, before any sandbox is initialized. The library
// is not fully initialized with IPC support until a ChildProcess is later
// constructed, as initialization spawns a background thread which would be
// unsafe here.
if (IsMojoCoreSharedLibraryEnabled()) {
  CHECK_EQ(mojo::LoadCoreLibrary(GetMojoCoreSharedLibraryPath()),
           MOJO_RESULT_OK);
}
```

ContentMain::InitializeMojo
content模块会负责初始化Mojo
|
mojo::core::Init
|
InitializeCore
|
MojoEmbedderSetSystemThunks

1. content_main.cc

1. embedder.cc
1. entrypoints.cc
1. thunks.cc

## 分层

mojo由四层组成：

C System API是稳定的，具有版本的API，通过一个单例的结构体对外公开，

1. Mojo Core:
Mojo核心功能的实现层，由C++编写。
2. Mojo System API(C):
Mojo导出的C API层，
3. Mojo System API(C++/Java/JS):
4. Mojo Bindings:

=====

## 概念

### Port

一个Port本质上一个存储地址的循环链表中的一个节点，这个循环链表有时候也被称为“路由”。
路由是一个基本的中间件，所有的Node event循环都发生在它之上，所以它就自然的成为了Mojo message passing的脊柱。

每个Port都具有一个128-bit的标识，这个标识的作用范围在一个Node之内。
一个Port本身没有做任何事情，它只是一种具体的状态集合。它所属的那个Node对象会负责管理所有事件的：产生，传递，和处理逻辑。

端口可能处于少数状态中的任何一种（请参阅下面的状态），这些状态决定了它们如何对针对它们的系统事件做出反应。

在最简单也是最普遍的情况下，Ports在创建的时候是一个关联的pair，两个Ports都处于`kReceiving`的状态。例如，A和B构成的Port pair具有如下的形式：

```text
+-----+          +-----+
|     |--------->|     |
|  A  |          |  B  |
|     |<---------|     |
+-----+          +-----+
```

A可以通过`peer_node_name`和`peer_port_name`来引用B。
一个Node，永远无法感知到当送给某个Port的message是谁发送的，它只知道当前的消息要发送给哪个Port。

为了便于记录，我们将路由中的一个接收端口称为另一个接收端口的“共轭”。接收端口的共轭在初始创建时也是其对等体，但由于代理，以后可能不是这种情况。

访问Port数据结构必须先获取lock_，并且只能够通过PortLocker, PortLocker 确保单个线程上重叠的端口锁获取始终以全局一致的顺序获取。

Port的属性:

1. peer_node_name & peer_port_name
这两个属性在Port中的声明如下所示:

```C++
NodeName peer_node_name;
PortName peer_port_name
```

1. prev_node_name & prev_port_name
TODO，暂时无法理解

1. pending_merge_peer
标记当前的Port将要被merge.

1.

这两个属性的组合可以确定当前Port的共轭端口，peer_node_name可以确定共轭端口所处的Node，而peer_port_name可以在Node中精确定位到共轭端口。
当由消息从此端口中发出时，这两个属性就可以确定消息的目的地。

Port的状态：

1. kUninitialized
处于这种状态的Port还没有配对，因此还无法使用。(参见Node::CreateUninitializedPort或Node::InitializePort)
1. kReceiving
处于这种状态的Port在Node外部是公开可见的，可以用来发送或者接受消息。在任何一个路由中都最多具有两个处于`kReceiving`的状态。
从接收端口发送的用户消息事件始终沿端口的路由循环，会出现两种结果：
直到它到达死胡同（在这种情况下，路由处于被破坏的状态）
或到达路由中的另一个处于`kReceiving`状态端口（在这种情况下，它降落在该端口的传入消息队列中，该队列可以通过用户代码读取。
1. kBuffering
TODO
1. kProxying
TODO
1. kClosed
处于这种状态下的端口被关闭了并且永远都是无法再重新启用，只有处于`kReceiving`状态下的端口可以被关闭。

### Node

Node维护了由一个128-bit的id所标识的Ports的集合，并且执行路由以及在Ports之间的事件传递，Ports可以是在同一个Node之内，也可以是跨Nodes。
典型的情况是一个系统进程内只有一个Node对象（这是最佳实践，也是Chromium目前唯一使用的方式）。所以Node边界可以作为进程边界的一种有效的建模方法。

新的Ports可以通过以下的方式创建：首先调用CreateUninitializedPort()函数创建一个未被初始化Ports，并且可以通过InitializePort完成初始化。
或者，直接通过CreatePortPair()创建一个完全初始化的PortPair。完成初始化的Port有且只有一个共轭端口，这个端口是消息传递的最终接受者。
