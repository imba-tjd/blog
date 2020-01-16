--- layout: post title: 反射、特性和动态类型 date: 2017-08-13 09:11:18.000000000 -05:00 type: post parent\_id: '0' published: true password: '' status: publish categories: - C\# tags: [] meta: \_wpcom\_is\_markdown: '1' \_rest\_api\_published: '1' \_rest\_api\_client\_id: "-1" \_publicize\_job\_id: '8208346176' \_wp\_old\_slug: "%e5%8f%8d%e5%b0%84%e5%92%8c%e7%89%b9%e6%80%a7" author: login: imbalancedweb email: imba.tjd@gmail.com display\_name: imba-tjd first\_name: '' last\_name: '' permalink: "/2017/08/13/%e5%8f%8d%e5%b0%84%e3%80%81%e7%89%b9%e6%80%a7%e5%92%8c%e5%8a%a8%e6%80%81%e7%b1%bb%e5%9e%8b/" ---

> 《Illustrated C\# 2012 (4th Edition)》
> 《C\# 6.0 学习笔记：从第一行代码到第一个项目设计》

反射
====

-   有关程序及其类型的数据被称为元数据，它们保存在程序的程序集中
-   程序在运行时，可以查看其他程序集或其本身的元数据，这种行为叫做反射
-   使用GetType方法和typeof运算符来获取Type对象

System.Type类部分成员
---------------------

-   Name
-   Namespace
-   Assembly：如果类是泛型的，返回定义这个类型的程序集
-   GetFields
-   GetProperties
-   GetMethods
-   GetCustomAttributes
-   IsDefined：是否应用了某个特性

返回该类所有属性及其值
----------------------

    string ObjToStr()
    {
        string ret_str = this.GetType().Name + "的属性信息：\n";
        PropertyInfo[] props = this.GetType().GetProperties(BindingFlags.Public | BindingFlags.Instance);
        foreach (var p in props)
            ret_str += $"{p.Name} : {p.GetValue(this)?.ToString() ?? "null"}\n";
     
        return ret_str;
    }

特性
====

-   将应用了特性的程序结构叫做目标
-   设计用来获取和使用元数据的程序叫做特性的消费者（编译器、CLR、浏览器）
-   编译器获取源代码并且从特性产生元数据，然后把元数据放到程序集中
-   特性名使用Pascal命名法并以Attribute后缀结尾。当目标应用特性时，可以不使用后缀
-   多个特性可以用多层结构（一行一个）或单层结构（中括号内逗号分隔）
-   使用命名参数按需初始化部分属性或字段

### 将特性应用于目标对象的完整格式

默认条件下，特性将应用于跟随其后的对象。但是在用于方法的返回值时必须使用完整格式：

    [<目标> : <特性列表>]

特性目标关键字：

-   field
-   event
-   method 用于方法或get/set访问器
-   param 用于方法中的参数或属性定义中的set访问器中的value
-   property
-   type 用于类型
-   return 用于方法的返回值或get访问器的返回值，不能省略
-   assembly 用于当前程序集
-   module 用于当前模块

预定义保留的特性
----------------

### Obsolete特性

将程序结构标注为过期的，并且在代码编译时显示有用的警告信息

    [Obsolete("Message")]
    [Obsolete("Message"), true] // bool类型的第二个参数指定目标被标记为错误

### Conditional特性

允许我们包括或排斥特定方法的所有调用。为方法声明应用Conditional特性并把编译符作为参数使用。

如果定义（\#define）了编译符号，则和普通方法没有区别。如果没有定义，那么编译器会忽略这个方法的所有调用，代码仍包含在程序集中。

    [Conditional("DEBUG")]
    public static void Print(string message);

### 调用者信息特性

-   可以使程序访问文件路径、代码行数、调用成员的名称等源代码信息
-   这些特性只能用于方法中的可选参数，无法用于动态类型。

