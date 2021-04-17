---
title: 类的转换和模式匹配
---

> 《Illustrated C# 2012 (4th Edition)》

值类型转换
----------

### double转到float

* 如果值太小，那么值会被设置为正0或负0
* 如果值太大，那么值会被设置为正无穷大或负无穷大

### float或double转换到整数类型

* 如果用强制转换，值会舍掉小数，截断为最近的整数；负数也是如此
* 强制转换如果转换后的值不在目标类型的范围内（溢出）：如果溢出检测上下文是checked，则CLR会抛出OverflowException异常；如果上下文是unchecked，则未定义
* 如果用Convert类，会发生四舍五入。溢出时必定抛异常

类型转换帮助器
--------------

### Convert类

* ChangeType方法：将实现了IConvertible接口的类型转换为指定的类型
* 以To开头的方法：将参数中传入的对象转换为指定的类型对象
* ToBase64String/CharArray, FromBase64String/CharArray：前两者先要用Encoding类转换成byte[]，后两者返回byte[]后要再解码；~~另有try...方法处理可能出现不符合base64规则字符的情形~~并没看见这样的方法
* IsDBNull：检测对应的值是否为DBNull

### BitConverter

* BitConverter.GetBytes：处理int和char等，返回byte[]；只能处理单个元素，不能处理字符串，因为一个int就要4个byte；用循环直接输出和Encoding.Unicode得到的byte[]一样
* BitConverter.ToString：接受一个byte[]，以横杠作为分隔符输出16进制数字，可接`.Replace("-","")`去掉横杠；更高性能的参见： [https://stackoverflow.com/questions/311165](https://stackoverflow.com/questions/311165)
* BitConverter.ToInt32和ToDouble等：byte[]转换为对应类型
* 用循环直接输出byte[]元素会以十进制数字输出
* 16进制明文字符串转普通字符串：没有太好的方法。对于ASCII，用span/substring每两个字符转换成int再转换成char加到stringbuilder里；否则byte.Parse加到List\<byte\>里再解码

### 进制转换

#### 整数

```
// 这些转换其实会进行端序变换，比如二进制的1110，最低位是0，但是是最后写出来
// string -> int
// 二进制转十进制
int intTen = Convert.ToInt32("10", 2);
// 十六进制转十进制
intTen = Convert.ToInt32("0xa", 16);
// 或：
intTen = int.Parse("a", System.Globalization.NumberStyles.HexNumber); // 第一个参数不需要加0x的前缀

// int -> string
// 十进制转二进制
string s = Convert.ToString((int)a, 2).PadLeft(8, '0'); // char可直接强转；不用padding会省略前面的0
// 十进制转八进制
s = Convert.ToString(intTen, 8);
// 十进制数转十六进制
s = intTen.ToString("X"); // 或者使用string.Format和Convert类也可以

// Binary literal
intTen = 0b_1010;
```

#### 浮点数

```
// https://stackoverflow.com/questions/8272733
// SingleToInt32Bits和相反的函数需要.net core 2.0+，而Double的版本早就有了
// 如果没有，可以用BitConverter.GetBytes -> BitConverter.ToInt32 -> Convert.ToString；因为float和int都是4字节，double用相应的long即可

// float -> int -> 二进制string
float f = -7f;
int i = BitConverter.SingleToInt32Bits(f); // .net core 2.0+
string s = Convert.ToString(i, 2);
// 第二种方法：float --BitConverter.GetBytes--> byte[] --

// 二进制string -> int(2) -> float
string s = "11000000111000000000000000000000";
int i = Convert.ToInt32(s, 2);
float f = BitConverter.Int32BitsToSingle(i);

// 手动方法：https://stackoverflow.com/questions/21244252
static string FloatToBinary(float f)
{
    StringBuilder sb = new StringBuilder(32);
    byte[] ba = BitConverter.GetBytes(f);
    foreach (byte b in ba)
        for (int i = 0; i < 8; i++)
            sb.Insert(0,((b>>i) & 1) == 1 ? "1" : "0");
    string s = sb.ToString();
    string r = s.Substring(0, 1) + " " + s.Substring(1, 8) + " " + s.Substring(9); //sign exponent mantissa
    return r;
}
```

隐式引用转换
------------

* 所有引用类型可以被隐式转换为object类型
* 任何类型可以隐式转换到它继承的接口

### 类可以隐式转换到：

* 它继承链中的任何类
* 它实现的任何接口

### 委托可以隐式转换成：

* System.Delegate
* System.MulticastDelegate
* System.ICloneable
* System.Runtime.Serialization.ISerializable

### 数组可以隐式转换成：

* System.Array
* System.ICloneable
* System.IList
* System.ICollection
* System.IEnumerable

### 数组之间的隐式转换：

* 两个数组有一样的维度
* 元素类型都是引用类型
* 元素类型存在隐式转换
* 然而数组的协变被认为是语言缺陷，不要使用

显式引用转换
------------

* 从object到任何引用类型的转换
* 从基类到从它继承的类的转换

如果显式引用转换后的对象存在对显式引用转换前的对象不存在的成员的引用，CLR会在**运行**时捕获到这种不正确的强制转换并抛出InvalidCastException异常，但它不会导致编译错误。

```
class A { }
class B : A { }
static void Main()
{
    // A a1 = new A();
    // B b1 = (B)a1;
    // 异常

    A a2 = new B(); // 隐式转换
    B b2 = (B)a2; // 允许

    A a3 = null;
    B b3 = (B)a3; // 允许
}
```

运算符重载和用户自定义转换
--------------------------

* 只可以为类和结构定义用户自定义转换
* 不能重定义标准隐式转换或显示转换
* 对于相同的源和目标类型，不能定义隐式或显示转换
* 源类型和目标类型不能通过继承相关联
* 源类型和目标类型都不能是接口类型或object类型
* 转换运算符必须是源类型或目标类型的成员
* 如果转换会丢失数据或抛出异常，要写成显式转换，不要写成隐式的
* 无法直接重载逻辑条件运算符&&和||，需要重载常规逻辑运算符&和|还有关键字true和false达到这个效果；后者仅用于判断短路，返回bool；前者就是&&和||运算时的结果，只是不会有短路效应；与位操作无关，返回的是原类型
* 优先创造目标类型的构造函数，除非目标类型的代码不受控制

```
// 类型转换；注意operator是关键字
public static implicit/explicit operator TargetType(SourceType identifier) {
    //Some Code
    return ObjectOfTargetType;
}

// 运算符重载；其中x是要重载的
public static TargetType operator x(SourceType operand){
    return operand x operand;
}
```

Clone方法复制数组
-----------------

* Clone方法为数组进行浅复制
* Clone值类型数组会产生两个独立数组
* Clone引用类型数组会产生指向相同对象的两个数组
* 其实实现就是object.MemberwiseClone

泛型集合转换
------------

* Cast\<T\>：把集合中的所有元素转换成T，如果无法转换，会在遍历到那一项时抛出异常
* OfType\<T\>：把集合中能够转换成T的转换了

以上两个转换可以把弱类型集合转换为强类型集合，**只能进行引用类型的转换或拆箱**。

类型推断
--------

* C#3开始，类型推断分为两个阶段：阶段一处理显示类型实参，阶段二处理隐式类型实参
* 如果阶段二能够通过已知的类型确定剩下的非固定类型，则固定那些类型，并重复阶段二
* 如果非固定类型有多个推断，看它们是否能转换成其中的一个，则把那个作为固定类型。这称作协同确定类型实参
* Lambda表达式的类型依赖于主体和参数输入的内容，所以需要重复阶段二

out变量的作用域
---------------

```
// https://github.com/dotnet/docs/issues/7552

void F1()
{ // This brace opens the scope for result.
  if (int.TryParse("123", out var result)) // if的小括号不产生scope
        Console.WriteLine(result);
    Console.WriteLine(result); // 可用，与is声明不同，out变量的赋值会泄露到外面
} // This brace closes the scope for result.

void F2() {
    if (false) ;
    else if (int.TryParse("123", out var result)) // in "else" scope
        Console.WriteLine(result);
    Console.WriteLine(result); // 'result' does not exist in the current context
}

void F3() {
    if (false) ;
    else { if (int.TryParse("123", out var result))
        Console.WriteLine(result);
    else Console.WriteLine(result); } // 不出错，因为声明处于可省略的大括号的范围中
}
```

as 和 is 运算符
---------------

* A a = b as A：仅执行引用转换（两者都是引用类型）、可空值类型的转换和装箱转换（仅当b为值或值表达式且A为object）；无法执行用户定义的转换
* if(b is A)：可以执行所有as允许的转换加上拆箱转换的检测，即b是object对象时A可以是int；能转换过去时值为true；另外b为null时会评估为false，但此时as能成功，虽然结果也为null
* if(b is A a)：相当于`A a; if(b is A) { a=b as A; ... }`，但其中a只在if子域中有**值**，而与if**同级**的域内a仍**存在但未初始化**，如果if子域不存在（比如`_=b is A a;`）就相当于声明但没赋值，出了if同级域后完全消失；如果A用var，则a的类型与表达式b相同且永远会成功，即使b为null，一般用于临时在括号内创建变量
* 模式变量还可用于catch中的when，则域为catch
* 7.0开始，后面可以跟枚举常量、const变量、字符串常量

### as和cast对比

* 优先使用as，少用cast：
* cast即使成功了，还是要判断是否为null，因为原变量就可能为null
* 对于存在自定义转换的两个类A和B，如果从共同基类的引用（比如object o = new A()）转换到具体的实例（o as B），as和cast都会在运行时失败，因为cast只会考虑编译期的类型（object）和目标之间有没有转换；当然，如果仅从A自己独有的父类转换到B，两者都会报编译期错误
* foreach语句如果类型和序列不一样，用的是cast；如果序列为非泛型的IEnumerable，又用了自定义类型转换，就会发生上一点的错误

switch的模式匹配
----------------

* switch语句的case不再限制为常量，变得可以声明变量，与is类似；可以
* case的顺序对执行变得有影响，常量需要排前面否则先匹配到int之类的就无效了
* 可以为case A a:...、值类型、case null；可以用`var a when ...`（用var不需加when时用default就行），弃元也可匹配到一切；此时模式变量的域仅在case标签中
* case和catch后可加when语句指定条件
* 可以方便地遍历`IEnumerable<object>`

```
RGBColor FromRainbow(Rainbow colorBand) => colorBand switch{ //变量名加switch
    Rainbow.Red   => new RGBColor(0xFF, 0x00, 0x00), // 不用写break
    Rainbow.Green => new RGBColor(0x00, 0xFF, 0x00),
    Rainbow.Blue  => new RGBColor(0x00, 0x00, 0xFF),
    _             => throw new ArgumentException("invalid enum value", nameof(colorBand)), // 相当于原来的default
};
```


