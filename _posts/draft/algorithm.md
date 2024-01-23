---
title: 算法
---

## 基础理论

### 循环不变式

1. 初始化：循环的第一次迭代前为真
2. 保持：如果循环的某次迭代前为真，那么下次迭代前仍为真
3. 终止

### 渐进记号

* O表示渐进上界，Ω表示渐进下界，Θ表示渐进确界
* 求解递归式：代入法、递归树法、主方法

#### 代入法

* 失败时可以考虑**减去**一个低阶项
* T(n)=2T(n/2)+n -> T(n)=O(nlgn)
* T(n)=2T(n/2)+c -> T(n)=O(n)
* T(n)=2T(sqrt(n))+lgn -> 令m=lgn, T(n)=O(mlgm)
* T(n)=9T(n/3)+n -> T(n)=O(n^2)
* T(n)=T(2n/3)+c -> T(n)=O(lgn)
* T(n)=8T(n/2)+n^2 -> T(n)=O(n^3)

### 在线雇佣

类似于选苹果的故事。在n个人中只雇佣一次，希望最大化质量，则做法为：选取一个正整数k<n，拒绝前面k个，雇佣其后面第一个比前面所有分数都高的人。k应取n/e，雇到最好的应聘者的概率至少为1/e。

### 模拟指针和对象的实现

* 多数组表示：每个属性都用一个数组表示，则相同属性储存在一起，每个数组用相同索引访问就是同一个对象的属性；此时模拟单向链表只需要一个next数组指示下一个结点的索引和一个全局的头索引
* 分配与释放：用一个全局的free值指示下一个空闲的索引位置，所有空闲的索引用next数组联系起来；分配时返回free当前指向的索引，并把free改成指向的指向的索引，释放时类似
* 单数组表示：把一个对象的所有属性储存在一起，这样允许不同长度的对象储存于同一数组中，但是管理更困难，访问属性需要用对象的位置再手动偏移

### 产生不重复随机数

* 1-m中取n个数，m较小时用洗牌算法；较大时用哈希表，重复了再抽一次
* 分布式可以用GUID
* 假随机可以在1到m-1中生成一个质数，每次取加自己模m的数，因为这是个循环群

## 分治法

1. 分解 (Divide) ：原问题分解为若干子问题，子问题是原问题的规模较小的实例
2. 解决 (Conquer) ：递归地求解子问题
3. 合并 (Combine) ：合并子问题

### 例子

* 大整数乘法
* 二分查找
* 矩阵乘法
* 归并排序
* x的n次幂：分奇偶，n是偶数就是f(n/2)相乘，是奇数就再乘以x或f(n/2+1)*f(n/2-1)
* 快速排序

### 最大子数组问题

例如：[13, -3, --25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7]，18到12之和即为最大子数组。此问题中部分元素必须小于0否则无意义；或者如果全是正的而要算变化值最大的，可以用后一个减去前一个，就会出负的

* 分治算法：取中点，任何一个子数列只有{完全位于左边、完全位于右边、跨过中间}三种情况，取三者最大值。前两者是原问题的较小规模，递归求解。第三个问题不是原问题，但容易求解，就是从中间往两边延伸，记录最大值
* dp：与常规化的不同。定义dp[i]为以nums[i]结尾的数组的最大子数组值，dp[i]=max{nums[i], dp[i-1]+nums[i]}，最后遍历一遍dp取最大值（不是返回dp[最后一个]）
* 在线处理：只需遍历一遍即可完成。从头开始加，记录下最大的时候；如果加到了小于0就之间把前面的部分全部舍弃，因为当前评估的值会抵消掉之前所有的和，直接不要了（最大的还是要记录着）

## 动态规划

### 原理

* https://www.zhihu.com/question/23995189/answer/613096905
* 最优子结构：一个问题的最优解包含其子问题的最优解。计算出子问题的最优解后通过状态转移方程计算原问题的最优解
* 重叠子问题：反复求解相同的子问题；同时也是与分治的区别
* 无后效性：后续决策不受历史决策过程影响，只依赖状态，不依赖到达此状态的步骤。如最长路径不具有最优子结构的性质。要求出解决问题使用了某一种方式不行，只求结果可以

### 步骤

1. 刻画一个最优解的结构特征。
2. 递归定义最优解的值。
3. 计算最优解的值，通常采用自底向上的方法。
4. 利用计算出的信息构造一个最优解。

另一种自底向上的思路：先根据已知条件把显而易见的算出来，再按一种方向把剩余的填充。

### 例子