<!-- -->

    using System;
    using System.Runtime.CompilerServices;
     
    namespace CSharpConsoleTest
    {
        class Program
        {
            static void MyTrace(string message,
                [CallerFilePath] string fileName = "",
                [CallerLineNumber] int lineNumber = 0,
                [CallerMemberName] string callingMember = "")
            {
                Console.WriteLine($"Message:       {message}");
                Console.WriteLine($"FileName:      {fileName}");
                Console.WriteLine($"Line:          {lineNumber}");
                Console.WriteLine($"Called From:   {callingMember}");
            }
     
            static void Main()
            {
                MyTrace("test");
                Console.ReadKey();
            }
        }
    }

### DebuggerStepThrough特性

调试器在执行目标代码时不会进入内部调试。位于System.Diagnostics命名空间。可用于类、结构、构造函数、方法或访问器。

### AttributeUsage特性

可以为类应用特性，而特性本身就是类。AttributeUsage特性可以应用到自定义特性来限制自定义特性能应用到什么类型的程序结构。

#### AttributeUsage的公共属性

-   ValidOn：目标类型的列表，构造函数的第一个参数必须是AttributeTarget类型的枚举值
-   Inherited：指示特性是否会被装饰类型的派生类所继承。默认值为true
-   AllowMultiple：指示目标是否被应用多个特性实例。默认值为false

#### AttributeUsage的构造函数

接受单个位置参数，使用按位或运算符来组合使用类型。例如，在下面的代码中，被装饰的特性只能应用到方法和构造函数上。

    [AttributeUsage(AttributeTarget.Method|AttributeTarget.Constructor)]
    public sealed class MyAttributeAttribute:System.Attribute{...}

#### AttributeTarget枚举的成员

-   All
-   Assembly
-   Class
-   Constructor
-   Delegate
-   Enum
-   Event
-   Field
-   GenericParameter
-   Interface
-   Method
-   Module
-   Parameter
-   Property
-   ReturnValue
-   Struct

### 友元程序集

    [assembly:InternalVisibleTo("...")]
    [assembly:InternalVisibleTo("..."), PublicKey="..."]

其中PublicKey可由`sn -Tp xxx.dll`得到，且需要Key而不是Key token。

### 其他预定义特性

|特性|意义|
|:---|:---|
|CLSCompliant|声明可公开的成员应该被编译器检测是否符合CLS。兼容的程序集可以被任何.NET兼容的语言使用|
|Serializable|声明结构可以被序列化|
|NonSerialized|声明结构不能被序列化|
|DLLImport|声明是非托管代码实现的|
|WebMethod|声明方法应该被作为XML Web服务的一部分暴露|

自定义特性
----------

-   继承自System.Attribute的类即可
-   特性类应该表示目标结构的一些状态
-   如果特性需要某些字段，可以通过包含具有位置参数的构造函数来收集数据，可选字段可以采用命名参数按需初始化
-   除了属性之外，不要实现公共方法或其他函数成员
-   为了更安全，把特性类声明为sealed
-   在特性声明中使用AttributeUsage来显式指定特性目标组

动态类型
========

特点
----

-   在CLR级别，dynamic就是应用了DynamicAttribute的object
-   几乎所有的CLR类型都可以隐式转换为dynamic（和object，所有dynamic类型的表达式都可以隐式转换为CLR类型，但在类型推断时不认为dynamic类型能隐式转换为其他类型（而是别的转换为它）
-   如果用var声明的变量是由dynamic类型的表达式初始化的，该变量的类型会被静态地推断为dynamic。这会可能引起混淆
-   如果对象是动态的，则就是动态的；如果函数重载有参数是动态的，则先尝试使用实际的类型，不行再用动态的重载，还没有则运行时错误；普通函数即使有动态参数，但如果已知部分类型错误或者数量不对，仍然可以编译期报错
-   动态表达式并不总是动态地求值，如果普通的求值能成功就不会；大多数动态运算的结果还是dynamic，但并不总是如此，比如new SomeType(d)

