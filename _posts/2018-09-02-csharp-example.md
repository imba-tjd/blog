---
title: C#实例
---

打开/获取“我的电脑”
-------------------

```
System.Diagnostics.Process.Start("::{20D04FE0-3AEA-1069-A2D8-08002B30309D}");
System.Diagnostics.Process.Start("explorer.exe /n," + Environment.GetFolderPath(Environment.SpecialFolder.MyComputer));
```

当前程序的真实路径
------------------

* System.Reflection.Assembly.GetExecutingAssembly().Location
* System.Windows.Forms.Application.ExecutablePath，与MainModule.FileName一样；还有个Application.StartupPath，与CWD一样
* System.Diagnostics.Process.GetCurrentProcess().MainModule.FileName
* AppContext.BaseDirectory和AppDomain.CurrentDomain.BaseDirectory：目录，含有末尾反斜杠
* .NET6：Environment.ProcessPath

设置环境变量
------------

* Environment.SetEnvironmentVariable(string name, string path, EnvironmentVariableTarget target)

判断Windows10
-------------

如果不做任何操作，Environment.OSVersion.ToString()为`Microsoft Windows NT 6.2.9200.0`，在.NET5中返回正确的值了，如`Microsoft Windows NT 10.0.19043.0`。

第一种解决办法是添加一个manifest，去掉兼容性中Windows10的id的注释。

第二种办法搜索注册表，仅限Win：`(string)Microsoft.Win32.Registry.LocalMachine.OpenSubKey(@"SOFTWARE\Microsoft\Windows NT\CurrentVersion")?.GetValue("ProductName")`->`Windows 10 Pro for Workstations`

第三种办法，使用win32库的VersionHelpers.h

检测其他Win版本：https://stackoverflow.com/questions/2819934/detect-windows-version-in-net

System.Runtime.InteropServices.RuntimeInformation.OSDescription：`Microsoft Windows 10.0.19043`字符串

判断是不是Windows：OSVersion.Platform == PlatformID.Win32NT

当前程序以x86还是x64运行：System.Runtime.InteropServices.RuntimeInformation.OSArchitecture、Environment.Is64BitProcess

### 判断框架版本

* Core3后，Environment.Version能正确返回版本，如7.0.2。FX和Core3之前是4.0.30319.42000
* System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription为`.NET Framework 4.8.4300.0`或`.NET 7.0.2`。Core3之前是错的

检测以及获得管理员权限
----------------------

> https://blog.csdn.net/linux7985/article/details/50525381

启动前：改manifest，用`<requestedExecutionLevel level="requireAdministrator" uiAccess="false" />`

启动后：

```
if(!new System.Security.Principal.WindowsPrincipal(
    System.Security.Principal.WindowsIdentity.GetCurrent()
    ).IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator))
    { 不是管理员 }
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

## Windows服务

* Core：https://learn.microsoft.com/zh-cn/dotnet/core/extensions/windows-service
* FX：https://learn.microsoft.com/zh-cn/dotnet/framework/windows-services

## 从资源中加载程序集

```c#
AppDomain.CurrentDomain.AssemblyResolve += OnResolveAssembly; // 当找不到时自动执行此委托
static Assembly OnResolveAssembly(object sender, ResolveEventArgs e) {
    var thisAssembly = Assembly.GetExecutingAssembly();
    foreach (string name in thisAssembly.GetManifestResourceNames()) // 当前程序集的所有资源名。以项目名.开头，因此下面要用EndsWith
        if (name.EndsWith(e.Name + ".dll")) {
            var stream = thisAssembly.GetManifestResourceStream(name);
            var block = new byte[stream.Length];
            stream.Read(block, 0, block.Length);
            return Assembly.Load(block);
        }
    return null;
}
```
