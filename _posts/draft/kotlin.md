# Kotlin

* @Volatile：代替关键字
* Xxx::class.java 等价于 Xxx.javaClass
* 1.5.roundToInt()
* try-catch、if、lambda 都是最后一个表达式作为返回值
* == 相当于py的，相当于equals()且兼容null。=== 相当于java的 ==

## 类型

* var可变，val不变，const val编译期常量
* Unit类型：相当于None，不返回值的函数会隐式返回Unit，也是个单例对象
* Any?：相当于Object
* Nothing：构造函数为私有的类，没有实例
  * 是所有类型的子类型，只有它这样是例外，因为kotlin不允许多继承
  * `val emptyList: List<Nothing> = listOf()`，可以赋值给任何`List<T> empty`
  * 是throw的返回值。TODO() 一个内置函数，里面就是抛异常
  * 是 return空 的返回值

## 属性委托

有一些对象，想让它们用起来像属性，但对getter/setter有一些常见的需求，已经标准化实现了，用by复用，左边对象的读写行为由右边对象实现。如果用=，就要用xxx.value。\
代理无法传递，如果把这个属性委托赋值给另一个同类型的普通变量（包括函数传参），后者无委托效果。

* val lazyValue: String by lazy { println("计算中...") "Hello" }
* var name: String by observable("初始值") { _, old, new -> println("$old -> $new") }
* var age: Int by vetoable(0) { _, _, new -> new >= 0 当此条件表达式为false时赋值会被阻止 }

接口委托：class C: 接口 by 对象，会自动转发调用对象实现接口。此对象可以被构造函数参数传入

## 字符串

* "$var"、"${a + b}"
* readline()、println()
* "123".toInt()、toIntOrNull()处理无效输入

## 集合

* 数组
  * arrayOf(变参)、intArrayOf() 代替new int[] { ... }
  * `mutableListOf<Int>()`
* Pair：val p = a to b; p.first; val (aa,bb) = p  其中to是个二元infix函数；后面还可以再继续连用to。第三句是解构，也可用在for中
* mapOf(a to b, c to d)。支持解构：for((k,v) in m)

## 循环

* for (i in 1..6) 闭区间。1..<6 左闭右开。称为range。不支持 大..小。支持"a".."z"、LocalDate
* for(i in 5 downTo 0) 闭区间。0 until 5 左闭右开。都可以配合step n正数
* for (i in arr.indices) 获得下标。arr.withIndex() 获得 (i,e) 类似于enunerate()
* repeat(n) it为ndx

## when

```kotlin
when(exp) { // 顺序执行
    match1, match2 -> body
    in 1..10 -> ...
    bool_exp() -> ...
    is String if xxx -> ... // 守护语句
    else -> ...
}

when { // 代替级联if-else
    比较语句 -> ...
}
```

## 空安全

* 默认非空
* Elvis运算符：val yyy = xxx ?: default。其中xxx可以是nullable表达式。default还可以写return -1或throw，则之后再用yyy相当于xxx不为null时的结果
* xxx!!.f()
* xxx?.let { f(it) } // 相当于if(xxx != null){ f(xxx) } 注意返回值是f的

## 类

* 无需new
* class MyClass(private val 构造函数参数名且作为成员变量: 类型) : 父类(调用父类构造函数参数)
  * init { ... } 每次实例化都会调用
  * 次构造函数 constructor():this() { ... }
  * 属性：var xxx get() = xxx set(value) { 验证; field = value } 所有字段本身就是property，自带getter/setter
* data class：自带toString、equals、解构
* 匿名对象：object { 成员字段和函数 }，第一种用法类似于C#的，第二种用法类似于Java的“new接口”
* 单例模式：object Sin { fun f }; Sin.f() 调用f时自动创建Sin单例。也可看作static class，Lazy性质也相同，即第一次用到类时加载所有成员，之后始终存活
  * companion object { ... } 类的伴生单例：达到类似于Java的static成员的效果，外部用起来也类似，但可以实现接口
