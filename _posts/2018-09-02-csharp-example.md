---
title: C#实例
---

## 打开“我的电脑”

```c#
System.Diagnostics.Process.Start("::{20D04FE0-3AEA-1069-A2D8-08002B30309D}");
System.Diagnostics.Process.Start("explorer.exe /n," + Environment.GetFolderPath(Environment.SpecialFolder.MyComputer));
```

## 当前程序的真实路径

* System.Reflection.Assembly.GetExecutingAssembly().Location
* System.Windows.Forms.Application.ExecutablePath，与MainModule.FileName一样；还有个Application.StartupPath，与CWD一样
* System.Diagnostics.Process.GetCurrentProcess().MainModule.FileName
* AppContext.BaseDirectory和AppDomain.CurrentDomain.BaseDirectory：目录，含有末尾反斜杠
* .NET6：Environment.ProcessPath

## 判断系统版本

* FX默认的Environment.OSVersion.ToString()为`Microsoft Windows NT 6.2.9200.0`。.NET5后正确，如`Microsoft Windows NT 10.0.19043.0`。FX添加manifest去掉兼容性中Windows10的id的注释后正确
* 搜索注册表：(string)Microsoft.Win32.Registry.LocalMachine.OpenSubKey(@"SOFTWARE\Microsoft\Windows NT\CurrentVersion")?.GetValue("ProductName") -> "Windows 10 Pro for Workstations"，CurrentBuild -> 22621
* System.Runtime.InteropServices.RuntimeInformation.OSDescription -> "Microsoft Windows 10.0.19043" 需4.7.1+
* 判断是不是Windows：OSVersion.Platform == PlatformID.Win32NT、RuntimeInformation.IsOSPlatform(OSPlatform.Windows)
* 当前程序以x86还是x64运行：RuntimeInformation.OSArchitecture、Environment.Is64BitProcess

### 判断框架版本

* Core3后，Environment.Version正确，如7.0.2。FX和Core3之前是4.0.30319.42000
* System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription为`.NET Framework 4.8.4300.0`或`.NET 7.0.2`。Core3之前是错的

## 检测、获得管理员权限和UAC

* 启动前：改manifest，用`<requestedExecutionLevel level="requireAdministrator" uiAccess="false" />`
  * uiAccess默认false，当想要像任务管理器或放大镜那样置顶时必要条件改为true
  * 即使不需要管理员权限，也最好添加它，否则会启用UAC虚拟化，程序向%ProgramFiles%或注册表里写入时会隐式重定向到权限低的地方，只适用于32位应用
* 启动后：这样可解决最常见的情景，但严格来说不正确，因为Administrator这个组与UAC之间没有必然联系
  ```c#
  if(!new System.Security.Principal.WindowsPrincipal(
      System.Security.Principal.WindowsIdentity.GetCurrent()
      ).IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator))
      { 不是管理员 }
  ```
* 在按钮或菜单上绘制UAC盾牌图标：
  ```c#
  [DllImport("user32")]
  static extern int SendMessage(IntPtr hWnd, uint Msg, int wParam, nint lParam);
  const uint BCM_SETSHIELD = 0x160C;
  SendMessage(btn1.Handle, BCM_SETSHIELD, 0, 1);
  ```

## 调用Office

* 第三方库：EPPlusSoftware/EPPlus nissl-lab/npoi dotnet/Open-XML-SDK mini-software/MiniExcel和MiniWord
* _Workbook和Workbook的区别：基本上一样，后者继承前者；后者表示当前Excel的Application里打开的对象，前者就是Excel对象
* 例子：https://github.com/dotnet/samples/tree/main/core/extensions/ExcelDemo

```c#
// COM，实测需要装WinSDK的一个工具
using Excel = Microsoft.Office.Interop.Excel;
Application app = new Excel.Application();
Workbook wb = app.Workbooks.Open(file);
Worksheet ws = wb.Worksheets.Item[1];
Range col = ws.Columns[Type.Missing, "B"];
wb.Save();
wb.Close();
app.Quit();
```

### 出错

* 无法将类型为“Microsoft.Office.Interop.Excel.ApplicationClass”的COM对象强制转换为接口类型“Microsoft.Office.Interop.Excel._Application”
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
