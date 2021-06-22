---
title: 反射、特性和动态类型
category: dotnet
---

## Marshal

* AllocHGlobal、FreeHGlobal、ReAllocHGlobal
* StringToHGlobalAnsi、PtrToStringAnsi、Auto、UTF8、Uni，HGlobalToString...
* PtrToStructure、StructureToPtr：后者把结构体封送到非托管内存中，空间必须先分配
* ReadByte、Read16...、ReadIntPtr，Write...
* Copy
* SizeOf
* GetDelegateForFunctionPointer、GetFunctionPointerForDelegate：仅用于C
* C调用C#的函数：https://www.zhihu.com/question/58336072

### IntPtr

* 32位系统4字节长，64位8字节，因此可用作储存指针的数据
* 等价于Handle或者void*，是一些Marshal类方法的返回类型
* ToPointer方法就可以转换成void*
* System.Runtime.InteropServices.SafeHandle包装了它，使用它可避免实现析构函数；不过它本身是抽象类，Microsoft.Win32.SafeHandles提供了一些实现

## P/Invoke

* win32的API可直接用http://pinvoke.net/
* c++会对函数名进行改写，需要用`extern "C"`；但`__stdcall`好像也会修改函数名；官方推荐c++写COM
* WinAPI的long是32位的，而C#的long是64位的，直接用int就好；char对应sbyte，wchar_t对应char，const char*对应string，char*对应StringBuilder；MarshalAs数组时可以指定SizeConst；其它的参考：http://www.cnblogs.com/wangjt18/archive/2011/10/08/2202365.html
* 具体情况具体分析，还可以使用共享内存，消息，IPC，管道，Socket，文件，数据库，队列，COM等等
* 例子：https://zhuanlan.zhihu.com/p/29161824
* https://github.com/microsoft/CsWin32

```c#
int __declspec(dllexport) __stdcall multiply(int a, int b) { return a*b; }
----
using System.Runtime.InteropServices;

[DllImport("xxx.dll/so",  EntryPoint="函数名", CharSet = CharSet.Ansi/, CallingConvention=CallingConvention.StdCall/Cdecl, SetLastError=true)]
static extern void print(string message);
```

## 序列化

```c#
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

不学二进制序列，Core1的时候曾删掉了，也不安全。不学XML序列化，现在没人用XML。

TODO：https://devblogs.microsoft.com/dotnet/evolving-the-reflection-api/
