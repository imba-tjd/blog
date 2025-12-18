## ProcessExplorer

颜色：承载服务的进程为粉色，用户启动的为蓝色，新启动的为绿色，退出的为红色。修改高亮时间：Options-DifferenceHighlightDuration

## 进程

* EPROCESS：代表进程的数据结构。PCB：含有调度信息，是EPROCESS的第一个成员。PEB：位于用户空间，包含已加载模块列表、堆信息
* Job：一组进程，可以限制资源的使用

### 进程通信

* 父子：命令行、stdio
* 剪贴板
* 环境变量
* 消息（常用WM_COPYDATA）
* 管道
* socket
* 共享内存
* COM、OLE
* MailSlot邮件槽：支持一对多
* 其他唯一命名内核对象如Event

### 线程局部存储(Tls)的原理

1. 有一个进程级index bit array，一般是64位，会自动扩缩容。
2. 每个线程创建时在各自的TEB中有长为64的PVOID数组用来存数据。
3. 对于一个业务任务，在开多线程前，用一次TlsAlloc()获得索引，这个index在不同线程中是相同的，但存取数据时从各自的TEB中读写。所有线程结束后要释放。
4. 静态Tls：声明一个index变量，在编译时由编译器分配。因此Tls需要多方协同实现。

## Hook技术

1. 在处理某些消息前调用指定函数，一般包括键鼠、应用事件循环的所有GetMessage/PeekMessage调用。
2. 可以全局安装（线程id指定为0），也可以给某个进程中的线程安装。如果要勾上非本进程，称为远程线程钩子，钩子函数必须在dll中，由OS加载进被勾上的进程中（但并非首次加载）。
3. 钩子链：应在钩子函数里最后return CallNextHookEx，但只需传递参数，不用像剪贴板那样维护下一个节点。

### 实现

在dll项目中：

1. DllMain用于处理dll的加载和卸载，保存hModule即dll自身的模块句柄。
2. 创建一个Shared段的hwnd全局变量，在所有dll实例中共享。
3. 创建一个InstallHook函数，由使用者调用，保存使用者的hwnd，调用API安装钩子 SetWindowsHookEx(想监听的消息类型, 钩子函数回调, 钩子函数所在模块的句柄, 要监视的线程id)。
4. 钩子函数回调中，处理逻辑后，通知使用者PostMessage(hwnd, WM_COPYDATA)，键盘一般用SendMessage。

在使用者代码中，也要引入dll。InstallHook，之后处理COPYDATA

## Shadow API

假设某程序要调用MessageBoxA，反汇编动态调试，给系统函数打断点，就能找到其实现的二进制代码。把代码复制到程序的另一处，把原来call系统函数地址改为新地址。

## 剪贴板监控变化

1. 加入、传递、退出监视链，需要链上的每一个进程正确处理。
2. 加入时，API会返回下一级节点（之前加入的），需要在当前进程的内存里记录下来。
3. 操作系统只会记录、发消息给链头。
4. 进程收到消息后，要传递给下一级。
5. 某个进程退出链时，自身调用一下API即可。但链上的每个进程都会收到通知，检查退出的是不是自己的下一级，如果是，要把退出的那个进程的下一级保存起来，作为新的下级。

## 异步

IOCP 与事件驱动模型（如 select/epoll）的区别：IOCP 是“完成通知”模型，后者是“就绪通知”模型。

主动发起一个读请求，不管有没有数据。投递给工作线程。去干别的事。等查询时缓冲区已经填充好了。

使用过程：

1. 创建一个IOCP主handle（CreateIoCompletionPort第一个参数用INVALID_HANDLE_VALUE）、n个工作线程（一般为核数*2），或用TrySubmitThreadpoolCallback系统线程池。
2. 主线程创建Socket并指定要用Overlapped，listen，调用8次AcceptEx，结束，让工作线程处理AcceptEx的结果。这个8一般是固定值，不与工作线程数有关。AcceptEx不返回Socket，而是填充参数，与客户端通信的Socket要自己创建出来放到overlapped context里。老方法：主线程不断循环阻塞accept()得到Socket，绑定到IOCP
3. 工作线程中不断GetQueuedCompletionStatus，此方法是阻塞休眠的。根据key（作为状态）决定之前完成的是accept还是别的操作。如果是accept，调用CreateIoCompletionPort与已有的iocp主handle关联，处理一些逻辑后释放context，之后①要投递一个接收数据请求否则不能直接读。②再投递一个AcceptEx。如果GetQueuedCompletionStatus一开始返回了False，要处理释放

Overlapped：

本身只提供“异步发起”的能力，但“如何得知 I/O 完成”需要额外机制。不用IOCP时用WaitForSingleObject(ov.hEvent)。

通常使用“结构打包”方式，嵌入自定义结构中，每次异步调用都要new，之后就不管了。等到iocp报告完成时会自动取出来，处理完后要释放。

## COM

* 列出：`gci HKLM:\Software\Classes -ea 0 | ? {$_.PSChildName -match '^\w+\.\w+$' -and (gp "$($_.PSPath)\CLSID" -ea 0)} | ft PSChildName`
  * 列出接口：需要oleview.exe和iviewers.dll。也可以考虑试试Get-Member
  * https://github.com/tyranid/oleviewdotnet Registry - CLSID By Name - 右键Create Instance，下面查看想要的接口中的函数
