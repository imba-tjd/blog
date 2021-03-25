---
title: 异步和多线程
category: dotnet
---

## Async/Await

* 接口的函数声明不需要async修饰
* 参数不能为out或ref参数
* IO绑定用await和普通的异步函数，不要使用并行库，不会开新线程；CPU绑定使用await Task.Run(()=>Fun())，需要长时间运行时用TaskFactory.StartNew的一个重载
* 最好不要把async和LINQ结合，因为后者会延迟执行，可能会阻塞；可用ToList去掉lazy特性
* ~~不能在catch和finally块~~ C#6可以了、非异步匿名函数、lock语句块或不安全代码中使用
* 不要在一个表达式中直接多次使用await，如`r=await F1()*await F2()`，这样无法并行任务，因为C#规定先对左边求值再对右边求值
* 如果需要对参数进行验证，需要编写一个同步的方法验证参数，再return调用异步方法 （可以用匿名函数），这样可以非lazy处理
* 异步Main：不存在async void Main；VS无法正常识别async Task Main
* await 异步IO不会创建新线程，await new Task才会，await普通的async方法，完成后WPF等会恢复到之前的线程上，Console不会
* 构造函数不能用async，一定要调用异步函数最好只调用async void的

### Task

* 一般返回`Task<T>`，如果函数有async修饰，不管有没有await，在函数体中直接return T就会自动包装，也无法返回`Task<T>`
* 返回Task的可以await但没有值，返回void的无法await
* 没有async修饰也可返回Task，return Task.FromResult/Fromxxx/CompletedTask或调用返回值为Task的函数即可。这样在函数内不能await，异常会往上走，会同步执行，但资源消耗小，也用于把同步方法封装成异步方法
* Task的状态：t.Status、t.IsFault、IsCompleted(包括成功、取消和失败)、IsCompletedSuccessfully
* 同步等待/在同步方法中调用异步方法：t.Result、t.Wait()、t.GetAwaiter().GetResult()、t.WaitAll()、t.WaitAny()、Thread.Sleep()；其中GetAwaiter()不会抛AggregateException而已，在WPF中都可能死锁
* 异步等待：await、await Task.WhenAll()、await Task.WhenAny()、await Task.Delay()、await Task.Yield()离开当前的消息队列，回到队列末尾
* 不要调用它的Dispose，4.5后做了大更改

### Task.Run的重载

几种普通的重载略，这里有两种特殊的，看起来返回值不是自己，而是参数里的Task。

```c#
Task t = Task.Run(() => Task.Run(() => Console.WriteLine(3))); // Task Task.Run(Func<Task> fun);
int value = await Task.Run(() => Task.Run(() => 4)); // Task<TResult> Task.Run(Func<Task<TResult>> fun);
```

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

### 异常处理

* 如果在Task中catch住了exception，则Task没有被取消**且**没有未处理的异常，所以Task的Status为RanToCompletion；没有未处理的异常，所以IsFaulted为False
* Task在后台运行时出现异常，不会直接抛出来，但此时Status为Faulted，IsFaulted为True
* 出现异常后，使用t.Wait()、t.Result、t.Exception会获得具有InnerExceptions的AggregateException；使用GetAwaiter().GetResult()、await会获得第一个内部异常，如果有多个，后面的就丢掉了
* 任务取消时，如果它只有一个线程，Status会变为Canceled，await会抛出TaskCanceledException；如果有多个线程，Status会变为Faulted，await会抛出AggregateException
* async void方法抛的异常永远也不会捕获到
* WhenAny：任意一个子任务完成、失败或取消时即返回，且此函数本身返回的Task会为完成，即await此函数总是不会抛出异常；可以很容易地做到完成一个任务后取消其他的任务，或循环处理一部分完成了的`List<Task>`
* WhenAll：当参数是`Task<TResult>[]`时，此方法获得的返回值是`Task<TResult[]>`；如果出现异常，await它只会抛出未包裹的第一个异常

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