### 限制

-   不能在动态类型上直接调用扩展方法，但可以主动使用扩展方法
-   不能直接把委托和lambda表达式赋给dynamic，但可强制转换后赋
-   构造函数无法返回动态类型的对象，但是可以用工厂方法
-   不能声明基类为dynamic的类，不能用于泛型类型参数的约束(where T: dynamic)，不能用于实现泛型接口的类型参数(:IEnumerable\<dynamic\>)；可以用于实现泛型类的类型参数(:List\<dynamic\>)或泛型接口变量

使用实例
--------

### 静态方法动态调用

    private static bool AddConditionallyImpl<T>(IList<T> list, T item)
    {
        if (list.Count > 10)
        {
            list.Add(item);
            return true;
        }
        return false;
    }

    public static bool AddConditionally(dynamic list, dynamic item)
    {
        return AddConditionallyImpl(list, item);
    }

### 弥补泛型操作符不足

-   不要声明成IEnumerable\<dynamic\>，因为协变性无法用于值类型

<!-- -->

    public static T DynamicSum<T>(this IEnumerable<T> source)
    {
        dynamic total = default(T);
        foreach (T element in source)
        {
            total = (T) (total + element); // 如果不强制转换，运算时会进行提升；如果T为byte，返回时就会异常
        }
        return total;
    }

动态语言
--------

### 使用脚本

                ScriptEngine engine = Python.CreateEngine();
                engine.Execute("print 'hello, world'");
                engine.ExecuteFile("HelloWorld.py");

### 使用变量

                string python = @"
    text = 'hello'
    output = input + 1
    ";
                ScriptEngine engine = Python.CreateEngine();
                ScriptScope scope = engine.CreateScope();
                scope.SetVariable("input", 10);
                engine.Execute(python, scope);
                Console.WriteLine(scope.GetVariable("text"));
                Console.WriteLine(scope.GetVariable("input"));
                Console.WriteLine(scope.GetVariable("output"));

### 使用函数

                string python = @"
    def sayHello(user):
        print 'Hello %(name)s' % {'name' : user}
    ";
                ScriptEngine engine = Python.CreateEngine();
                ScriptScope scope = engine.CreateScope();
                engine.Execute(python, scope);
                dynamic function = scope.GetVariable("sayHello");
                function("Jon");

### 最佳可访问类型

    static void Execute(dynamic x, string y) => Console.WriteLine("dynamic string");
    static void Execute(dynamic x, object y) => Console.WriteLine("dynamic object");
    static void Main()
    {
        object text = "text";
        dynamic d = 10;
        Execute(d, text); // dynamic object
        Execute(d, (dynamic)text); // dynamic string
    }

DLR
---

-   调用点、接收器、绑定器
-   多级缓存

实现动态行为
------------

-   位于System.Dynamic下
-   可用于DOM树

### ExpandoObject

-   无法被继承，显式实现了IDictionary
-   可以直接对不存在的属性赋值；如果属性名不是字面量，用dictionary的索引器，即作为属性名
-   可以把强制转换后的委托赋给属性，即成为函数；如果赋给ToString，效果是new而不是override
-   无法自定义它本身的索引器
-   赋值的时候是复制的值，它在建立的时候就需要生成整个树

<!-- -->

    dynamic expando = new ExpandoObject();
    IDictionary<string, object> dictionary = expando;
    expando.First = "value set dynamically";
    Console.WriteLine(dictionary["First"]);

    dictionary["Second"] = "value set with dictionary";
    Console.WriteLine(expando.Second);

### DynamicObject

-   需要继承后使用
-   构造函数设为私有，提供返回dynamic类型的工厂方法
-   可以添加自己的索引器，这样直接使用自己就是自己，索引以后就是自己的兄弟；如果没有的话就要再来一个列表属性
-   只有在使用的时候才会获取对应的值；即使获取相同的元素，也返回的是不同的引用
-   支持简单成员；如果要实现动态成员和各种运算，需要覆盖Try...方法；binder.Name为调用的名字，不存在时返回基类的该方法；获取它的所有成员，覆盖GetDynamicMemberNames方法

