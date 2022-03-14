---
title: 类的转换和模式匹配
---

值类型转换
----------

* double转换到float
  * 如果值太小，结果为正0或负0
  * 如果值太大，结果为正无穷大或负无穷大
* double转换到int
  * 强制转换会去掉小数，向0截断为最近的整数
  * 强制转换如果转换后的值不在目标类型的范围内（溢出）：在checked()或{}中会抛OverflowException，如果溢出上下文是unchecked则未定义
  * 如果用Convert类，会发生四舍五入，溢出时必定抛异常
* TODO: int溢出是否是定义了的行为。checked()不会检查左移右移溢出

### 类

* ChangeType方法：将实现了IConvertible接口的类型转换为指定的类型
* 以To开头的方法：将参数中传入的对象转换为指定的类型对象
* ToBase64String/CharArray, FromBase64String/CharArray：前两者先要用Encoding类转换成byte[]，后两者返回byte[]后要再解码；Core提供TryFromBase64String用于可能存在非base64字符的字符串
* .NET5引入了 Convert.ToHexString 和 FromHexString 把byte[]转成与十六进制字符串互转

### BitConverter

* BitConverter.GetBytes：处理int和char等，返回byte[]；只能处理单个元素，不能处理字符串，因为一个int就要4个byte；用循环直接输出和Encoding.Unicode得到的byte[]一样
* BitConverter.ToString：接受一个byte[]，结果类似于`"E1-0A-DC-39"`
* BitConverter.ToInt32和ToDouble等：byte[]转换为对应类型
* 用循环直接输出byte[]元素会以十进制数字输出
* 十六进制明文字符串转普通字符串：没有太好的方法。对于ASCII，用span/substring每两个字符转换成int再转换成char加到stringbuilder里；否则byte.Parse加到List\<byte\>里再解码

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
* 要定义在作用对象的类中，否则报“二元运算符的参数之一必须是包含类型”

```
// 类型转换；注意operator是关键字
public static implicit/explicit operator TargetType(SourceType identifier) {
    //Some Code
    return ObjectOfTargetType;
}

// 运算符重载，其中x是待重载的运算符
public static TargetType operator x(SourceType operand) => operand x operand
```

switch的模式匹配
----------------

* switch语句的case不再限制为常量，变得可以声明变量，与is类似；可以
* case的顺序对执行变得有影响，常量需要排前面否则先匹配到int之类的就无效了
* 可以为case A a:...、值类型、case null；可以用`var a when ...`（用var不需加when时用default就行），弃元也可匹配到一切；此时模式变量的域仅在case标签中
* case和catch后可加when语句指定条件
* 可以方便地遍历`IEnumerable<object>`
* https://docs.microsoft.com/zh-cn/dotnet/csharp/fundamentals/functional/pattern-matching https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/patterns https://docs.microsoft.com/zh-cn/dotnet/csharp/fundamentals/tutorials/pattern-matching

```
RGBColor FromRainbow(Rainbow colorBand) => colorBand switch{ //变量名加switch
    Rainbow.Red   => new RGBColor(0xFF, 0x00, 0x00), // 不用写break
    Rainbow.Green => new RGBColor(0x00, 0xFF, 0x00),
    Rainbow.Blue  => new RGBColor(0x00, 0x00, 0xFF),
    _             => throw new ArgumentException("invalid enum value", nameof(colorBand)),
};
```

https://zhuanlan.zhihu.com/p/475688767
