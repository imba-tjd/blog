---
title: 枚举器和迭代器
---

> 《Illustrated C# 2012 (4th Edition)》
> 《C# in depth》

迭代器语法
----------

* foreach的循环变量是一个**只读**的局部变量
* 多次调用GetEnumerator应产生不同的迭代器，所以必须让一个和实现了IEnumerable平行的类实现IEnumerator，调用时new一个
* 仅获得迭代器、在开始迭代（调用MoveNext）前，迭代器的代码不会被执行。所以如果要对返回可迭代类型的方法进行参数检查时，需要在公开方法里检查，return私有方法/本地方法，再yield return。否则获取迭代器的时候不会报错，迭代时才报
* 执行遇到yield return时，程序返回一个值并暂停执行，下次在此处继续；yield生成的迭代器未实现reset；可用于属性
* 可以用try...finally...包裹迭代器块，遇到yield break或者正常结束或者手动释放迭代器或抛出异常后就会执行；如果存在catch块，则不能在try中用yield return
* 如果需要接受需手动释放的资源，可以选择公开接受创建资源的参数，私有重载创建资源，完成后自己释放
* 其他文章：https://blogs.msdn.microsoft.com/ericlippert/tag/Iterators/
* 应将迭代逻辑与操作、谓词和函数解耦
* 什么时候不应使用yield：https://stackoverflow.com/questions/3856625

### 返回类型为IEnumerable的函数

* 如果使用了yield return，会自动生成状态机，后面跟的是单个元素
* 在完全不使用yield的情况下，可以直接return一个IEnumerable类型
* 如果想返回空，可以用yield break或者Enumerable.Empty
* 如果接受一个IEnumerable，又仍想返回一层的IEnumerable：如果已经用了yield，只能foreach消费传入的再yield return；要不就返回两层的，再在外面包一层SelectMany；如果没有，可以用Concat/Append，如果是在开头添加，可以从Empty接上

IEnumerator\<T\>接口 : IEnumerator , IDisposable
------------------------------------------------

### T Current

* 只读属性
* 返回构造类型的引用

### object IEnumerator.Current

* 显式实现IEnumerator接口的方法
* 返回object类型的引用

### bool MoveNext()

* 把枚举器位置前进到集合中下一项的方法
* 如果新的位置是有效的，返回true，否则返回false
* 在第一次使用Current之前就被调用

### void Reset()

* 把位置重置为原始位置

### void Dispose()

* 实现IDisposable接口

IEnumerable\<T\>接口 : IEnumerable
----------------------------------

### IEnumerator\<T\> GetEnumerator()

* 返回枚举器类的实例

### IEnumerator IEnumerable.GetEnumerator()

* 显式实现IEnumerable接口

### ElementAt

* 因为没有索引，使用此方法的复杂度可能是o(n)

* * * * *

迭代器块
--------

* 有一个或多个yield语句
* 方法主体、访问器主体、运算符主体中的任意一种都可以是迭代器块
* yield return语句指定了序列中返回的下一项
* yield break语句指定咋序列中没有其它项
* 根据迭代器块的返回类型，你可以让迭代器块产生枚举器或可枚举类型

迭代器模式
----------

* 实现返回枚举器的迭代器时，必须通过实现GetEnumerator来让类可枚举
* 实现返回可枚举类型的迭代器时，需要实现返回IEnumerable\<T\>接口的方法，调用迭代器方法来使用由迭代器返回的可枚举类
* 也可以实现GetEnumerator，让它调用迭代器方法返回的可枚举类对象的GetEnumerator方法来让类本身可枚举
* 产生多个可枚举类型，只需实现多个返回IEnumerable\<T\>接口的不同的方法即可
* 可以将迭代器（返回IEnumerator\<T\>）作为属性，GetEnumerator方法根据判断返回指定的迭代器
* 在编译器生成的枚举器中，Reset方法没有实现

随机获取序列内容
----------------

* 在不知道序列中有多少元素的情况下，经过一次遍历就获取序列中随机一个元素，并且获取到每个元素的概率相等（为1/n）
* 伪代码见《杂项算法》

``` {.wp-block-preformatted}
public static T RandomElement<T>(this IEnumerable<T> source, Random random)
{
    if (source == null)
        throw new ArgumentNullException("source");
    if (random == null)
        throw new ArgumentNullException("random");

    ICollection<T> collection = source as ICollection<T>; // 如果实现了ICollection
    if (collection != null)
    {
        int count = collection.Count; // ICollection实现了Count属性
        if (count == 0)
            throw new InvalidOperationException("Sequence was empty.");

        int index = random.Next(count);
        return source.ElementAt(index);
    }

    // 如果没有实现ICollection，则不知道序列一共有多少元素
    using (IEnumerator<T> iterator = source.GetEnumerator())
    {
        if (!iterator.MoveNext())
            throw new InvalidOperationException("Sequence was empty.");

        int countSoFar = 1;
        T current = iterator.Current;
        while (iterator.MoveNext())
            if (random.Next(++countSoFar) == 0)
                current = iterator.Current;

        return current;
    }
}
```


