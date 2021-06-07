---
title: C#实例
---

打开/获取“我的电脑”
-------------------

> https://zhidao.baidu.com/question/357446118.html
> http://bbs.csdn.net/topics/240045376

```
System.Diagnostics.Process.Start("::{20D04FE0-3AEA-1069-A2D8-08002B30309D}");
```

```
WebBrowser wb = new WebBrowser();
wb.Navigate(@"c:\temp");
```

```
// 添加引用SystemRoot%\system32\SHELL32.dll
Shell32.ShellClass sh = new Shell32.ShellClass();
sh.Explore(@"c:\");
```

```
string s = Environment.GetFolderPath(Environment.SpecialFolder.MyComputer);
Console.WriteLine(s); // 空行
var a = Directory.GetFiles(s); // 异常
System.Diagnostics.Process.Start("explorer.exe", "/n," + s); // 有效
```

```
string[] d = Environment.GetLogicalDrives();
DriveInfo[] di = DriveInfo.GetDrives();
```

当前程序的真实路径
------------------

* System.Reflection.Assembly.GetExecutingAssembly().Location
* System.Windows.Forms.Application.ExecutablePath
* System.Diagnostics.Process.GetCurrentProcess().MainModule.FileName

### 当前程序的MD5

* https://stackoverflow.com/questions/8875296/how-do-i-get-the-hash-of-current-exe

设置环境变量
------------

* Environment.SetEnvironmentVariable(string name, string path, EnvironmentVariableTarget target)

处理路径中的斜杠和反斜杠
------------------------

> https://github.com/xunit/xunit/commit/3c006e5db2fa79c9aa4b1b2718e611091f253ccc

```
if (Path.DirectorySeparatorChar == '/')    return "/" + codeBase;return codeBase.Replace('/', Path.DirectorySeparatorChar);
```

判断Windows10
-------------

如果不做任何操作，Environment.OSVersion.ToString()为`Microsoft Windows NT 6.2.9200.0`，在.NET5中返回正确的值了，如`Microsoft Windows NT 10.0.19043.0`。非Windows应该一直可以用。

第一种解决办法是添加一个manifest，去掉兼容性中Windows10的id的注释。

第二种办法搜索注册表，仅限Win：`(string)Microsoft.Win32.Registry.LocalMachine.OpenSubKey(@"SOFTWARE\Microsoft\Windows NT\CurrentVersion")?.GetValue("ProductName")`->`Windows 10 Pro for Workstations`

第三种办法，使用win32库的VersionHelpers.h

检测其他Win版本：https://stackoverflow.com/questions/2819934/detect-windows-version-in-net

System.Runtime.InteropServices.RuntimeInformation.OSDescription：`Microsoft Windows 10.0.19043`字符串

判断是不是Windows：OSVersion.Platform == PlatformID.Win32NT

### 判断框架版本

* 从Core3开始，Environment.Version能正确返回版本；之前一直是4.0.30319.42000
* System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription为`.NET Framework 4.8.4300.0`，但Core3之前的信息仍是错的

检测以及获得管理员权限
----------------------

> https://blog.csdn.net/linux7985/article/details/50525381

启动前：改manifest，用`<requestedExecutionLevel level="requireAdministrator" uiAccess="false" />`

启动后：

```
if(!new System.Security.Principal.WindowsPrincipal(
    System.Security.Principal.WindowsIdentity.GetCurrent()
    ).IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator))
    ....
```

在按钮或菜单上绘制UAC盾牌的图标：

```
[DllImport("user32", CharSet = CharSet.Auto, SetLastError = true)]
public static extern int SendMessage(IntPtr hWnd, UInt32 Msg, int wParam, IntPtr lParam);
public const UInt32 BCM_SETSHIELD = 0x160C;

SendMessage(button1.Handle, BCM_SETSHIELD, 0, (IntPtr)1);
```

调用Office
----------

第三方库：https://github.com/EPPlusSoftware/EPPlus

_Workbook和Workbook的区别：基本上一样，后者继承前者；后者表示当前Excel的Application里打开的对象，前者就是Excel对象