#### Try...方法

-   TryBinaryOperation：二元运算，如x + y
-   TryConvert：转换，如(Target)x
-   TryCreateInstance：创建对象表达式，C\#中没有等价的操作
-   TryDeleteIndex：移除索引的操作，C\#中没有等价的操作
-   TryDeleteMember：移除属性的操作，C\#中没有等价的操作
-   TryGetIndex：获取索引器中的项，如x[10]
-   TryGetMember：获取属性的值，如x.Property
-   TryInvoke：视为函数（委托）调用，如x(10)
-   TryInvokeMember：成员调用，如x.Method(10)
-   TrySetIndex：设置索引器中的项，如x[10] = 20
-   TrySetMember：设置属性，如x.Property = 10
-   TryUnaryOperation：一元运算，如!x或-x

### IDynamicMetaObjectProvider

-   太复杂，看不懂

Marshal
-------

-   AllocHGlobal、FreeHGlobal、ReAllocHGlobal
-   StringToHGlobalAnsi、PtrToStringAnsi、Auto、UTF8、Uni，HGlobalToString...
-   PtrToStructure、StructureToPtr：后者把结构体封送到非托管内存中，空间必须先分配
-   ReadByte、Read16...、ReadIntPtr，Write...
-   Copy
-   SizeOf
-   GetDelegateForFunctionPointer、GetFunctionPointerForDelegate：仅用于C
-   C调用C\#的函数：https://www.zhihu.com/question/58336072

### IntPtr

-   32位系统4字节长，64位8字节，因此可用作储存指针的数据
-   等价于Handle或者void\*，是一些Marshal类方法的返回类型
-   ToPointer方法就可以转换成void\*

P/Invoke
--------

-   win32的API可直接用http://pinvoke.net/
-   c++会对函数名进行改写，需要用`extern "C"`；但`__stdcall`好像也会修改函数名；官方推荐c++写COM
-   WinAPI的long是32位的，而C\#的long是64位的，直接用int就好；char对应sbyte，wchar\_t对应char，const char\*对应string，char\*对应StringBuilder；MarshalAs数组时可以指定SizeConst；其它的参考：http://www.cnblogs.com/wangjt18/archive/2011/10/08/2202365.html
-   具体情况具体分析，还可以使用共享内存，消息，IPC，管道，Socket，文件，数据库，队列，COM等等
-   例子：https://zhuanlan.zhihu.com/p/29161824

``` {.wp-block-code}
int __declspec(dllexport) __stdcall multiply(int a, int b) { return a*b; }
----
using System.Runtime.InteropServices;

[DllImport("xxx.dll/so",  EntryPoint="函数名", CharSet = CharSet.Ansi/, CallingConvention=CallingConvention.StdCall/Cdecl, SetLastError=true)]
static extern void print(string message);
```

序列化
------

``` {.wp-block-code}
[DataContract(Name="error"), Serializable]
public class Error
{
    [DataMember(Name="code")]
    public string Code{ get; set; }
    [DataMember]
    public string Message{ get; set; }
    [NonSerialized]
    public string others; // 逆序列化时会为null
}

var serializer = new DataContractJsonSerializer(typeof(Error));
var result = (T)serializer.ReadObject(stream); // 或用as；不能用string
```

``` {.wp-block-code}
using System.Runtime.Serialization
using System.Runtime.Serialization.Formatters.Binary;
var stream = new MemoryStream();
var bf = new BinaryFormatter();
bf.Serialize(stream, new Vector2{x = 1d, y = 1d});
var o = bf.Deserialize(stream) as xxx

using System.Xml.Serialization
var writer = new XmlSerializer(typeof(x));
writer.Serialize("file.xml", new x);
```

