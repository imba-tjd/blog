---
title: 异步和多线程
category: dotnet
---

### CancellationToken

* 包含一个任务是否应该被取消的信息
* 拥有CancellationToken对象的任务需要主动定期检查其令牌（token）状态，如果CancellationToken对象的IsCancellationRequested属性为true，任务需停止其操作：进行清理然后用token.ThrowIfCancellationRequested()或Task.FromCancellation(token)；直接返回会把任务设置为RunToComplete
* 调用CancellationTokenSource对象的Cancel方法可以设置其创建的CancellationToken对象的IsCancellationRequested属性为true
* 如果任务取消，Task对象的Status将会为Canceled而不是Faulted，await/Wait或获取Result时会抛出异常，详见下文
* 如果有多个ContinueWith的后续任务，会都取消
* 已经完成或出现其他异常的任务，进行取消不会变成Canceled；在任务开始前（状态为Created）进行取消能成功
* 在任务开始（Started）后但未分配时间片时进行取消，此时无需委托检查仍会抛出TaskCanceledException
* 对已取消的任务调用Start会抛InvalidOperationException
* CancelAfter可以指定超时时间

```c#
CancellationTokenSource cts = new CancellationTokenSource();
CancellationToken token = cts.Token;
Task t = new Task(fun(..., token)/()=>{....;token.ThrowIfCancellationRequested()}, token);
cts.Cancel();
```

### 按完成顺序重新组合任务序列

```c#
public static IEnumerable<Task<T>> InCompletionOrder<T>(this IEnumerable<Task<T>> source) {
    var inputs = source.ToList();
    var boxes = inputs.Select(x => new TaskCompletionSource<T>()).ToList(); // 尚未含有结果的Task

    int currentIndex = -1;
    foreach (var task in inputs)
    {
        task.ContinueWith(completed =>
        {
            var nextBox = boxes[Interlocked.Increment(ref currentIndex)];
            PropagateResult(completed, nextBox);
        }, TaskContinuationOptions.ExecuteSynchronously);
    }
    return boxes.Select(box => box.Task);
}

// 把结果包括错误复制到box里
private static void PropagateResult<T>(Task<T> completedTask,
    TaskCompletionSource<T> completionSource)
{
    switch (completedTask.Status)
    {
        case TaskStatus.Canceled:
            completionSource.TrySetCanceled();
            break;
        case TaskStatus.Faulted:
            completionSource.TrySetException(completedTask.Exception.InnerExceptions);
            break;
        case TaskStatus.RanToCompletion:
            completionSource.TrySetResult(completedTask.Result);
            break;
        default:
            // TODO: Work out whether this is really appropriate. Could set
            // an exception in the completion source, of course...
            throw new ArgumentException("Task was not completed");
    }
}
```

### 其他

* ContinueWith：OnlyOnRanToCompletion枚举可以使得只有前一个方法成功完成后才执行后续操作
* 与Parallel结合使用，可以创建一个Task，内部调用Parallel，await它；但注意如果Parallel出现异常，它本身会就抛AggregateException，await以后就还是它（Wait就是两层了）

## Parallel

```c#
Parallel.For(0, arr.Length, index => { Interlocked.Add(ref sum, arr[index]); });
Parallel.For(from, to, () => 0【线程局部变量初始化】, (i, loopstate, subtotal) => arr[i]+subtotoal【在一个线程中进行一定范围的计算，返回值作为下一次的subtotal】, x => Interlocked...【最终访问共享资源】);
Parallel.ForEach(arr, item => Interlocked.Add(ref sum, item);
Parallel.Invoke( // 并发调用多个函数
    () => WriteLine(),
    () => WriteLine()
)
```

## 线程池

* 默认情况下，“最小线程数量”与CPU的线程数相同。如果需求的线程数量少于这个数，会直接开线程；如果大于，会等待1秒看看有没有已有的空出来，然后才开
* 设ThreadPoolTaskScheduler的CreateOptions为LongRunning会直接开

## 计时器

### System.Threading.Timer

```c#
Timer( TimerCallBack callback, object state, uint dueTime, uint period);
void TimerCallback( object state);
```

* 当计时器到期后，系统会从线程池中的线程上开启一个回调方法，并传入state参数
* dueTime是第一次到期之前的时间；如果被设置为Timeout.Ifinite则计时器不会开始，设为0，回调函数会被立即调用
* period是两次调用回调函数之间的间隔，如果设置为Timeout.Ifinite则回调在被首次调用后不会被再次调用
* state可以为null

### System.Windows.Forms.Timer

* 定期把WM_TIMER消息放到程序的消息队列中。程序从队列获取消息后会在主用户接口线程中同步处理

### System.Timers.Timer

* 这个类很复杂，包含了很多成员，使我们可以通过属性和方法来操作计时器。可以运行在用户接口线程或工作线程上

## 互斥与同步机制

总共要解决三个问题：原子性、有序性、可见性（读到的一定是最后一次写入的，不会缓存到寄存器里）

Lock、Monitor、Mutex可用于进程间同步，用到了Win32内核对象；4.0后新增了几个轻量的，只能用于线程间同步

### Volatile

* 告诉编译器变量在作用域以外的地方被修改
* 每次读取都从内存中读，写都写到内存中
* 可以保证有序性和可见性，~~以及变量单次的读和写的原子性~~。但是不能保证复合操作（比如++）的原子性
* 不可用于局部变量，只能用于类成员。不能用于long和double，感觉只能用于本来的读取和写入就是原子的类型。
* Thread.VolatileRead和Thread.VolatileWrite可保证可见性，但是不能用于Volatile修饰的变量。应该是比把变量声明为volatile的开销更小

#### 双重检查锁定（可用于单例）

* 因为重排序，如果没有volatile，以下代码可能出现instance已被赋值，但却未初始化的情况？
* 另一种方式是使用静态构造函数，也可以直接把单例对象直接初始化赋值给静态或readonly成员，但是这样会在加载这个类的时候就占资源
* 但是.NET 2.0以后可不用volatile了，lock的barrier更“有效”
* 具体的单例代码移到了总的C#笔记里

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

* Lock是锁定数据或被调用的函数，用于锁定被调用端。Mutex则多用于锁定多线程间的同步调用，锁定调用端
* Mutex只能互斥线程间的调用，但是不能互斥本线程的重复调用
* mut.WaitOne()、mut.ReleaseMutex()

### 其他锁

* ReaderWriterLockSlim可提供读取器/编写器锁，允许多个线程同时读取资源。AcquireWriterLock、AcquireReaderLock、ReleaseWriterLock

### 标签锁？

* MethodImplAttribute用于方法，SynchronizationAttribute用于类
* 命名空间位于：System.Runtime.Remoting.Contexts
* [MethodImplAttribute(MethodImplOptions.Synchronized)]标签应用到实例方法，相当于对当前实例加锁lock(this)；应用到静态方法，相当于当前类型加锁，相当于 lock (typeof(Account))
* [Synchronization(SynchronizationAttribute.REQUIRED, true)]，类需要继承ContextBoundObject

## 参考

* 《Illustrated C# 2012 (4th Edition)》
* 《C# in depth》
* https://docs.microsoft.com
* https://www.cnblogs.com/lxblog/archive/2013/01/05/2846026.html
* https://www.zhihu.com/question/35284600/answer/583728189

### TODO

* System.Threading.SemaphoreSlim：信号量，相比于lock对异步更友好
* ThreadPool.QueueUserWorkItem
* https://docs.microsoft.com/zh-cn/dotnet/standard/threading/
