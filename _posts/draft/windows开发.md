## ProcessExplorer

颜色：承载服务的进程为粉色，用户启动的为蓝色，新启动的为绿色，退出的为红色。修改高亮时间：Options-DifferenceHighlightDuration

## 进程

* EPROCESS：代表进程的数据结构。KPROCESS(PCB)：含有调度信息，是EPROCESS的第一个成员。PEB：位于用户空间，包含已加载模块列表、堆信息
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
* 创建：new-object -com "xxx.xxx"、dynamic o = Activator.CreateInstance(Type.GetTypeFromProgID("xxx"))但实际一般用工作集引用、win32com.client.Dispatch
* OLE：基于COM，专门用于“复合文档”，如在Word中嵌入Excel
* WSH：用于一些自动化，是一些常见操作的封装 https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/windows-scripting/f51wc7hz(v=vs.84)

## WMI、CIM

* Get-WmiObject，Get-CimInstance，System.Management
* GUI查看：wbemtest.exe、https://github.com/vinaypamnani/wmie2、https://www.ks-soft.net/hostmon.eng/wmi/index.htm

## 核心架构和组件

```
Hyper-V
-------------------------------------------
内核态：
HAL
内核、设备驱动
执行体
-------------------------------------------
用户态：
Ntdll.dll
子系统DLL、系统进程
用户进程、服务进程
```

* Ntoskrnl：执行体（内核的上层）和内核
* Win32k.sys：子系统的内核态驱动，提供下列支持：窗口管理器（显示窗口、收集键鼠输入）、dwm、GDI、DirectX、conhost
* Hvix64：hypervisor
* Ntdll：执行体函数的stub
* kernel32.dll、advapi32.dll、user32.dll、gdi32.dll：Windows子系统
* smss：会话管理器，启动子系统。其中会话0为系统核心进程，非交互式不可登录；用户登录的桌面在会话1。会创建winlogon和wininit
* csrss：环境子系统进程。提供：进程线程的创建 等
* System进程：其线程均为内核态，来自于Ntoskrnl和驱动
* winlogon：负责登录和注销。处理ctrl+alt+shift，会启动LogonUI。获取用户名密码后发送给Lsass进行验证

### VBS

解决“获得了Ring0就攻破了系统”的问题，允许一个小的只实现了少量功能的内核，放关键服务。

虚拟机之前在Ring0跑自己的hypervisor。启用VBS后，只允许hyper-v作为唯一的根hypervisor，其它虚拟机要用它的API

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

## ACL

* 在不UAC提权时，用户具有Administrator组的身份，但其“允许”权限被剥夺，“阻止”权限保留
* 查看某个用户最终具有的权限：高级 - 有效访问

## UAC

* 虚拟化：对于32位程序，如果写入%programfiles%、windows等，会重定向到LocalAppData\virtualstore里。某些注册表也会虚拟化
* 启发式：如果exe名字里有Setup、Install、Update，且未设置manifest，则会触发

## WinRT

* 调用部分需要使用窗口(CoreWindow)的组件（组件实现了IInitializeWithWindow）：WinRT.Interop.InitializeWithWindow.Initialize(组件, 程序hWnd) 支持WinUI3 WPF Winform。有的不用这样但不支持控制台程序，如Clipboard报0x8001010E(The application called an interface that was marshalled for a different thread)
* API列表：https://learn.microsoft.com/zh-cn/uwp/api/ https://microsoft.github.io/windows-docs-rs/doc/windows/index.html 其中后者Win32命名空间下对应的是传统API
* Windows.winmd下载：https://github.com/microsoft/windows-rs/tree/master/crates/libs/bindgen/default 直接下Github Blob，不用看Readme
* Windows.UI是传统UWP，必须打包，内部实现在系统里（system32下有dll）。Microsoft.UI是WinUI3，独立更新，nuget包WinAppSDK，但好像系统里也自带了Runtime

### C++

