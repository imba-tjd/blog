---
title: 杂项算法
---

计算π的值
---------

* [http://blog.csdn.net/xfxyy_sxfancy/article/details/48378121](http://blog.csdn.net/xfxyy_sxfancy/article/details/48378121)
* [https://www.zhihu.com/question/20756479](https://www.zhihu.com/question/20756479)
* pi/2 = 1 +∑( πn / π(2n+1) )

获取序列中随机一个元素
----------------------

此算法可以在不知道序列长度的前提下，只用一次遍历就获取序列中随机一个元素，并且获取到每个元素的概率相等，任何时候都为1/n，其中n为当前遍历到的数量。原理是被获取的那个数(n)需要在中间被选到(1/n)并且之后(n+1)不被替换(n/n+1)，之前的不用管。

```
Item GetRandomItemInSequence(Source source){    int countSoFar = 1;    Item current = source.GetNext();    if(current == null)        return ERROR;    Item temp;    while((temp = source.GetNext()) != null)        if(random(++countSoFar) == 0)            current = temp;    return current;}
```

递归排列一些数字
----------------

> https://bbs.csdn.net/topics/392059014

此方法使用递归从后往前处理，把数字分为自己和递归获取到的，自己从t1到t2进行迭代，再和递归到的进行拼接；p指的是需要几位数字，predicate指定的是每一位都不相同。

```
using System;
using System.Linq;
using System.Diagnostics;
using System.Collections.Generic;
class Permutation
{
    static IEnumerable<T> Prepend<T>(T first, IEnumerable<T> rest)
    {
        yield return first;
        foreach (var item in rest)
            yield return item;
// 无法写成return rest.Any()?Prepend(rest.First(), rest.Skip(1)):Enumerable.Empty<T>();
// 因为yield块中无法使用return，只能用yield return和yield break，否则会破坏迭代器直接消费下一个
    }
    static IEnumerable<IEnumerable<int>> M(int p, IEnumerable<int> source, Func<int, IEnumerable<int>, bool> predicate)
    {
        if (p == 0)
            yield return Enumerable.Empty<int>();
        else
            foreach (var first in source)
                foreach (var rest in M(p - 1, source, predicate))
                    if (predicate(first, rest))
                        yield return Prepend(first, rest);
    }
    static void Main()
    {
        Stopwatch sw = new Stopwatch();
        int count = 0;
        sw.Start();
        foreach (var sequence in M(4, Enumerable.Range(1, 9), (first, rest) => !rest.Contains(first)))
        {
            // Console.WriteLine(string.Join(", ", sequence));
            count++;
        }
        sw.Stop();
        Console.WriteLine("共{0}种组合，耗时{1}ms", count, sw.ElapsedMilliseconds);
    }
}
```

检测一个表达式是否是数字常量
----------------------------

> https://www.zhihu.com/question/36336058

```
define ICE_P(x) (sizeof(int) == sizeof((1 ? ((void)((x) * 0l)) : (int*)1)))
```

解整数方程
----------

* 把一部分多项式移到等式右边，左边遍历计算所有值加到哈希表中，右边遍历计算所有值判断是否在哈希表中；在变量较多的时候，可以把10*10*10*10这样的变成10*10+10*10

二分查找
--------

* mid = first + (last - first) / 2，这样不会溢出

合并两个已经排序好的数组
------------------------

```
static IEnumerable<int> MergeTwo(IEnumerable<int> a, IEnumerable<int> b)
{
    IEnumerator<int> ea = a.GetEnumerator();
    IEnumerator<int> eb = b.GetEnumerator();
    if (ea.MoveNext() != false && eb.MoveNext() != false)
        while (true)
        {
            if (ea.Current < eb.Current)
            {
                yield return ea.Current;
                if (ea.MoveNext() == false)
                {
                    yield return eb.Current;
                    break;
                }
            }
            else
            {
                yield return eb.Current;
                if (eb.MoveNext() == false)
                {
                    yield return ea.Current;
                    break;
                }
            }
        }
    while (ea.MoveNext() != false)
        yield return ea.Current;
    while (eb.MoveNext() != false)
        yield return eb.Current;
}
```

把集合按数量依次多一的方式返回
------------------------------

```
static IEnumerable<IEnumerable<int>> GetSeq(IEnumerable<int> collection)
{
    int step = 0;
    int count = 0;
    while ((count = step * (step + 1) / 2) < collection.Count())
        yield return collection.Skip(count).Take(++step).OrderBy(x => x);
}
```

Base64
------

* 使用A-Za-z0-9和加减表示，原文非3字节倍数的补0并在编码后加等号
* 无法缓存，除非缓存整个网页，但HTML/CSS可能变化很快
* 大小变为4/3，不过gzip后差不多

一行GCD(答案结果为y)
--------------------

```
while(x^=y^=x^=y%=x);
```

生成素数
--------

```c#
static IEnumerable<int> GetEvens() // 获取从3开始所有的奇数
{
    int i = 1;
    while (true)
        yield return i += 2;
}
static IEnumerable<int> GetPrimes()
{
    yield return 2;
    var nums = GetEvens();
    while (true)
    {
        int n = nums.First();
        yield return n;
        nums = nums.Where(x => x % n != 0); // n会被捕获；且因为是赋给自己，会链式调用
    }
}
public static void Main()
{
    var list = GetPrimes();
    foreach (var item in list.Take(10)) // 能获取到无限的素数
        Console.WriteLine(item);
}
```

把数组按负数、0、正数顺序排列
-----------------------------

```
typedef enum{negative, zero, positive} ntype;
void Arrange(int a[], int n)
{
    int i=0,j=0,k=n-1; // j为当前元素，i之前的全为负数，i到j为0，k以后的全为正数
    while(j<=k) // 时间复杂度为O(n)
    switch(a[j]){
        case negative: Swap(a[i],a[j]);i++;j++;break;
        case zero: j++;break;
        case positive: Swap(a[j],a[k]);k--;
        // 最后一句没有j++，以防止交换后a[j]仍为正数的情况；负数时不用考虑因为i已经评估过了，而k没有
    }
}
```

牛顿迭代法求平方数
------------------

![]({{%20site.baseurl%20}}/assets/img/e59bbee78987.png)

这种算法的原理很简单，我们仅仅是不断用(x,f(x))的切线来逼近方程x^2-a=0的根。根号a实际上就是x^2-a=0的一个正实根，这个函数的导数是2x。也就是说，函数上任一点(x,f(x))处的切线斜率是2x。那么，x-f(x)/(2x)就是一个比x更接近的近似值。代入 f(x)=x^2-a得到x-(x^2-a)/(2x)，也就是(x+a/x)/2。

```
float SqrtByNewton(float x) {
    float val = x;//最终
    float last;//保存上一个计算的值
    do
    {
        last = val;
        val =(val + x/val) / 2;
    }while(abs(val-last) > eps);
    return val;
}
```

AC自动机
--------

* 他们的总结是在Trie树上跑KMP
* https://zhuanlan.zhihu.com/p/80325757
* https://www.cnblogs.com/jason2003/p/9651073.html
* https://blog.csdn.net/bestsort/article/details/82947639

[表格驱动法](https://www.zhihu.com/question/37943171)
-----------------------------------------------------

表格驱动的意义在于：**逻辑和数据分离。**在程序中，添加数据和逻辑的方式是不一样的，成本也是不一样的。简单的说，**数据**的添加是非常**简单**，**低成本和低风险**的；而**逻辑**的添加是**复杂**，**高成本**和**高风险**的。 在单元测试中，逻辑必须测试，而数据无需测试。 在多人开发的项目中，逻辑无序而不可控（每个人写的风格不同）；数据格式有序而极易控制。

不过毕竟是增加了一层，如果是不易变化的，用原来的方式也行。

```
// PHP
function contry_initial($country){
    if ($country==="China" ){
       return "CHN";
    }else if($country==="America"){
       return "USA";
    }else if($country==="Japan"){
      return "JPN";
    }else{
       return "OTHER";}}

// 改进：
function contry_initial($country){
  $countryList=[ // 容易添加和修改，可以再改成传入参数
      "China"=> "CHN",
      "America"=> "USA",
      "Japan"=> "JPN",
    ];

    if(in_array($country, array_keys($countryList))) {
        return $countryList[$country];
    }
    return "Other";
}

// JS
function doSomething(a) {
    var lookup = {x: doX, y: doY}, def = doZ; //数据
    (lookup[a] || def)();                     //实现
}
```

## 一种避免goto的方法

```
A:
    f1()
B:
    if c1 goto A
    f2()
    if c2 goto B
end

# 改成
f1() # 手动重复一次，c1就变为入口条件了
do {
    while c1
        f1()
    f2()
} while c2
```
