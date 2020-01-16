---
title: 调试类
---

> 《C# 6.0 学习笔记：从第一行代码到第一个项目设计》

禁用断言失败时弹出错误提示框
----------------------------

    // 用代码实现
    using System.Diagnostics;
    var listener = Debug.Listeners[0] as DefaultTraceListener;
    listerner?.AssertUiEnabled = false;

    // App.config，在configuration标签中插入
    <system.diagnostics>
        <assert assertuienabled = "false">
    </system.diagnostics>

Assert类的静态方法
------------------

* Assert.Inconclusive() 表示一个未验证的测试
* Assert.AreEqual()，Assert.AreNotEqual()
    测试指定的值是否相等，如果相等（不相等），则测试通过
* Assert.AreSame()，Assert.AreNotSame()
    测试指定的两个对象变量是否指向相同（不同）的对象，如果相同（不同），则测试通过
* Assert.IsTrue()，Assert.IsFalse()
    测试指定的条件是否为True（False），如果为True（False），则测试通过
* Assert.IsNull()，Assert.IsNotNull()
    测试指定的对象是否为空引用，如果为空（不为空），则测试通过
* Asset.IsInstanceOfType，Asset.IsNotInstanceOfType 测试对象是否为指定类型的实例，如果是（不是），则测试通过
* Asset.Fail 不进行任何检查，直接报告断言失败

使用日志文件
------------

Debug类和Trace类都有一个静态的Listeners属性，它是一个集合，里面可用包含实现了TraceListener抽象类的对象。DefaultTraceListener类是最常用的类，在程序运行后，一般会生成一个DefaultTraceListener实例，该类可用直接使用。

    using System.Diagnostics;
    var listener = Debug.Listeners[0] as DefaultTraceListener;
    if(listener == null)
    {
        listener = new DefaultTraceListener();
        Trace.Listeners.Add(listener);
    }
    listener.LogFileName = "example.log";

    Trace.Print("日志");