* 常规化的定义：用dp[i]表示对于nums[0..i]满足“问题”的结果，假如已知dp[i-1]，分类讨论如何算出dp[i]，最后返回它即可
* 钢条切割、矩阵链乘：自底向上，f(x)=max/min{g(分割点i)}
* 0-1背包问题：价值 arr[重量]，arr[x]=max{arr[x-w(i)]+v(i)}；找钱问题：RMB可用贪心，但如面额只有11、5、1，找15时只能用DP，数量 arr[金额]
* 最长公共子序列（LCS）：对于两个序列X和Y来说，如果最后一个元素相等，则LCS去掉最后一个元素形成的序列Z-1是X-1和Y-1的LCS；如果不等，则X和Y的LCS=max{(X-1,Y), (X,Y-1)}，它有重叠子问题(X-1,Y-1)。一直到(X,0)和(0,Y)=空
* 最长递增子序列（LIS）：dp[x]表示**以第 x 元素为结尾的LIS**的长度，则=max{dp[i]+1, if arr[x]\>arr[i]}，这样复杂度为O(n^2)，还有[nlogn的做法](https://blog.csdn.net/joylnwang/article/details/6766317)；可以建树但想不清复杂度是多少

## 贪心算法

* 无状态，不会重新选择
* 局部最优解就是全局最优解：每次选择分支的时候，只要选择一个分支，这个分支的解就一定比其他选择更优

## 剑指Offer

* 二维数组中的查找：如一个二维数组从上到下递增，从左到右递增，在里面查找给定的数。只保证了左上小于右下，若当前值小于给定值，则给定值可能在右上、右下、左下；若大于，则在右上、左上、左下。正确做法是从最右上开始处理，若小于，则在左边，若大于，则在下面
* 用两个栈实现队列：入队列时压栈，出队列时先看另一个栈是否为空，如果非空则直接出栈，如果为空则把第一个栈里的放进去再按前一种情况。关键是再入队时无需把另一个栈放回去

---

## 杂项

把尾递归改成递推：https://zhuanlan.zhihu.com/p/36587160

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

一行GCD(答案结果为y)：`while(x^=y^=x^=y%=x);`


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

表格驱动法：逻辑和数据分离。添加数据和逻辑的方式和成本不一样。数据的添加是非常简单，低成本和低风险的；而逻辑的添加是复杂，高成本和高风险的。在单元测试中，逻辑必须测试，而数据无需测试。 在多人开发的项目中，逻辑无序而不可控（每个人写的风格不同）；数据格式有序而极易控制。


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

## Bloom Filter

* 它只能告诉我们一个元素绝对不在集合内或可能在集合内
* 可以节省时间和空间，但是有几率误判
* 无法取出原来的数据
* 插入：建立一个bit数组(m)，对于一个输入，使用多个(k)快速的Hash算法，每个获得一个索引，将对应的位置设为1
* 检测：如果所有的对应位置都为1，则可能在集合内；只要有一个位置不为1，就一定不在集合内
* Hash算法应当彼此独立且均匀分布
* 无法删除元素，改用int数组存count则可以删除

## 二项式定理

* 关键点是不能只写`binom 0 0 = 1`等那几个，那样递归太多

```hs
binom :: Integer -> Integer -> Integer
binom n 0 = 1
binom n k = if n == k then 1 else x + y
    where
    x = binom (n-1) k
    y = binom (n-1) (k-1)
```

## 化简分数

* 用(a,b)表示a/b
* (2,4) -> (1,2)，(-2,-4) -> (-1,2)，(2,-4) -> (-1,2)

```hs
normalize:: Fraction -> Fraction
normalize (0, b) = (0, 1)
normalize (a, b) = (x, y)
    where
    a' = abs a
    b' = abs b
    gcd' = gcd a' b'
    sign = div a a' * div b b'
    x = sign * div a' gcd'
    y = div b' gcd'
```

TimSort：
最好O(n)，最差O(nlogn)，但空间需求O(n)
https://zhuanlan.zhihu.com/p/50451255 https://www.zhihu.com/question/36280272

算法知识集合网站：
https://www.hello-algo.com/
https://labuladong.github.io/algo/

打点标记法处理整数区间：如24小时对应int[24]，未公开对应0，公开且空闲对应1，占用中对应2。
区间合并，如(1,2),(3,4)->(1,4)：先按起始值排序，遍历数据更新“当前区间”，当取出的和当前无交集时则当前的处理完毕。

题目中数据量很大(如10^9)时，也许要从另一个不那么大的数据量(如10^5)入手遍历，从总数中减去遍历计算的结果。
二分法：如一个值n在小于M时满足要求，求M（或求满足要求的n的最大值）。1必定满足，再宽松确定一个n的不满足的上界，然后看n/2。