例子：https://github.com/dotnet/samples/tree/master/core/extensions/ExcelDemo

```
// 未测试
using Microsoft.Office.Interop.Excel;
Application app = new Application();
Workbook wb = app.Workbooks.Open(file);
Worksheet ws = wb.Worksheets.Item[1];
Range col = ws.Columns[Type.Missing, "B"];
wb.Save();
wb.Close();
app.Quit();
```

### 出错

> 无法将类型为“Microsoft.Office.Interop.Excel.ApplicationClass”的COM对象强制转换为接口类型“Microsoft.Office.Interop.Excel._Application”……

* 清理注册表的方法：https://www.cnblogs.com/sunxin88/articles/3456395.html
* 重新安装WPS，再完整卸载

System.Diagnostics.Process类
----------------------------

* 可以new出来，设置StartInfo属性，然后Start；也可以用静态的StartNew方法，传ProcessStartInfo实例
* 静态方法：GetCurrentProcess、GetProcesses、GetProcessesByName/ID
* 不清楚Close和Kill的区别，看起来都会强行关闭，可能是发送的信号不同
* WaitForExit()、ExitCode
* EnableRaisingEvents为true后可用Exited事件

### StartInfo

* UseShellExecute：当不指定真正的exe时需要设为true，例如FileName为网页和txt文件，这样才能读取关联；需要重定向输入输出流时必须为false；默认值FX为true，Core为false；基本上为false就是在当前控制台中执行
* WorkingDirectory：当UseShellExecute为true时，为需要执行的文件的路径，如果为空，与自己相同，而工作目录直接与自己相同；为false时，为设定那个文件的工作目录
* Verb：启动时的谓词，是个字符串，可用Verbs获取所有可用的；一般包括Edit、Print等，不同后缀可用的不同
* WindowStyle：普通、最小/大化、隐藏；UseShellExecute必须为true
* CreateNoWindow：默认为false，为true时必须确保目标程序能自己结束，否则就必须用Kill；UseShellExecute为true或是Core会忽略该选项，前者一定会创建新窗口，而后者不支持创建窗口

### 执行CMD命令

* 以管理员权限运行程序：Verb="runas"；但UseShellExecute必须为true
* 如果Path中存在同名程序须小心，发生过CMD指定的是System32下的但Process.Start执行的是在后面自定义的

```
// 效果和执行了一个无回显的batch脚本一样，不会显示命令，只会有输出
Process.Start(
    new ProcessStartInfo("cmd.exe", "/C" + "...") { UseShellExecute = false }).WaitForExit();
// 有人说需要加"&exit"，表示前面一个命令不管是否执行成功都执行后面的exit；但我用了/C，应该只会更好。而且最好就是通过cmd调用，如果直接用想执行的程序，那样不能用shell命令（如>nul)，而且不会推动命令行位置移动，等自己再输出时会把“已经输出了的”覆盖掉

// 效果和用户在交互界面手动输命令一样，每次会有路径，会显示“输入”了的内容
var psi = new ProcessStartInfo("cmd.exe") // 这样就不能/C了，否则直接退出
{
    UseShellExecute = false,
    RedirectStandardInput = true,
    // 不重定向输出就会直接输出到终端，否则不会输出，可以自己处理p.StandardOutput
    // RedirectStandardOutput = true,
    // RedirectStandardError = true,
    // 当上面两项为false，CreateNoWindow为true时也无法输出信息，不懂
};
var p = Process.Start(psi);
// 加上下面这句仍会显示一句路径加@echo off，之后才只会有结果
// p.StandardInput.WriteLine(@"@echo off");
p.StandardInput.WriteLine(@"echo 123");
p.StandardInput.WriteLine("exit"); // 必须有，否则会一直等待；要不就Kill
p.WaitForExit();

// 控制台用UTF8输出，未测试是否有效：
cmd.Start(); // 重定向了流，CreateNoWindow = true,UseShellExecute = false
cmd.StandardInput.WriteLine("chcp 65001");
cmd.StandardInput.Flush();
cmd.StandardInput.Close();
Console.OutputEncoding = Encoding.UTF8;
```
