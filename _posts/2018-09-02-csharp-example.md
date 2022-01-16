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
WebBrowser wb = new WebBrowser();
wb.Navigate(@"c:\temp");
```

```
// 添加引用SystemRoot%\system32\SHELL32.dll
Shell32.ShellClass sh = new Shell32.ShellClass();
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
* System.Windows.Forms.Application.ExecutablePath 还有个 StartupPath
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