* 创建：new-object -com "xxx.xxx"、dynamic o = Activator.CreateInstance(Type.GetTypeFromProgID("xxx"))、win32com.client.Dispatch
* OLE：基于COM，专门用于“复合文档”，如在Word中嵌入Excel
* WSH：用于一些自动化，是一些常见操作的封装 https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/windows-scripting/f51wc7hz(v=vs.84)

## WMI、CIM

* Get-WmiObject，Get-CimInstance，System.Management
* GUI查看：wbemtest.exe、https://github.com/vinaypamnani/wmie2、https://www.ks-soft.net/hostmon.eng/wmi/index.htm

## VBS

解决“获得了Ring0就攻破了系统”的问题，允许一个小的只实现了少量功能的内核，放关键服务。

```
           Hypervisor (VTL2)
-------------------------------------------
 VTL1 (Secure World)
   - Secure Kernel (Ring0)
   - Credential Guard secure storage
   - HVCI runtime
   - Key & policy components
   - Isolated User Mode (IUM) 进程
-------------------------------------------
 VTL0 (Normal Windows)
   - Windows Kernel (Ring0)
   - Drivers
   - User-mode processes (Ring3)
       - explorer.exe
       - chrome.exe
       - lsass.exe (front-end)
       - svchost.exe
       - 所有其他应用
```

## UAC

在不提权时，用户具有Administrator组的身份，但其“允许”权限被剥夺，“阻止”权限保留。

## WinRT

* 调用部分需要使用窗口(CoreWindow)的组件（组件实现了IInitializeWithWindow）：WinRT.Interop.InitializeWithWindow.Initialize(组件, 程序hWnd) 支持WinUI3 WPF Winform。有的不用这样但不支持控制台程序，如Clipboard报0x8001010E(The application called an interface that was marshalled for a different thread)
* API列表：https://learn.microsoft.com/zh-cn/uwp/api/ https://microsoft.github.io/windows-docs-rs/doc/windows/index.html 其中后者Win32命名空间下对应的是传统API
* Windows.winmd下载：https://github.com/microsoft/windows-rs/tree/master/crates/libs/bindgen/default 直接下Github Blob，不用看Readme
* Windows.UI是传统UWP，必须打包，内部实现在系统里（system32下有dll）。Microsoft.UI是WinUI3，独立更新，nuget包WinAppSDK，但好像系统里也自带了Runtime

### C++

* 用cppwinrt从winmd生成头文件。
* 先调用winrt::init_apartment()
* 常用工具：check_hresult()、`create_instance<IT>(guid_of<T>(), CLSCTX_ALL)`替代CoCreateInstance、__uuidof
* mingw链接：-lwindowsapp，需-std=c++20。无需-lole32 -lruntimeobject -loleaut32

## API

* 某些帮助库：https://github.com/microsoft/wil
* WSAGetLastError：现在没用了，就用普通的GetLastError即可

## PE文件

```
DOS_MZ头固定64字节，0x3C处指示了PE头的偏移。
DOS Stub块现在没用
PE头：一个Signature PE\0\0，一个标准PE头20字节，一个扩展PE头一般是224字节。
标准PE头：运行平台（i386、amd64）、Section的个数、扩展PE头的长度（32位一般是0xE0，64位是0xF0因为有些指针变大了）、属性（是EXE还是DLL等）
扩展PE头：记录了入口点RVA、Section对齐（包括内存和文件）、DataDirectory等。
节表：每个长度固定，数量为标准PE头里的NumberOfSections。VirtualAddress是RVA，PointerToRawData是节在文件中的位置（FOA）。SizeOfRawData在文件中的长度（已对齐），可以为0，即在文件中无数据，但装载后会按VirtualSize分配内存。之后装载器再按扩展PE头中的DataDirectory使用节表，具体来说，DataDirectory是个下标有固定含义长度固定16的数组，如0代表导出表，1代表导入表，2代表资源表；DataDirectory的成员也有VirtualAddress和Size两个变量，它们是真正数据的位置，要落在某个节的空间之内，节表是一种保护手段


RVA：加载到内存后，某个东西相比于基址ImageBase的偏移；VA内存地址=ImageBase+RVA。FOA：数据在PE文件中的偏移量，文件读写、十六进制编辑器看到的就是
程序映射到内存后，如果无地址随机化，则ImageBase加载到0xB40000。如果被重定位了，用GetModuleHandle(NULL)就能得到ActualImageBase
PE头在文件中（即FOA）要对齐（即不满时填充0）到0x400，放入内存后对齐到（大小为）0x1000。
对齐x：起始地址和长度均为x的倍数
已知某数据在内存中的地址，怎么找到文件中的地址（RVA->FOA）：先从VA计算RVA。遍历节表，看已有的地址落在哪一节里。找到节后，减去节的RVA，得到相对于节的偏移，再加上节的FOA
重定向表：指示哪些地方用到了ImageBase（相当于绝对地址），则装载器会去修改它的值。如果用的是 RIP-相对地址 则不需要动
```