* 类默认是final的。open class、open fun 表示可继承，子类的函数用override关键字。属性也可open
* sealed class：相当于ADT/Sum Type的概念，表示有限类型（如Result）。必须在里面创建data class且继承外面的类（如Success、Error），或外面的类的object单例（当不需要数据时）。而data class相当于Product Type
* 可见性：默认public。支持internal关键字
* annotation class

## 函数

fun print() = println(xxx)

函数类型：如 (Int) -> Unit。lambda：如 { x:Int -> ... }

把函数作为参数：用双冒号+函数名，其实是创建了一个对象。匿名函数和lambda是函数对象而不是函数

扩展函数：fun Int.isPrime() { this为Int }。如果写在class里就只在那个类里能用；如果想在类外用，要另外写一个函数，参数加函数，implicit receiver设为类，则大括号内就可用该扩展函数。支持 = 普通函数(第一个参数是那个receiver类型)。还有扩展属性

尾参lambda：如果一个函数的最后一个参数是lambda，如 ()->Xxx。则调用传实参时可以把lambda写在小括号外面且做一些省略，如 data.map { it.fun() }，相当于data.map(it->{it.fun()})
大括号内的this默认是外面作用域的，但创建者可指定receiver改变：`T.() -> Unit`，则调用者{}中的this就是T，称为implicit receiver。如果要用外面的，用this@Outer，其中Outer是类名，类似于Java的内部类

inline：对于高阶函数，加上能自动把参数也在调用方处展开，避免创建小对象；功能上，允许lambda的{}里有return且表示结束调用方的外层函数（不是lambda的返回值），因为展开后inline的函数就不存在了。还是泛型参数具化的基础。另一种性能优化：fun f = g，还可以再加@InlineOnly

return@label：普通lambda不能有return语句，而匿名函数可以有。如果lambda赋值了对象，label设为那个对象名，就可以return。函数中定义函数，label可指定外层函数名。在协程中label可指定launch等，Flow中指定操作符名。自定义标签`myLabel@`，相当于goto

## 函数式

### 作用域函数

* 用于创建对象后立刻设置它的成员：val c = C().apply { （隐含this.）f(1); m=2 } 此处的this就是前面的变量，且会返回它，相当于c = C(); c.f(1); c.m=2
  * 类似功能但不隐式改变this，而是显式用it：C().also { println(it) } 一般用于打印日志，要把刚创建的对象用作另一个函数的参数
* 隐含this，且返回值是最后一个表达式：D().run { f(); getData() } 整个表达式结果为d.getData()
  * 类似功能但用it：let。用于 空检查、把A转换成B
* 独立的隐含改变this：with(o) { f() } 相当于 o.f()
* 实现原理：implicit receiver

### result

```
val result: Result<Int> = Catching { "123".toInt() }

result.onSuccess { value ->
    println("成功: $value")
}.onFailure { exception ->
    println("失败: ${exception.message}")
}
```

## IO

* 扩展了Closeable：xxx.use { it }

## 泛型

* 使用时支持类上泛型的参数推断
* 可变性用in和out，可以加在接口上。无界通配符用星号
* 约束用冒号
* smart-cast：如`l: MutableList<Int>`，if (l is ArrayList) { l是`ArrayList<Int>` }
* 模板替换，不擦除：inline fun <reified T> f(a: Any) { if (a is T) println(a) }

## 协程

