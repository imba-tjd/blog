---
title: PowerShell实例
---

获取系统信息
------------

默认的Get-ComputerInfo太慢，而且不知道为什么几乎没有获取到什么信息；但这两个都暂时在Linux上用不了。

> https://gallery.technet.microsoft.com/scriptcenter/PowerShell-System-571521d1
>
>     function Get-SystemInfo
>     {
>      param($ComputerName = $env:COMPUTERNAME)
>
>      $header = 'Hostname','OSName','OSVersion','OSManufacturer','OSConfiguration','OS Build Type','RegisteredOwner','RegisteredOrganization','Product ID','Original Install Date','System Boot Time','System Manufacturer','System Model','System Type','Processor(s)','BIOS Version','Windows Directory','System Directory','Boot Device','System Locale','Input Locale','Time Zone','Total Physical Memory','Available Physical Memory','Virtual Memory: Max Size','Virtual Memory: Available','Virtual Memory: In Use','Page File Location(s)','Domain','Logon Server','Hotfix(s)','Network Card(s)'
>
>      systeminfo.exe /FO CSV /S $ComputerName |
>      Select-Object -Skip 1 |
>      ConvertFrom-CSV -Header $header
>     }

> - This cmdlet takes a while to load for me because it loads a lot of information that I currently won't need.
> Is there a way to only load the section you want, i.e.: only load "Computer information" or "Operating system information"
>
>  
>
> - The faster way is to use WMI to query just the information you need. See https://docs.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance?view=powershell-6
> 或看这个：https://docs.microsoft.com/zh-cn/powershell/scripting/getting-started/cookbooks/collecting-information-about-computers?view=powershell-6
>
> ~~但这两个在我的电脑上都运行不了：客户端无法连接到请求中指定的目标。 请验证该目标上的服务是否正在运行以及是否正在接受请求。 有关目标(通常是 IIS 或 WinRM)上运行的 WS 管理服务，请查阅日志和文档。 如果目标是 WinRM 服务，则在目标上运行以下命令来分析和配置 WinRM 服务: "winrm quickconfig"。
> ~~去掉`-ComputerName .`就好了

创建选择菜单
------------

    $SwitchUser = ([System.Management.Automation.Host.ChoiceDescription]"&Switchuser")
    $LoginOff = ([System.Management.Automation.Host.ChoiceDescription]"&LoginOff")
    $Lock= ([System.Management.Automation.Host.ChoiceDescription]"&Lock")
    $Reboot= ([System.Management.Automation.Host.ChoiceDescription]"&Reboot")
    $Sleep= ([System.Management.Automation.Host.ChoiceDescription]"&Sleep")

    $selection = [System.Management.Automation.Host.ChoiceDescription[]]($SwitchUser,$LoginOff,$Lock,$Reboot,$Sleep)
    $answer=$Host.UI.PromptForChoice('接下来做什么事呢？','请选择:',$selection,1)
    "您选择的是："
    switch($answer)
    {
    0 {"切换用户"}
    1 {"注销"}
    2 {"锁定"}
    3 {"重启"}
    4 {"休眠"}
    }

效果：

    接下来做什么事呢？
    请选择:
    [S] Switchuser  [L] LoginOff  [L] Lock  [R] Reboot  [S] Sleep  [?] 帮助 (默认值为“L”): Reboot
    您选择的是：
    重启

 

 

 

创建快捷方式
------------

* https://stackoverflow.com/questions/9701840


