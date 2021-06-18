---
title: System.IO、Buffer和Span
---

> 参考文章
>
> https://docs.microsoft.com/zh-cn/dotnet/standard/io/
> https://docs.microsoft.com/zh-cn/dotnet/api/system.io?view=netcore-2.1
> 未读：https://zhuanlan.zhihu.com/p/39223648 Pipelines
> ArrayPool\<T\>，需安装System.Buffers
> [C# 7 Series, Part 10: Span\<T\> and universal memory management](https://blogs.msdn.microsoft.com/mazhou/2018/03/25/c-7-series-part-10-spant-and-universal-memory-management/)

System.IO
---------

### 读写器

* TextReader(abstract) -> StreamReader(Stream/string path)、StringReader(string src)
* Synchronized静态方法返回线程安全的实例
* BaseStream返回底层的流
* TextWriter的AutoFlush指定是否每次Write后都flush，默认false
* TextReader的构造函数可指定是否读取BOM头来决定编码
* 默认自动检测BOM，TextReader在第一次读取后才可用CurrentEncoding指示当前编码，或者用构造函数接受一个Encoding类的实例的重载指定编码方式；如果强制不自动检测，则为UTF8-BOM，读和写都是
* StreamWriter默认是覆盖写，如果文件已有，不会创建新文件
* BinaryReader/Writer构造函数接受Stream，之后Write接受许多重载，而读取使用许多以Read为前缀的方法
* 理论设计上读写器能处理任意Stream，但不拥有Stream；实际中StreamWriter是Stream的Owner

### 流

* BinaryReader(File.Open(fileName, FileMode.Open))
* Stream -> (BufferedStream、FileStream、MemoryStream、NetworkStream、PipeStream)：Read、Write、Seek、CanRead、CanWrite、CanSeek、Position、**Length**、SetLength；FlushAsync、Read/WriteAsync、Read/WriteByte
* Null：空流
* Synchronized：静态方法，接受Stream，返回线程安全的流
* 定位到流末尾：Seek(0, SeekOrigin.End)
* 读写都需要手动指定byte数组、开始的位置、数据长度，比较底层
* 写入的时候长度可用byte数组的长度，读取时byte数组的长度可用流的长度
* 是二进制流，在这个层面无法确定编码，要读写器包装后指定；或者自己写入GetPreamble的内容和读取后用指定的编码解码
* MemoryStream不需要Dispose，但其它流基本上都要，为了通用，Stream就继承了IDispose

#### FileStream

* 构造函数的isAsync参数或
    useAsync或FileOptions.Asynchronous可以使IsAsync属性为true；如果为false，调用以Async结尾的方法不会阻塞UI，但实际IO流是同步的；而调用Begin开头的函数会变成真异步IO，但读取小文件反而会变慢
* 构造函数的FileShare.None可以以独占的方式打开，其他程序读都不让
* Name：打开的文件的绝对路径
* Lock、Unlock：锁定/解锁文件的一部分

#### MemoryStream

* 接受byte数组，之后可用流的读写器
* 写入完成之后读取前需要Seek一下
* 有ToArray方法；而ToString是未重载的，返回"System.IO.MemoryStream"

### FileSystemInfo

* 创建时间、访问时间、修改时间
* Attributes：是FileAttributes的枚举，系统、隐藏什么的

#### File/FileInfo

* File在每次使用时接受路径参数，FileInfo在实例化时接受路径参数，但该路径都可以不存在，Directory同理
* FileStream Create/Open/OpenRead/OpenWrite；StreamReader/Writer  CreateText/AppendText/OpenText；注意必须要用using，否则文件不能移动和删除
* 如果返回IEnumerable（比如用了Linq或者yield），序列会直到使用时才生成；如果是接受stream的方法，可能使用时文件已经被关闭，或者如果同时调用两个序列，文件会打开失败，解决方法：要么先全部读到数组里再yield返回，要么只接受文件名和Action，打开和关闭文件在函数自己内部操作（这个也只能开一个）
* Exists：File是方法，FileInfo是属性
* Move/MoveTo/Copy/CopyTo：参数路径只接受string，可以为相对路径；目标路径必须写到文件名，不能写到文件夹名，否则会说目标文件已存在，其实是不能把文件的名字改成那个文件夹的名字，在最后加上斜杠也不行
* Create、Delete：略
* Encrypt、Decrypt：用的是EFS；但只有Windows下才能用，而且其实EFS应该是透明的，最好不要用这个
* Replace：使用当前文件替换指定路径的文件并创建备份
* FileInfo独有：Name：带后缀的纯文件名、Directory：返回DirectoryInfo、DirectoryName：最后无斜杠的完全路径、Length：文件大小，单位为字节、IsReadOnly：**设置**/获取文件是否只读
* File独有：Append/Read/WriteAllLines(string[])/Text(string)/Bytes[Async]：都是先打开，读取/写入后再关闭；还有一些读取/设定修改时间的函数，在FileInfo中是属性
* FileInfo.ToString：如果主动使用构造函数，即使传递给它相对路径，也会返回绝对路径；如果使用DirectoryInfo.GetFiles/EnumerateFiles，则只会返回文件名，即使枚举子文件夹

#### Directory/DirectoryInfo

* DirectoryInfo独有：Name：文件夹的基本名、FullName：绝对路径，末尾没有斜杠、Root：根目录、Parent：父目录Info、CreateDirectory：以该info为相对路径创建文件夹
* GetFiles/EnumerateFiles：前者返回string[]和FileInfo[]，必须等数组完全建立好才行；后者返回IEnumerable string/FileInfo，等到枚举开始时才获取内容，不会缓存结果，每次获取枚举内容都会重新检索一遍；searchPattern中可以使用通配符，但如果基本名使用*并且后缀名只有三个字母，则会把多于三个字母后缀的文件也匹配了（DOS遗留问题，CMD也是如此但PS不是这样）。**小心隐藏文件**。
* Directory.GetFiles/EnumerateFiles：如果path是相对路径，返回的也是；DirectoryInfo的见FileInfo.ToString
* CreateDirectory(string path).ToString()的返回值与path相同，如果只写文件夹名不写完整路径，返回的也只有文件夹名，当然创建文件夹会成功
* ToString：如果用的是构造函数，返回原始的路径；如果是GetDirectory创建的，返回基本名
* Directory.Get/SetCurrentDirectory与Environment.CurrentDirectory效果一样；AppContext.BaseDirectory是程序集所在目录

### Path

* 大部分方法不检测路径是否有效
* 基本上都是把参数当成文件对待，比如`Path.GetFileName(Environment.CurrentDirectory)`才能返回工作目录的文件夹名词，因为参数末尾不包含反斜杠；如果用GetDirectoryName，会返回父目录
* 如果路径中存在GetInvalidPathChars()中的字符，会抛出异常
* 点指的是当前路径/工作路径，而不是Info类实例的相对路径

#### 分隔符

* DirectorySeparatorChar：Windows下是反斜杠，另两个是斜杠；AltDirectorySeparatorChar：三个系统都是斜杠
* PathSeparator：Windows下是分号
* VolumeSeparatorChar：Windows下是冒号

#### Get开头的方法

* GetDirectoryName：路径的文件夹前面的部分都会保留，末尾没有分隔符；如果实参末尾就是分隔符，仅仅会去掉它；如果实参是根目录，返回null
* GetExtension：返回的后缀包括点；如果不具有后缀或是目录分隔符，返回空字符串
* GetFileName：包含扩展名；如果是目录分隔符，返回空字符串
* GetFileNameWithoutExtension
* GetFullPath：基于当前路径，把相对路径变成绝对路径；.net core2.1以后允许指定基路径，之前可先用Combine
* GetPathRoot：如果参数没有根目录，会返回空字符串而不会报错；否则返回C:\这样的

#### 其他方法

* ChangeExtension：如果本来没有点，会加上点；如果后缀传null，会去掉最后一个后缀和点；如果后缀传""，只会去掉后缀不会去掉点；如果路径传""，不会有改变
* Combine：第一个参数指定基位置，再用GetFullPath即可变成指定基路径的绝对路径；如果中间出现绝对路径，会重新开始；如果路径中有空格，不会去掉；不能用于添加后缀，会把前一个参数当成目录的
* HasExtension
* IsPathRooted：路径是否以根目录开头，包括Windows形式的和Linux形式的
* IsPathFullyQualified：是否是绝对路径
* Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments)可以获取特殊文件夹的路径