* 用cppwinrt从winmd生成头文件
  1. curl -L https://www.nuget.org/api/v2/package/Microsoft.Windows.CppWinRT -o cppwinrt.zip
  2. cppwinrt -input local -output out -optimize
* mingw链接：-lwindowsapp，需-std=c++20。无需-lole32 -lruntimeobject -loleaut32
* 先调用winrt::init_apartment()
* 常用工具：check_hresult()、`create_instance<IT>(guid_of<T>(), CLSCTX_ALL)`替代CoCreateInstance、__uuidof
  * winrt::handle safeHandle { fun_that_returns_HANDLE(); }; if (!safeHandle) {return 1;} fun_accept_handle(safeHandle.get())

## API

* CommandLineToArgvW：把一行命令（一个U16字符串）解析成argv和argc
* GetBinaryType：判断exe是32位还是64位的，但对dll无效
* GetCurrentProcess：返回值是-1，表示当前进程对象的伪句柄。为了与将来的操作系统兼容最好调用本函数而不是硬编码
* Socket：用winsock2.h和ws2_32.dll，别用wsock32.dll
  * 如果要用Win的扩展，如IOCP：mswsock.h
  * 头文件必须在windows.h之前include
  * 从17063(可能是2018)后支持Unix socket（AF_UNIX）。提供双向关闭语义，而命名管道没有。类型仅支持流(SOCK_STREAM)，寻址格式支持pathname；abstract实际上不支持，unnamed因为socketpair不存在而基本不支持
* -DNOMINMAX，要在包含windows.h前定义
* -DWIN32_LEAN_AND_MEAN：当不使用COM时定义，能加速；会排除加密、DDE、RPC、Shell、Socket。还有一些其他选项如NOCOMM，但好像如果优化开高就区别不大了
* `_WIN32_WINNT`和WINVER：MinGW64目前默认为0xa00(Win10)，MinGW.org的为0x500(Win2000)。NTDDI_VERSION：比前者划分得更精细
* NOMINMAX
* WSAGetLastError：现在没用了，就用普通的GetLastError即可
* 现在的Windows不存在global heap，GlobalAlloc和LocalAlloc是废弃的。应该使用HeapAlloc等
* 某些帮助库：https://github.com/microsoft/wil

### 错误码

* 合集：https://learn.microsoft.com/zh-cn/windows/win32/debug/system-error-codes
  * 输入数字自动匹配可能的：https://www.magnumdb.com/ 国内ip访问会404
  * Microsoft错误查找工具：输入后自动多次尝试分辨是不是HRESULT、十六进制转换等
  * FORMAT_MESSAGE_FROM_SYSTEM：https://learn.microsoft.com/zh-cn/windows/win32/api/winbase/nf-winbase-formatmessage
* 0xc0000135：缺失DLL
* 0xc0000139：定位符号失败，如将MSVC的DLL与MinGW的exe链接到一起
* WSAGetLastError() 返回的socket错误码从10000开始：https://learn.microsoft.com/zh-cn/windows/win32/winsock/windows-sockets-error-codes-2

## 时间片、定时器相关概念

* Precision精度（详细程度）不是 Accuracy准度：如现在时间其实是1:00:00，如果回答1:30:00则精度高（使用了更多bit），回答1:05则准度高
* GetTickCount函数有1ms的精度，但准确度取决于计时器tick频率。QueryPerformanceCounter有更高准度，但使用前要先用QueryPerformanceFrequency获得精度，且用起来更慢
* Resolution分辨率：最小增长（或distinguishable可分辨）的单位。Unix是1ms，Win1803后是1ms，之前是16ms
* 对于时间来说和精度不完全相等但相近，因为超过分辨率的精度感觉没意义，低于分辨率的精度等效于低分辨率。如int具有32位精度，1分辨率
* quantum量程，即CPU最小时间片，与调度相关。在高级系统设置-性能-高级里如果选后台服务则会增加，WinServer默认后台。不清楚与Resolution是否是同一概念
* 计时器类型：TSC时间戳计时器（CPU提供，不精确，速度快），HPET高精度事件计时器（一般在南桥中），PMT平台计时器（主板芯片组上，访问开销大，作为保底；唤醒时使用），RTC实时时钟（BIOS记录现实时间，精度为秒级，CMOS电池供电断电不丢失）。QueryPerformanceCounter会优先用TSC。bcdedit中，useplatformclock选yes，会不使用TSC，会根据BIOS中是否启用了HPET而选择它或PMT

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

