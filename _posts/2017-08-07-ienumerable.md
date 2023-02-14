---
title: 枚举器和迭代器
---

## 迭代器

* foreach的循环变量是一个**只读**的局部变量

## 实现可迭代对象的模式

* 多次调用GetEnumerator应产生不同的迭代器，所以必须让一个和实现了IEnumerable平行的类实现IEnumerator，调用时new一个
* 仅获得迭代器、在开始迭代（调用MoveNext）前，迭代器的代码不会被执行。所以如果要对返回可迭代类型的方法进行参数检查时，需要在公开方法里检查，return私有方法/本地方法，再yield return。否则获取迭代器的时候不会报错，迭代时才报
* 执行遇到yield return时，程序返回一个值并暂停执行，下次在此处继续；yield自动生成的迭代器未实现reset；可用于属性
* 可以用try...finally...包裹迭代器块，遇到yield break或者正常结束或者手动释放迭代器或抛出异常后就会执行；如果存在catch块，则不能在try中用yield return
* 支持using打开流，yield内容，会在消费后才关闭流，不会返回时就关闭

### 返回类型为IEnumerable的函数

* 如果使用了yield return，会自动生成状态机，后面跟的是单个元素
* 在完全不使用yield的情况下，可以直接return一个IEnumerable类型
* 如果想返回空，可以用yield break或者Enumerable.Empty
* 如果接受一个IEnumerable，又仍想返回一层的IEnumerable：如果已经用了yield，只能foreach消费传入的再yield return；要不就返回两层的，再在外面包一层SelectMany；如果没有，可以用Concat/Append/Prepend

## IEnumerator

* Current：只读，返回构造类型的引用
* MoveNext()
  * 把枚举器位置前进到集合中下一项
  * 如果新的位置是有效的，返回true，否则返回false
  * 应在在第一次使用Current之前就调用

## 迭代器模式

* 实现返回枚举器的迭代器时，必须通过实现GetEnumerator来让类可枚举
* 实现返回可枚举类型的迭代器时，需要实现返回IEnumerable<T>接口的方法，调用迭代器方法来使用由迭代器返回的可枚举类
* 也可以实现GetEnumerator，让它调用迭代器方法返回的可枚举类对象的GetEnumerator方法来让类本身可枚举
* 产生多个可枚举类型，只需实现多个返回IEnumerable<T\>接口的不同的方法即可
* 可以将迭代器（返回IEnumerator<T>）作为属性，GetEnumerator方法根据判断返回指定的迭代器
* 在编译器生成的枚举器中，Reset方法没有实现

不支持Py的yield from