### 管道

* 位于System.IO.Pipes命名空间中，用于进程间传递消息
* 分为匿名管道和命名管道
* 示例参见[文档教程](https://docs.microsoft.com/zh-cn/dotnet/standard/io/pipe-operations)

### 内存映射文件

* 可以处理极大的文件，可以让多个程序同时使用
* 位于System.IO.MemoryMappedFiles命名空间
* 示例参见[文档教程](https://docs.microsoft.com/zh-cn/dotnet/standard/io/memory-mapped-files)

权限
----

* https://docs.microsoft.com/zh-cn/dotnet/api/system.security.permissions.fileiopermission
* 如果文件不可访问，在枚举的时候可能会发生异常；如果仍要进行处理，可以考虑用try包裹GetFiles，只获取当前目录下的文件。再对所有文件夹递归使用本方法

Span、Memory
------------

> https://blogs.msdn.microsoft.com/mazhou/2018/03/25/c-7-series-part-10-spant-and-universal-memory-management/

* System.MemoryExtensions类包含许多扩展方法：AsSpan对stirng转换成ReadOnlySpan，对所有数组转换成普通Span、Trim、IsWhiteSpace（不需要复制整个string了）、ToUpper；因为是System的静态类，所有方法直接都有了
* Span是ref struct，不能装箱或分配给object和dynamic、不能是类的字段、不能跨await和yield边界；但是Span实例可以指向托管类型
* Slice方法：类似于SubString方法，注意两者第二个参数都是length，不是to
* ToArray：仍是堆分配，用于某些只支持数组不支持Span的方法
* ToString：对于Char的Span，返回字符串，否则返回Span的长度
* TryCopyTo：目标长度不够时返回false；不用Try则抛异常
* Fill、Clear：略
* System.Runtime.InteropServices.MemoryMarshal类的AsMemory静态方法允许把ReadOnlyMemory转换成Memory
* 如果要改变Span里的数据，可用局部的ref变量：ref char first = ref span[0];。这样不会分配真正的变量，而是直接用类似于指针的方式，所以也许可以提升一点点性能；但ReadOnlySpan无法这样用，因为不允许改变内容
* Memory用于异步方法和流，具有Span属性

```c#
// 隐式转换或用AsSpan（只读）或传递给构造函数
Span<char> span1 = new char[] { 's', 'p', 'a', 'n' };

// Use stackalloc
Span<byte> span2 = stackalloc byte[50];

// Use constructor
IntPtr array = new IntPtr();
Span<int> span3 = new Span<int>(array.ToPointer(), 1)
```

System.Buffer
-------------

https://stackoverflow.com/questions/415291/best-way-to-combine-two-or-more-byte-arrays-in-c-sharp；必须是基元数组，不能是string[]

UnauthorizedAccessException
---------------------------

* The caller does not have the required permission.
* The file is an executable file that is in use.
* Path is a directory.
* Path specified a read-only file.

https://sakno.github.io/dotNext/features/io/index.html
