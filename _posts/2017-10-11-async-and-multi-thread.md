---
title: 异步和多线程
category: dotnet
---

## 线程池

* 默认情况下，“最小线程数量”与CPU的线程数相同。如果需求的线程数量少于这个数，会直接开线程；如果大于，会等待1秒看看有没有已有的空出来，然后才开
* 设ThreadPoolTaskScheduler的CreateOptions为LongRunning会直接开

## 互斥与同步机制

总共要解决三个问题：原子性、有序性、可见性（读到的一定是最后一次写入的，不会缓存到寄存器里）

Lock、Monitor、Mutex可用于进程间同步，用到了Win32内核对象；4.0后新增了几个轻量的，只能用于线程间同步

### volatile

* 告诉编译器变量在作用域以外的地方被修改
* 每次读取都从内存中读，写都写到内存中
* 保证有序性、可见性、变量单次读和写的原子性。不保证复合操作如++的原子性
* 不可用于局部变量，只能用于类成员。不能用于long和double
* 是System.Threading.Volatile.Read(ref x)/Write()的简化版，它支持long double 数组，而关键字不支持

#### 双重检查锁，可用于单例

* 因为重排序，如果没有volatile，以下代码可能出现instance已被赋值，但却未初始化的情况？
* 另一种方式是使用静态构造函数，也可以直接把单例对象直接初始化赋值给静态或readonly成员，但是这样会在加载这个类的时候就占资源
* 但是.NET 2.0以后可不用volatile了，lock的barrier更“有效”

```c#
class A {
    volatile static A instance;
    A() { } // hide construction method
    public A GetA() {
        if (instance == null)
            lock (typeof(A)) {
                if (instance == null)
                    instance = new A();
            }
        return instance;
    }
}
```

### Lock锁

* 最佳做法：private static readonly object obj = new object();，加上readonly可以确保不被修改
* lock会对expression做装箱操作，然后进行比较。不要lock值类型，因为装箱后必然不同；不要lock字符串字面量，因为它在CLI中是唯一对象，其他的可能会使用
* 不要对public类使用lock(this)，因为可能被修改；而且这样也只对类里的对象有效，不会对不同类实例有效
* Lock的底层是用Monitor实现的，大概类似于TryEnter再在finally里Exit
* 退出时应该会保证修改都同步到内存里去了。不过不会保证lock内部的有序性

### [Interlocked](https://docs.microsoft.com/zh-cn/dotnet/standard/threading/interlocked-operations)

* 提供原子操作，加/减/自增/交换；其中加只有int的，但文档给了个实现浮点的例子
* 如果是操作静态方法，可能还是要加volatile

### SpinLock

```c#
static SpinLock _spinlock = new SpinLock();
bool lockTaken = false;
try {
    _spinlock.Enter(ref lockTaken);
    _queue.Enqueue( data );
} finally {
    if (lockTaken) _spinlock.Exit(false);
}
```

* SpinWait.SpinWait、SpinWait.SpinUntil

### Mutex

* Lock用在被调用端上。Mutex多用在调用端上
* Mutex能互斥线程或进程间的调用，不互斥本线程的重复调用
* mut.WaitOne()、mut.ReleaseMutex()

### 其他锁

* ReaderWriterLockSlim可提供读取器/编写器锁，允许多个线程同时读取资源。AcquireWriterLock、AcquireReaderLock、ReleaseWriterLock

### 标签锁

* 命名空间位于：System.Runtime.Remoting.Contexts
* 用于方法：[MethodImplAttribute(MethodImplOptions.Synchronized)]应用到实例方法相当于对当前实例加锁lock(this)；应用到静态方法，相当于当前类型加锁lock(typeof(MyClass))
* 用于类：[Synchronization(SynchronizationAttribute.REQUIRED, true)]，类需继承ContextBoundObject，故没用

## 参考

* https://www.cnblogs.com/lxblog/archive/2013/01/05/2846026.html
* https://www.zhihu.com/question/35284600/answer/583728189

### TODO

* System.Threading.SemaphoreSlim：信号量，异步时用。lock里面不能用await，因为Monitor不能释放不在当前线程获取的锁
* https://docs.microsoft.com/zh-cn/dotnet/standard/io/common-i-o-tasks
* https://docs.microsoft.com/zh-cn/dotnet/standard/threading/index
* https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/index
* https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
* https://stackoverflow.com/questions/50985593/how-do-i-implement-an-async-i-o-bound-operation-from-scratch
* https://stackoverflow.com/questions/1949131/net-dictionary-locking-vs-concurrentdictionary
* https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/
* https://zhuanlan.zhihu.com/p/46673002 看到 使用Monitor来同步
* https://zhuanlan.zhihu.com/p/345492089 https://zhuanlan.zhihu.com/p/349503079 https://devblogs.microsoft.com/dotnet/an-introduction-to-system-threading-channels/ System.Threading.Channels
* list.SyncRoot
