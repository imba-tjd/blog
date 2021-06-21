---
title: 析构函数和Dispose方法
category: dotnet
---

## Using语句

* 凡是继承了IDisposable接口的类，都可以使用using语句。在超出作用域后会自动调用Dispose方法
* using语句可保证一定会调用Dispose方法，即使发生异常。因为其实就相当于try finally
* 如果不清楚某个对象是否实现了IDisposable，可以用as，using(null)是没有问题的，不会产生效果也不会报错；但如果有两个using就不要这样用了，第二个构造函数抛异常时第一个不会Dispose
* 如果一次性使用的using太多，还不如自己写成try finally，就只用写一层；`A a; try...finally a?.Dispose();`；或者用C#8的using语法

## 析构函数

* 析构函数既没有修饰符，也没有参数
* 一个类只能有一个析构函数
* 不能在结构中定义析构函数，只能对类使用析构函数
* 无法继承或重载析构函数
* 无法显式调用析构函数，它们是被自动调用的
* 对于子类，析构函数的调用次序与他们对应的构造函数的调用次序刚好相反
* 不要在不需要的时候实现析构函数，会影响性能
* 析构函数不应该访问其他对象，因为无法认定这些对象是否已经被销毁
* 不能在析构函数中释放托管资源，因为析构函数是由垃圾回收器调用的，可能在析构函数调用之前，类包含的托管资源已经被回收了
* 析构函数在编译后自动变成Finalize方法，避免自己定义同名方法
* 唯一的用处：在类的使用者没有调用Dispose的时候，由GC释放非托管资源，算是保险

## 标准dispose模式

* 如果需要主动释放资源，应实现IDisposable接口，在代码中调用Dispose方法；应实现virtual的Dispose(bool)重载，以便子类重写和调用
* 析构函数只释放非托管资源，Dispose方法释放托管和非托管资源。标准模式下由析构函数调用`Dispose(false)`即可
* 如果子类有自己的资源要释放，就重写虚方法，否则不用重写；先清理自己，之后调用父类的同名函数
* 重复调用Dispose是没有问题的，此方法在首次调用后应该什么也不做
* 如果没有显式调用Dispose方法，垃圾回收器也可以通过析构函数（来调用Dispose）来释放非托管资源；垃圾回收器本身又具有回收托管资源的功能，所以能保证资源的正常释放。只不过这样会导致非托管资源释放不及时
* 资源释放后，再引用实例方法会引发ObjectDisposedException
* 如果只在类的内部使用实现了Dispose方法的对象，则不用实现析构函数，只在释放托管资源的部分调用对应对象的Dispose就行，如sql连接
* 应只进行清理工作，不要把this又添加到别的地方去，否则会“复活”
* C#8引入了IAsyncDisposable，实现时调用`await DisposeAsyncCore();`和`Dispose(false);`，在前者中继续异步Dispose具体的对象且用`ConfigureAwait(false)`，调用者用await using
* 按照我的理解，正确实现了IDisposable的类就变成托管对象了
* 语义上，Dispose后就不能再使用了。如果是close，也许还可以设计成能重新open

```c#
using System;
class BaseClass : IDisposable {
    // Flag: Has Dispose already been called?
    bool disposed = false;

    // Public implementation of Dispose pattern callable by consumers.
    public void Dispose() {
        Dispose(true);
        GC.SuppressFinalize(this); // 阻止调用析构函数；GC.ReRegisterForFinalize可恢复析构请求，GC.WaitForPendingFinalizers
    }

    // Protected implementation of Dispose pattern.
    protected virtual void Dispose(bool disposing) {
        if (disposed)
            return;

        if (disposing) {
            // 释放由该类自己创建的的托管对象，调用?.Dispose()，用于及时释放内存
        }

        // 释放非托管对象
        CloseHandle(handle); // 需PInvoke，见代码段末尾
        handle = IntPtr.Zero;

        // 把占用大量内存或使用短缺资源的托管对象设为null，如MemoryStream，这使得它们更容易变得不可达

        disposed = true;
    }

    ~BaseClass() => Dispose(false); // 只释放非托管资源
}

class DerivedClass : BaseClass { // 子类不应主动实现IDisposable接口
    bool disposed = false;

    protected override void Dispose(bool disposing) {
        if (disposed)
            return;

        if (disposing) {
            // ...
        }

        // ...

        disposed = true;

        // Call the base class implementation.
        base.Dispose(disposing);
    }

    ~DerivedClass() { Dispose(false); }
}

[System.Runtime.InteropServices.DllImport("Kernel32")]
private extern static Boolean CloseHandle(IntPtr handle);
```

### 常见的非托管资源

这些类已经实现了析构函数和Dispose方法，我们在自己的类中使用时只要实现Dispose即可，不需要实现析构函数

* ApplicationContext、Component、ComponentDesigner、Tooltip、Container、Cursor
* FileStream、文件句柄、Socket
* Font、Icon、Image、Brush、Pen
* Matrix
* Timer
* GDI资源
* 数据库连接、DataReader

## 避免频繁创建相同的对象

* 避免循环内使用using，可以改成readonly成员变量，再在类里实现Dispose
* static 成员惰性求值：Brush类。属性先检查后备字段是否为null，如果是则new，否则返回之前创建好的。这样如果没有用到就不会创建
* 对于不可变类型，可以创建builder类

## 参考

* http://blog.csdn.net/u014563989/article/details/53172707
* http://blog.csdn.net/u014563989/article/details/53172332
* http://www.cnblogs.com/tsoukw/archive/2006/12/08/586525.html
* https://docs.microsoft.com/zh-cn/dotnet/standard/garbage-collection/implementing-dispose
* 《Illustrated C# 2012 (4th Edition)》
* http://www.myexception.cn/c-sharp/1515938.html

### TODO

* http://www.myexception.cn/c-sharp/1515938.html