### ValueTask

* 需要nuget包：System.Threading.Tasks.Extensions
* 是struct类型的Task，不在堆上分配；有AsTask转换为普通的Task，但只能最多用一次，且之后就不要在与之前那个VT交互了
* 教程推荐在分析过用它会有性能改善的时候才用
* Task可以多次await，可以并发消费，可以保存到List里，可以用Whenxxx；而ValueTask不行，简单来说就只能直接await它（可用ConfigureAwait(false)），否则就转换为Task
* 当函数里的东西很可能能直接同步完成（hot path）时，最好使用这个。一种方式是函数不用写async，return时new ValueTask(result/Task.FromException)；或者像普通那样只把Task改成ValueTask，别的都不动。其实都和以前一样
* 在完成之前不能用GetAwaiter()
* 如果原本返回的是`Task<bool>`，没必要用VT，因为那两个结果对象会自动缓存

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

## SynchronizationContext

* GUI 应用运行的时候有默认的SynchronizationContext（同步上下文）。在异步操作结束的时候， 后续步骤的回调被交给该上下文，以让窗口处理完当前所有消息后执行该回调。这样你会看到异步操作前后都是在该窗口所在的（主）线程上。与STAThread无关，这个东西只和COM有关，完全没用
* 控制台应用并没有默认的SC。在这种情况下，异步操作结束后回调被交给默认的TaskScheduler处理，而默认的TS就是调用线程池。于是前后都是在不同的线程上
* 可以在类里面保存一下SynchronizationContext.Current，之后用Send和Post在UI线程上执行代码，区别是前者是同步的，会阻塞子线程，但会立即执行，而后者只是把回调加到消息循环里，且不会阻塞子线程
* SynchronizationContext.SetSynchronizationContext可以设置上下文
* 不知道为什么我做了一个SynchronizationContextRemover的笔记，但是这个东西是不存在于任何原生API中的
* WPF可用Application.Current.Dispatcher.Invoke，实际继承了SC

### 死锁与异步转同步

* WPF和UWP的UI线程都是单线程，如果在UI里使用了Wait()或Result，当异步方法完成后无法恢复到UI上，就会死锁，即对使用了t.ConfigureAwait也会死锁。如果实在要这样，应该使用返回Task的非async方法（方法中不要await它），最简单的就用Task.Run包一下就return
* 如果要用async：await Task对象.ConfigureAwait(false)可使得异步方法结束后不返回原来的上下文，而是新开一个线程执行剩下的代码，则不会死锁，也可能提高性能；一般如果不需要更新UI就可以这样用，比如连续两个await或者循环await。若要效果最大化，需要整个调用链都用（每次await，含caller的await）
* 在非UI线程上跑就不能去修改控件了，当你写的异步方法涉及到对于修改UI的委托的访问（即跨线程UI访问）时，不应使用这些方法。ASP.NET中最直接的影响就是HttpConext.Current的值为null；ASP.NET Core和CLI没有SynchronizationContext，用和不用没区别，相当于默认就是false；Standard因为可能被WPF使用，可以加
* 以上两种解决办法都是从库的角度出发的，假设UI为使用者用了Wait()
* 读写文件，访问网络等IO阻塞操作不会产生死锁，因为没有产生线程
* 还一种方式是用TaskCompletionSource

```c#
// WPF程序，点击按钮后的效果：文本不会改变，按钮不会禁用，直接无响应
// Winform会处理前两条语句然后无响应
private void btnDoStuff_Click() {
    btnDoStuff.IsEnabled = false;
    lblStatus.Content = "Doing Stuff";
    Thread.Sleep( 4000 );
    lblStatus.Content = "Not Doing Anything";
    btnDoStuff.IsEnabled = true;
}
```

### 同步转异步

* 可用委托的BeginInvoke加上TaskFactory.FromAsync

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