## Rundll32

* rundll32 xxx.dll,函数名 参数。会按GUI程序运行
* 创建：导出`void CALLBACK 函数名(HWND hwnd, HINSTANCE hinst, LPSTR lpszCmdLine, int nCmdShow)`
  * hwnd是作为本dll创建的所有窗口的owner，hinst是本dll的handle，nCmdShow是CreateProcess传递给它的表示是隐藏窗口创建的还是什么别的
  * 第三个参数：若加W后缀则为LPWSTR。不加后缀或A后缀为MBCS编码
  * CALLBACK是stdcall的意思，官方文档说必须要用它，否则rundll32在调用完函数后可能会崩溃。然而用它会修饰导出的符号，前面加_，后面加@16，或配合.def文件解决
  * 如果不需要命令行参数，实测也可以用void

## 内核/驱动相关

* https://github.com/hfiref0x/WinObjEx64 https://github.com/hfiref0x/SyscallTables https://github.com/hfiref0x/KDU
* https://github.com/MeeSong/Windows_OS_Internals_Curriculum_Resource_Kit-ACADEMIC/tree/master/CRK
* UMDF 2、Universal Driver：前者更简单，后者是发布/兼容性约束

## 内存

* 任务管理器
  * 使用中(in use) = 物理内存占用，但不包含缓存
  * 已提交：斜杠前称为committed charge，大概就是所有程序申请的总内存，包含已缓存。斜杠后称为limit = 总物理内存 + 页面文件
  * committed - in use = 缓存
  * 可用 = 缓存 + 空闲（不过占用条那里写的也叫“可用”）
* 进程
  * reserved和committed：前者基本不消耗资源，用于防止碎片化。后者一定要有某个backing store，后者会隐式前者。当写入一个已提交页面时会移动到工作集（占用物理内存）
  * working set：使用的物理内存，包括私有数据和共享数据。
  * committed：申请的私有内存，其中一部分在工作集中，还有一部分在页面文件中
  * 类型：Mapped File、Image、Shareable、Private Data、Stack、Heap

## 数字签名

* signtool替代：https://github.com/mtrojnar/osslsigncode https://www.ssesetup.com/download.html 前者支持PEM，后者是GUI好像不支持pem
* New-SelfSignedCertificate、Set-AuthenticodeSignature。只前者只能生成到系统储存里，后者可能不支持pem不支持用CWD的，不方便
* 签名的版本应该是V2，与证书V3无关
* 验证：signtool verify /pa。输出信息顺带验证：certutil -dump。实测对于后者，即使文件被修改了，也无法显示
* 时间戳服务器：如果不使用，证书过期后签名就失效了。但如果用，过期后还是有效的

## 保护

* 进程缓解
  * 禁用扩展点（EXTENSION_POINT_DISABLE）：禁止加载IME、Hook等
  * 禁止创建子进程
  * 控制流防护（CONTROL_FLOW_GUARD）
* 反调试：IsDebuggerPresent（C#为System.Diagnostics.Debugger.IsAttached）
* 自校验：WinVerifyTrust、把自身hash存在签名的外置数据中
* 加壳：保护完整性，一定程度反调试。虚拟化：与加壳正交，反编译。混淆器：obfuscator-llvm

## 参考

* Windows Internals 7th
* 深入浅出WindowsAPI程序设计
