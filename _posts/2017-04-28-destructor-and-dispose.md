---
title: 析构函数和Dispose方法
---

> 部分参考资料：
>  [http://blog.csdn.net/u014563989/article/details/53172707](http://blog.csdn.net/u014563989/article/details/53172707)
>  [http://blog.csdn.net/u014563989/article/details/53172332](http://blog.csdn.net/u014563989/article/details/53172332)[http://www.cnblogs.com/tsoukw/archive/2006/12/08/586525.html
> ](http://blog.csdn.net/u014563989/article/details/53172332)[https://docs.microsoft.com/zh-cn/dotnet/standard/garbage-collection/implementing-dispose
> ](https://docs.microsoft.com/zh-cn/dotnet/standard/garbage-collection/implementing-dispose)《Illustrated C\# 2012 (4th Edition)》

ToRead:
 [http://www.myexception.cn/c-sharp/1515938.html
 http://www.cnblogs.com/cxd4321/archive/2006/10/19/533265.html
 http://blog.csdn.net/alicehyxx/article/details/12948121
 http://stackoverflow.com/questions/732864/finalize-vs-dispose
](http://www.myexception.cn/c-sharp/1515938.html)

Using语句
---------

-   using语句可保证一定会调用Dispose方法，即使发生异常。因为其实就相当于try finally
-   如果不清楚某个对象是否实现了IDisposable，可以用as；using(null)是没有问题的，不会产生效果也不会报错；但如果有两个using就不要这样用了，第二个构造函数抛异常时第一个不会Dispose
-   如果一次性使用的using太多，还不如自己写成try finally，就只用写一层；A a; try...finally a?.Dispose();；或者用C\#8的语法

析构函数
--------

-   析构函数既没有修饰符，也没有参数
-   一个类只能有一个析构函数
-   不能在结构中定义析构函数，只能对类使用析构函数
-   无法继承或重载析构函数
-   无法显式调用析构函数，它们是被自动调用的
-   对于子类，析构函数的调用次序与他们对应的构造函数的调用次序刚好相反
-   不要在不需要的时候实现析构函数，这会严重影响性能
-   析构函数不应该访问其他对象，因为无法认定这些对象是否已经被销毁
-   不能在析构函数中释放托管资源，因为析构函数是由垃圾回收器调用的，可能在析构函数调用之前，类包含的托管资源已经被回收了
-   唯一的用处：在用户没有调用Dispose的时候，由GC释放非托管资源。算是保险吧，会影响性能

标准dispose模式
---------------

-   如果需要主动释放资源，应实现IDisposable接口，在代码中调用Dispose方法；应实现virtual的Dispose(bool)重载，以便子类重写和调用
-   析构函数只释放非托管资源，Dispose方法释放托管和非托管资源
-   如果子类有自己的资源要释放，就重写虚方法，否则不用重写；先清理自己，之后调用父类的同名函数
-   重复调用Dispose是没有问题的，此方法在首次调用后应该什么也不做
-   如果没有显式调用Dispose方法，垃圾回收器也可以通过析构函数（来调用Dispose）来释放非托管资源；垃圾回收器本身又具有回收托管资源的功能，所以能保证资源的正常释放。只不过这样会导致非托管资源释放不及时
-   凡是继承了IDisposable接口的类，都可以使用using语句。在超出作用域后，系统会自动调用Dispose方法
-   资源释放后，再引用实例方法会引发ObjectDisposedException
-   如果只在类的内部使用实现了Dispose方法的对象，则不用实现析构函数，只在释放托管资源的部分调用对应对象的Dispose就行，如sql连接
-   释放非托管资源：CloseHandle、handle = IntPtr.Zero
-   应只进行清理工作，不要把this又添加到别的地方去，否则会“复活”

```
using System;

class BaseClass : IDisposable
{
    // Flag: Has Dispose already been called?
    bool disposed = false;

    // Public implementation of Dispose pattern callable by consumers.
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    // Protected implementation of Dispose pattern.
    protected virtual void Dispose(bool disposing)
    {
        if (disposed)
            return;

        if (disposing)
        {
            // Free any other managed objects here.
            //
        }

        // Free any unmanaged objects here.
        //
        disposed = true;
    }

    ~BaseClass()
    {
        Dispose(false);
    }
}

class DerivedClass : BaseClass
{
    bool disposed = false;

    // Protected implementation of Dispose pattern.
    protected override void Dispose(bool disposing)
    {
        if (disposed)
            return;

        if (disposing)
        {
            // Free any other managed objects here.
            //
        }

        // Free any unmanaged objects here.
        //
        disposed = true;

        // Call the base class implementation.
        base.Dispose(disposing);
    }

    ~DerivedClass()
    {
        Dispose(false);
    }
}

[System.Runtime.InteropServices.DllImport("Kernel32")]
private extern static Boolean CloseHandle(IntPtr handle);
```

常见的非托管资源？
------------------

理论上这些已经实现了析构函数和Dispose方法，我们在自己的类中使用时只要实现Dispose即可，不需要实现析构函数

-   ApplicationContext
-   Brush
-   Component
-   ComponentDesigner
-   Container
-   Context
-   Cursor
-   FileStream
-   Font

避免频繁创建相同的对象
----------------------

-   循环内使用using，可以改成readonly成员变量，再在类里实现Dispose
-   static 成员惰性求值：Brush类。属性先检查后备字段是否为null，如果是则new，否则返回之前创建好的。这样如果没有用到就不会创建
-   对于不可变类型，可以创建builder类

ToRead
------

> （这是一个规则）如果一个类拥有一个实现了IDispose接口类型的成员，并创建（注意是创建，而不是接收，必须是由类自己创建）它的实例对象，则这个类也应该实现IDispose接口，并在Dispose方法中调用所有实现了IDispose接口的成员的Dispose方法。
> 只有这样的才能保证所有实现了IDispose接口的类的对象的Dispose方法能够被调用到，确保可以手动释放任何需要释放的资源。
>
> 对象变为不可访问后，将自动调用此方法，除非已通过 GC.SuppressFinalize 调用使对象免除了终结。在应用程序域的关闭过程中，对没有免除终结的对象将自动调用 Finalize，即使那些对象仍是可访问的。对于给定的实例仅自动调用 Finalize 一次，除非使用 GC.ReRegisterForFinalize重新注册该对象，并且后面没有调用 GC.SuppressFinalize。
> 对于 隐式实现 来说，你只需要调用 “new ClassA().Dispose()“，但是对于显式实现来说，dispose()不会是这个 classA 的成员函数。唯一的调用方式是先强制类型转换到 IDisposable ,即”new ClassA().Dispose()”,但是((IDisposable)new ClassA()).Dispose() 可以编译过。这样就符合了设计的要求：提供 close()，隐藏dispose(),并且实现 IDisposeable接口
> C\#中所有的类都隐式继承自object类，object类内包含了一个特殊的方法Finalized()，所有的类都可以覆盖它。.NET中的垃圾回收系统在对类的实例进行资源释放前会首先调用这个方法。需要注意的是，一旦我们为一个类显式地声明了析构函数，那么这个类在编译期间，会由编译器自动产生一个Finalized()方法。也就是说，如果我们为某个类明确地指明了析构函数，那么就不能再在类中覆盖Finalized()方法了。
> 当变量超出其说明的作用域是，析构函数被隐式调用。局部变量的析构函数在其说明的块不再激活时调用；而对于全局变量，析构函数在main()函数作为退出过程的一部分时被调用。当指向对象的指针超出作用域时，析构函数不被隐式调用，这就是说，为了删除这种对象，必须使用delete操作符。