```kotlin
import kotlinx.coroutines.* // kotlinx-coroutines-core，安卓再加kotlinx-coroutines-android

fun main() {
    println("普通函数")
    runBlocking { // 阻塞当前线程直到协程完成。里面允许调用suspend函数。一般用main=它。另一种用法：普通函数调用协程回调，它把协程封装成阻塞的
        mySuspendFunction()
    }
    println("普通函数结束")
}

suspend fun mySuspendFunction() {
    delay(1000) // 调用另一个suspend函数，挂起等待，让出线程
    println("挂起函数")
}

// 不等待，返回一个future，会往下走
val job = launch { 调用无返回值的suspend函数 }; job.join()
val deferredResult = async { 调用有返回值的suspend函数，最后一个表达式的值作为返回值 }; val result = deferredResult.await()

// 依赖另一个协程的结果的一种pattern：
val result = async { ... }; await { result.await() }

// Dispatchers预定义了几个线程池。Default用于CPU计算，数量等于CPU核心数。IO线程池数量>=64。Main主线程仅安卓能用，但一般会用runOnUiThread{}
withContext(Dispatchers.IO) {
    临时切换线程池，执行完后会切回去。一般会与里面的代码一起放在某suspend函数里，而不是由调用方决定ctx
    withContext是等待的，最后一个表达式作为返回值。launch也有参数可以切池，但非等待
    Dispatcher基本又叫ContinuationInterceptor，它们的父接口是CoroutineContext。还有一些其它功能完全不同的类也是Context，它们可以用+合并功能
    获取当前ctx：currentCoroutineContext()
}


// 结构化并发：批量管理多个协程，保证生命周期结束时里面的协程都结束了
Scope是CoroutineContext的容器，包括 线程池、异常处理（Handler）、失败传播/隔离（SupervisorJob()）、名字
CoroutineScope：创建全新scope，要指定ctx，隔离性大。scope.launch { 调用suspend函数 } 非等待，最后join()
coroutineScope{}：挂起父协程、统一等待子协程结束的子scope，本质上是新建了Job()。withContext就是临时修改ctx的coroutineScope
    launch也会新建scope，但不挂起父协程。launch{launch{}}.join() 没有等待内层执行完，外层就已经结束了
    try{coroutineScope{}}catch{} 能正常工作，而launch不行。如果没catch，仍会向上传播
GlobalScope：生命周期无限的Scope，使用EmptyCoroutineContext，没有Job()无法cancel，不推荐用。main加suspend就是它。安卓中如果用了它且引用了某Activity的资源，容易造成泄露，因为即使Activity关闭了还会持有

// 异常处理
使用async启动的协程，异常会被收集，在await()时重新抛出。而launch无此功能，会向上一直抛到根。无监管scope会取消所有子协程
如果协程内catch一个Exception，要特别处理放过CancellationException
处理“未捕获异常”：val handler = CoroutineExceptionHandler { _, throwable -> ... }; CoroutineScope(handler)
    此handler会自动放过CancellationException。即使安装了handler，对于无监管scope仍然会整个取消
supervisorScope{}：一个子协程抛异常，不会影响别的协程；但cancel和普通的一样。且可以在launch里安装handler
cancel：会取消它和它的子协程，而对父协程无影响

// 转换普通函数为挂起函数
suspend fun f() = suspendCancellableCoroutine { // 可有返回值
    it.invokeOnCancellation { 如果g支持取消，在这里做取消时的收尾工作 }
    try { val r= g(); it.resume(r) }
    catch (e) { it.resumeWithException(e) }
}
```

thread {} 相当于Java的new Thread(runnable).start()

## Flow

相当于IAsyncEnumerable。

```kotlin
import kotlinx.coroutines.flow.*

val intFlow = flow<Int> { // 也可自动推断
    delay(1000)
    for(i in 1..10) emit(i) // (1..10).asFlow()
    // 这里面不允许切换到别的协程如 launch{ emit() } 因为要调用者决定在哪个scope里运行
}
// 此处是cold flow：没有消费时不会执行。每次执行终结符会从头产生

intFlow.collect { println(it) 每次emit都会执行本块 }
```

### Operator

* 大部分会返回最后一个表达式的值
* map、transform 用emit命令式发数据到下游
* chuncked(n)
* collectIndexed()。withIndex() 在中间返回 (index, value)
* 过滤
  * filter
  * filterNot、filterNotNull、filterIsInstance
  * distinctUntilChanged[By] 去连续重。其中By相当于py取key的方式，无By的有双参重载
  * drop dropWhile take takeWhile
* onEach 使用it而不改变，一般是记录日志
  * collect 等效于 onEach().launchIn(scope)。还有flowOn()切换上游线程，类比withContext
