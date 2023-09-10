# Threading Model

## 核心概念

Task:

线程处理的基本工作单元，通常是函数，函数对象，lambda表达式。

Task Queue:

用于存储Task的队列。

base::Thread

一个物理线程，永远处理来自专用TaskQueue的消息，直到调用了Quiet()方法。

base::ThreadPool

负责管理多个线程，这些线程之间会共享一个TaskQueue。在Chromium中实际上是base::ThreadPoolInstance，每个Chromium进程中有且仅有一个base::ThreadPoolInstance的实例。所以一般情况下，在代码中并不需要直接使用base::ThreadPoolInstance。

Sequence

Chromium线程模型中提供的一种抽象化的概念，可以理解为是一种虚拟线程(virtual thread)。在Chromium中，Sequence有如下的特征：

1. 在任何时刻，Sequence上都只能有一个Task在执行。
2. 前驱Task的执行副作用可以保证被后继Task感知到，也就是说Sequence提供了thread- safety
3. Sequence中的Task可以在不同的物理线程上进行切换。

TaskRunner

在Chromium中，Task可以通过TaskRunner进行提交。

SequenceTaskRunner

base::SequenceTaskRunner是base::TaskRunner的一个子类，在base::TaskRunner的基础上提供了一下的保证：

1. 保证提交的Task会按照Post的顺序进行执行。
2. 前驱Task的执行副作用可以保证被后继Task感知到。

SingleThreadTaskRunner

是base::SequenceTaskRunner的一个子类，在其基础之上额外提供了一个保证：所有提交的Task会保证在同一个物理线程上执行。

## 线程架构

每一个Chrome进程都具有：

一个主线程

在Browser进程中主线程是UI线程(即BrowserThread::UI)
在Renderer进程中主线程是Blink的主线程

一个IO线程

Chrome中的所有的进程都有一个IO线程，所有的IPC消息都会先送达到这个线程，但是IPC消息的处理逻辑可以放在其他的线程(即，IO线程可以将消息派发或者路由到绑定了不同线程的Mojo接口)
另外，绝大多数通用的异步IO也发生在IO线程，比如通过base::FileDescriptorWatcher。

少量的特殊目的的专用线程

一个通用的线程池

大多数线程都有一个循环，该循环从队列中获取任务并运行它们（队列可以在多个线程之间共享）。

## Tasks

Tasks本质上就是一个base::OnceClosure，而一个base::OnceClosure就是base::OnceCallback的特化版本，即base::OnceCallback<void()>。base::OnceClosure会存储一个callback的实例(如函数指针，函数对象等等)以及参数，它有一个Run()方法可以使用存储的参数来调用函数指针。

### Posting Tasks

Task创建之后，可以提交到线程池中执行。按照提交的对象来分：有ThreadPool和TaskRunner两种，按照执行的方式来分：有Parallel, Sequenced和SingleThread三种类型。我们按照后一种方法来区分。

Parallel

Parallel执行方式下，不同的Tasks之间没有任何执行的顺序保证，可以以任何顺序在任何线程上执行。有两种提交Parallel Task的方式：

1. 通过base::ThreadPool

```C++
    base::ThreadPool::PostTask(
        FROM_HERE, {base::TaskPriority::BEST_EFFORT, MayBlock()},
        base::BindOnce(&Task));
```

1. 通过base::TaskRunner

```C++
class A {
 public:
  A() = default;

  void PostSomething() {
    task_runner_->PostTask(FROM_HERE, base::BindOnce(&A, &DoSomething));
  }

  void DoSomething() {
  }

 private:
  scoped_refptr<base::TaskRunner> task_runner_ =
      base::ThreadPool::CreateTaskRunner({base::TaskPriority::USER_VISIBLE});
};
```

如果提交的任务确定没有任何顺序上的依赖关系，那么可以直接使用base::ThreadPool::PostTask

Sequenced

如果提交的任务之间需要有时间的上的先后保障，那么可以通过base::SequencedTaskRunner提交

例如，创建一个base::SequencedTaskRunner, 然后提交TaskA和TaskB。那么TaskA必然会在TaskB之前执行，TaskB可以看到TaskA执行后的效果

```C++
scoped_refptr<SequencedTaskRunner> sequenced_task_runner =
    base::ThreadPool::CreateSequencedTaskRunner(...);

// TaskB runs after TaskA completes.
sequenced_task_runner->PostTask(FROM_HERE, base::BindOnce(&TaskA));
sequenced_task_runner->PostTask(FROM_HERE, base::BindOnce(&TaskB));
```

提交到当前的Sequence的最佳方式是使用base::SequencedTaskRunnerHandle::Get()

```C++
// The task will run on the current (virtual) thread's default task queue.
base::SequencedTaskRunnerHandle::Get()->PostTask(
    FROM_HERE, base::BindOnce(&Task);
```

请注意，base::SequencedTaskRunnerHandle::Get()返回当前Virutal Thread的默认队列。在具有多个任务队列的线程(例如 BrowserThread::UI)上，这可能是与当前任务所属的队列不同的队列。“当前”任务运行程序有意不通过静态getter公开。要么您已经知道它并可以直接发布到它，要么您不知道，唯一明智的目的地是默认队列。

优先使用Sequence而不是Lock，Chrome中不鼓励使用Lock来保障线程安全，而应该将Task提交到
Sequence中。

```C++
class A {
 public:
  A() {
    // Do not require accesses to be on the creation sequence.
    DETACH_FROM_SEQUENCE(sequence_checker_);
  }

  void AddValue(int v) {
    // Check that all accesses are on the same sequence.
    DCHECK_CALLED_ON_VALID_SEQUENCE(sequence_checker_);
    values_.push_back(v);
}

 private:
  SEQUENCE_CHECKER(sequence_checker_);

  // No lock required, because all accesses are on the
  // same sequence.
  std::vector<int> values_;
};

A a;
scoped_refptr<SequencedTaskRunner> task_runner_for_a = ...;
task_runner_for_a->PostTask(FROM_HERE,
                      base::BindOnce(&A::AddValue, base::Unretained(&a), 42));
task_runner_for_a->PostTask(FROM_HERE,
                      base::BindOnce(&A::AddValue, base::Unretained(&a), 27));

// Access from a different sequence causes a DCHECK failure.
scoped_refptr<SequencedTaskRunner> other_task_runner = ...;
other_task_runner->PostTask(FROM_HERE,
                            base::BindOnce(&A::AddValue, base::Unretained(&a), 1));
```