* 其它终结符
  * first、firstOrNull、last、single、count
  * toList、toSet 可选用参数指定已有集合。toCollection必须指定已有
  * reduce {acc, curv}、fold相当于reduce但能提供初始值
* 处理背压
  * 默认情况下，某条数据被collect消费完了，生产端emit才执行完，整体在collect的协程中“同步”执行
  * buffer(n) 默认64。变为异步，它会创建一个新协程运行上游代码
  * collectLatest：如果产生速度大于消费速度，当新产生时消费还未结束，会cancel。内置了buffer
  * conflate() 可理解为缓冲区容量为1，但生产者永不挂起，新值直接覆盖旧值。实际是buffer(0+溢出策略)
  * debounce(ms) 指定毫秒内只能通过一个。sample(ms) 取指定时间内的随机一个
* 合并多个流
  * combine和merge有顶层函数重载，支持多个流
  * combine 最新值的即时组合。每个流必须至少发射过一次，然后任意一个流产生新值，就再次触发，与其它流的旧值组合。还有combineTransform
  * merge 语义为Fan-In，多个流不管谁发了数据，统一产生为一个对象
  * zip 等待两个流的新数据
  * 铺平流中流：flattenConcat、flattenMerge 其中Concat等前一个流处理完了再处理后面的流。Merge并发处理，并发度默认16
  * 另一种更常见的铺平：并不是一开始就有`Flow<Flow<T>>`，而是先有`Flow<id>`，对单个id的处理返回了Flow，又要铺平
    * flatMapConcat、flatMapMerge：flow1.flatMapConcat { flow { emit } }
    * 在UI中只需要最新的：flatMapLatest，当flow1产生新值时会取消前面的map
* select { job1.onJoin { 返回值作为select的值 }; job2... }
* timeout 每条都会计时，超时了抛异常

### 异常处理

* 发射端产生异常，一般不处理。如果try{emit()}，会捕获下游（如collect里的）异常
* 中间catch(e->...)，能捕获到上游的异常，然后“接管流程”。它会忽略发送端emit抛出的来自下游的异常，称为异常透明性。可以emit数据到下游；如果不emit，下游就收不到
* retry(n)：如果上游发生异常，就重启前面的流。也是接管。还有retryWhen
* 收集端一般用普通的try-catch
* onCompletion { cause -> } 相当于finally，但要在collect前调用，即设定回调。如关闭数据库连接时使用，且不用调用者清理。cause如果为null表示无异常。如果协程被取消，仍然会执行
* onStart：每次最上游emit之前执行。如果有多个，后设定的先执行。这里抛的异常，try{emit()}捕获不到因为在它的上游，而catch()可以

### StateFlow

* 热流。它始终有一个值，收集者接入时获得最新值。值变化时通知收集者；如果赋相同值，不会动。类似于容量为1的缓冲区
* 用法：创建者 private val _state = MutableStateFlow(0); val state = _state.asStateFlow(); 每当_state.update{}，state就会触发。如果消费速度慢于改变速度，看起来就像中间的值丢失了。但collectLatest是在消费端，而它在生产端，且不会cancel收集者
* 某个State是从多个值一起处理得到，任何一个变化都要重新计算：combine
* 用于状态订阅，一般写在VM里。替代LiveData，但没有Activity生命周期感知
* 在ComposeUI层从VM使用，一种封装了的消费方式：state.collectAsState(initial=0)

### SharedFlow

* 类似于事件订阅机制、发送一次性通知，支持多个collect消费（一对多）
* 也是热流，是StateFlow的底层
* 没有初始值，可选重放数、缓冲数
* 创建：普通flow.shareIn(scope/this)。另一种类似于事件的方式：`MutableSharedFlow<T>()`，“外部”可以对它emit

### Channel

Flow的底层，用于跨协程传递数据。

* val ch = produce { send(1) } 创建channel，内部可以send，返回ReceiveChannel。相对应的有actor，内部消费，返回值为能send的ch
* for (data in ch)
* flow1.produceIn(scope) 把flow转换为ch
