---
title: PowerShell
category: windows
---

## 基本数据类型和运算符

* 常见的与bash不兼容的元字符：逗号、点号、波浪号
* 直接输入表达式就能输出运算结果，无需echo，即使在非REPL模式以及在函数中也一样
* bool运算用-and、-or，但好像-not可用!代替；也支持当作0和1进行运算，对应关系和C一样；pwsh7支持&&，但它是用来分别执行两边的命令的，而不是当作一个表达式（例如`$true && $false`跟用-and不一样）；pwsh7支持三元运算符
* 比较大小用`-eq`、`-nq`、`-lt`等；有的东西既不为真也不为假，-eq那俩bool类型都返回False；这些运算符也可对序列使用，达到过滤的效果
* 获得类型：`.GetType()`，判断类型：`$arr -is [array]`，类型转换：`[int]2.5`或`2.5 -as [int]`
* 数组([array]，其实是`object[]`)：声明用`@(元素1, ...)`，或用`元素1, ...`，改用强类型可用`[int[]]`；访问用中括号，索引从零开始，可为负，括号内也可是序列，则可一次取多个元素；长度固定，附加元素必须用+=，实际上是重新生成，直接给超出索引的地方赋值会报错；是引用类型，需要副本时用Clone()；对不可取索引的对象取[0]不会报错，仍返回它自己；生成数字序列：a..b，但好像无法指定间隔；可声明为`[System.Collections.ArrayList]`以获得更多特性；-contains返回bool，反过来用-in；也支持-eq等运算符，相当于filter，但必须数组对象在第一个，否则就与整个数组对象逻辑运算了；两个数组可以相加；可以解包到另一个变量数组
* 字典(其实是Hashtable)：`@{键=值; ...}`。声明时键不用加引号，访问用方括号或`.键`，不区分大小写；删除用Remove()，获取所有的键和值分别用.Keys和.Values
* 转义用反引号，反斜杠不具有特殊意义
* 支持进行容量计算：1kb+1kb，但结果为int的字节数
* 除了常见的类型以外还支持：datetime、timespan、guid、nullable、regex、scriptblock、switch、type等；把强类型转换为弱类型：`(Get-Variable var).Attributes.Clear()`；元组要用[ValueTuple]::Create()

## 字符串

* PS的字符串是个字符串类型的值，bash的是个命令。在字符串前加&，才能把它当作命令处理，且如有参数必须在外面
* -eq：判断字符串相等；+：合并字符串，但必须赋给一个变量或者**加括号**，否则加号可能被认为是普通的字符串的一部分，数字运算同理
* 替换：oldstr -replace pattern, newcontent；支持且必须用正则所以有的符号要转义，逗号不可忽略
* 匹配：-like使用dos通配符，*代表多个字符；-match使用正则（成功时填充$Matches）；-contains在这里又不把字符串看作序列了，变成完全匹配；这几个在前面都可以加no来取相反的集合，再在前面加c表示区分大小写，全都返回bool
* join：`127, 0, 0, 1 -join '.'`。注意优先级，最好加括号；分隔：``-split "`n"``
* 格式化字符串：`"{0:N0}" -f 1000`
* Select-String(sls)可使用正则搜索，但默认只匹配`$_.ToString()`的结果而不是交互式输出的结果，此时可用`| out-string -stream`，其中steram用于分行否则会视为一整个字符串，或者不用sls而用where的`$_.xxx -match`；sls的-Path可指定多个文件且可用通配符，如果传进来的不是路径，用`sls .*`可以替代%{$_.ToString()}，否则就会显示文件中所有的内容
* 提取字串：直接用[..]会变成object[]，要么用SubString，要么强转

## 变量

* 变量：`$[可选类型]变量名=值`、`$a=$b=1`、`$a,$b=$b,$a`
* 声明了但未赋值的变量的值等于（-eq）`$null`
* 变量名包含特殊字符或边界不明确时需用大括号
* 访问未声明的变量或数组越界不会报错，而是返回null，无输出
* 强类型变量即使赋字符串也会自动进行强转
* 声明变量还可以用New-Variable，具体见cmdlet的笔记
* 验证变量、虚拟驱动器、文件是否存在：Test-Path variable:变量名、驱动器名和冒号
* 清除变量：del variable:变量名或Remove-Variable(rv)，注意不能用del $变量名，这样会删除变量的内容指向的东西
* 查看（所有）变量：Get-Variable(gv)

### 预定义变量

* $home用户目录；VSC不支持~
* $host：基本信息 $pid：当前进程的pid $PSHome：安装路径
* `$?`上一次命令是否成功，bool类型；`$LastExitCode`上一次命令返回的数字值
* `$$`：上一次执行的代码的最后一个参数字符串；`$^`：第一个参数字符串
* $MyInvocation：有关当前命令的信息，如脚本的路径和文件名$myinvocation.mycommand.path或函数的名称$myinvocation.mycommand.name

### 虚拟驱动器

`ls 驱动器名加冒号`可打印内部的值，创建用New-Item(ni)，删除用ri/del/rm，注意此时无需加`$`。key可用dos通配符。只有使用里面的变量内容的时候才用`$`，如`$env:path`。使用Get-PSDrive可看到所有的驱动器，Get-Volume只看实际的卷。如果key中也含有变量，需用Set-Item。

* env：环境变量，其中修改path要用+=且记得加分号。永久修改需要用`[environment]::SetEnvironmentvariable`。
* alias：所有的别名。仅仅打印，要执行用&
* variable：所有的变量，与gv结果相同
* ${c:/123.txt}：直接取得路径指示的文件里的内容，必须要用大括号，必须是具体的路径，不能用变量替换
* HKCU、HKLM：注册表，可以直接cd进去。其实左边全是key，右边的是value。key用操纵文件夹的方式，而value要用Get/Set/Clear/Remove-ItemProperty [-name] ([-value])
* cd Regis+::：这个可能不是虚拟驱动器，因为有两个冒号

### 变量的作用域

* 变量的作用域：全局`$global`、本地`$local`、私有`$private`和脚本`$script`
* 默认`$local`，其他三个都需要用修饰符加冒号做前缀声明，再跟不加$的变量名，使用时可加但不必修饰符
* 同一作用域只能有一个类型的变量，用别的修饰符赋值可以覆盖，不同修饰符访问到的可能是同一个变量
* 进入新的子域后可以读取父域的$local，但赋值修改时会生成自己的$local变量，不会影响父域
* 父域的`$global`子域可以读取和写入，自己声明新的`$global`父域也能看到
* $private的变量子域无法读取，给它重新给它赋值成$global时会影响父域
* 使用点命令运行脚本会把$script扩展到$local，暂时来说$script和$local一样
* 使用`gv 变量名 | select Visibility`可以看到作用域​

## 流程控制

* if(){...}elseif(){...}else{...}；其中$null、0、空的字符串、数组、哈希表被认为假，非空的东西为真
* for ($i = 0; $i -lt $array.Count; $i++) {...}
* foreach ($item in $collection) {...}
* while、do while、do until：略，与C一致，只是不能用大于小于的符号
* switch有-wildcard/-regex/-case选项，要执行的内容用大括号，模式也可用大括号，则可用$_进行条件判断
* try catch finally与C#一样，throw直接跟字符串，用$Error.ExceptionMessage获取

## 函数

* function 函数名($参数1, $参数2=默认值) { 函数体 }；
* 如果不显式声明参数，仍可使用`$args`获取所有参数；否则可在函数名后加括号或在函数体中用param ($var, ...)，可加等号设定默认值
* .NET原生的函数，调用时不加圆括号会返回函数自己的信息（重载）；PS声明的函数调用无需括号，参数只用空格分隔就行
* 删除函数用`del function:xxx`
* switch类型的参数使用时只需指定参数名即可，即相当于flags，而bool类型的还要传$True
* 函数会将所有的表达式结果都算在返回值里，如果有多个就是object数组，接下来如果不赋给变量，就会显示在终端里；如果用了return，前面的表达式的仍会返回，return后就结束函数了；使用Write-Host才会只显示在终端、不算在返回值里
* 标准输入存放在$input数组中
* 如果使用filter替换function关键字，会变成流模式，每次只需处理$_；也可用begin{}process{}end{}进行处理，关键字可不变
* $args：所有参数的数组 $PsCmdlet：正在运行的cmdlet或函数 $PSScriptRoot：正在执行的脚本的路径 $MyInvocation

## 命令行参数

* 执行脚本时，可以用-b和-a指定命名实参
* 停止解析符号：`--%`：放在程序和参数之间，防止参数被解析成PS指令。比如参数中含有括号
* 运行脚本的时候也会加载，个人配置较多时很慢，可用`-nop`解决

对象
----

* 实例化对象：New-Object 类名 参数；也可以用.net的方式，调用`::new()`
* 添加成员时，对象可用管道传，则不用指定-InputObject(In) $ObjectName；其他参数默认顺序是：MemberType、Name、[Value]，后面可直接跟函数体
* PSMemberTypes可以是NoteProperty：“随后增加的属性”、ScriptMethod：函数和命令、AliasProperty：另外一个属性的别名、CodeProperty：通过静态的.Net方法返回属性的内容、CodeMethod：映射到静态的.NET方法、Property：真正的属性、Method：真正的方法
* Get-Member(gm)：查看对象的属性和方法，可用管道传给它；-MemberType(me)可指定类型
* 创建对象、继承对象：与C#类似，分别用class关键字和冒号，只不过成员的声明用的PS的语法

## 别名

* Get-Alias(alias)：根据别名查找原名；反过来用-Definition加原名来查找别名
* Set-Alias myalias cmd-let/func：无法像bash那样自动对参数链式展开

其他
----

* 小括号的语法作用是子表达式，即内部必须返回一个值；`$()`为子语句块，里面可以是if；在字符串内单纯的小括号不再具有语法效果，此时就也需要`$()`

## CMDLET

> https://ss64.com/ps/

* Invoke-WebRequest(iwr)：下载网页，结果的Content属性获取内容，`Headers.'xxx'`获取头
* Get-ChildItem(gci, ls)：获取文件夹内容，-r递归
* Invoke-Expression(iex)：调用字符串代表的命令，iex "$args"可直接带参数地执行传过来的命令；而`&`把后面的字符串仅解析成命令本身，在字符串外才可以跟那个命令的参数；比如要执行的程序文件名带空格，只能用&，iex和直接输没区别；如果命令本身又在引号里，就需要`iex "& $command"`，直接用&会把引号当作命令的一部分
* Get-Random：随机获取对象数组中的一个元素
* Get-Content(cat)
* Get-Process(ps)：获取的对象数组的元素具有kill()方法
* Start-Transaction：运行后会把当前会话的所有内容记录到文件中
* 创建软硬连接：https://docs.microsoft.com/zh-cn/powershell/wmf/5.0/feedback_symbolic；或用cmd -c
* prompt：改变每条命令前面的提示字符
* Start-Transcript：运行后会把当前会话的所有内容保存起来
* New-Variable varname -Value ...。变量名无需加$，-Option可指定readonly和constant，前者被赋值后不可修改但仍可被删除且可被-Force赋值；用这种方式在同作用域内在多次声明同名变量会报错
* Measure-Command -Expression {...}：测量运行时间
* Start-Job -ScriptBlock {...} ; $JobResponse = Get-Job | Receive-Job
* Compress-Archive、Expand-Archive
* Enter-PSSession -ComputerName RemoteComputer; Exit-PSSession

### meta

* Get-Location(pwd)：（以表格形式）显示当前路径；Get-Location | select -ExpandProperty Path的输出就和pwd一样了，或者先赋给变量，再用属性
* Get-Command(gcm)：显示匹配的cmdlet，模式可加`*`，一般用-None指定名词，-ParameterType加类型可查询以指定类型为参数的cmdlet。无参数调用比用`*`的结果少；Show-Command：显示一个gui窗口来填参数
* Get-History、Invoke-History *id*
* Update-Help

### 管道

* ForEach-Object(foreach和%)：`1..3 | % { echo $_ }`；可指定-Begin、-Process、-End，或直接用三个大括号代替，$ForEach表示索引，7之后支持-Parallel；第二种用法是不加大括号而跟字符串，代表取那一项成员
* Where-Object(where和?)：'You', 'Me' | ? { $_ -match 'u' }（投影）
* Select-Object(select)：选择多个属性（列），支持通配符；也可用于选择一定数量范围：-First(f)、-Last、-Skip、-SkipLast、-Index、-Unique、-Property（不加时默认用的这个）、-ExpandProperty（只显示属性的值，不显示属性名）；自定义列：@{Name=...;Expression={$_...}}
* Sort-Object(sort)：-Descending降序；如果某个对象不具有所指定的属性之一，则 cmdlet 会将该对象的属性值解释为 NULL，并将其放置在排序顺序的末尾；如果要多字段排序需要传哈希表对象
* Tee-Object(tee)：保存并显示管道输入的内容，会先创建文件再运行前面的命令；-Variable 变量名（不用加$）可以把结果保存到变量里
* Group-Object：进行分组，依据可为表达式；分组后有Count属性用于排序，Name属性为key，Group属性为内容
* Get-Unique(gu/unique): 从排序列表返回唯一项目。但其实只会比较前一个，所以如果没有先排序就无法发挥作用，只能保证相邻不重复
* Measure-Object(measure): 计算对象的数字属性以及字符串对象（如文本文件）中的字符数、单词数、行数、最大最小总和
* Compare-Object(compare/diff): 比较两个对象数组，SideIndicator的=>表示新增的对象；使用-Property参数可以比较对象的指定属性
* 任何平台的PS都不支持小于号的输入重定向，但Get-Content可以把文件中的内容写入管道

### 格式化和输入输出转换

一旦用了这些管道，就无法进一步处理了，也必须放到最后。默认会自动加Out-Default，等于ft/fl|oh。

* ConvertTo-Html: 将 Microsoft .NET Framework 对象转换为可在 Web 浏览器中显示的 HTML
* Export/Import-Clixml: 创建对象的基于 XML 的表示形式并将其存储在文件中
* Export/Import-Csv: 将 Microsoft .NET Framework 对象转换为一系列以逗号分隔的、长度可变的 (CSV) 字符串，并将这些字符串保存到一个 CSV 文件中
* Format-List(fl): 将输出的格式设置为属性列表，其中每个属性均各占一行显示。适合属性类型不同的情况。未知对象属性大于4个时会采用；fl *会显示所有属性
* Format-Table(ft): 将输出的格式设置为表，适合用于显示键值对，后可跟KeyName和对每一项的计算（比如/1KB）；-Autosize(auto) -Wrap：强制显示所有内容，如果不用，过长的字符串会显示...，这是因为PS是流模式，下一条命令的长度未知；如果要手动指定宽度和对齐方式等，每个键需要传一个哈希表给Property属性：`@{expression="KeyName"; width=40;label="Header"; alignment="center"}`；有一个-GroupBy参数可以在保持表格的情况下分组
* Format-Wide: 将对象的格式设置为只能显示每个对象的一个属性的宽表，样式类似于bash默认的ls
* 从用户处读取字符串输入：Read-Host
* ConvertTo-Json（默认-Depth为2）、ConvertFrom-Json

### 输出

* Write-Host：向控制台写文本，可用字符串插值（双引号内直接用$变量名），或者用类似`"asdf" $_ "asdf"`这样的式子，但空格还是会保留且不可省，除非加括号用字符串的处理方式；与echo(write-output)的区别是echo会把以空格分开的每个参数（当作$args数组？）在单独的一行输出，而它会把每个参数只用一个空格分隔后在一行输出
* echo不换行：`Write-Host "xxx" -NoNewLine`，但注意如果重定向输出，仍会输出到屏幕上
* Write-Debug：默认情况下（"SilentlyContinue"）不会往终端输出，$DebugPreference为"stop"时不允许使用此命令，为"Continue"时会显示，为"Inqure"时会显示并询问是否继续
* $ErrorActionPreference与debug类似，"SilentlyContinue"为隐藏，"Continue"为输出
* $host.UI.WriteErrorLine，WriteVerboseLine，WriteWarningLine：不受$DebugPreference控制
* Out-File: 将输出发送到文件，与`>`等效；Add-Content附加到文件
* Out-Null: 与`>$null`等效，删除输出，不将其发送到控制台
* Out-Printer: 将输出发送到打印机
* Out-Host -Paging：分页显示，功能比more更强大，而且后者在PS中是非流的

## 模块

* 可选安装PowerShellGet，但pwsh自带非常新的版本
* 如果不用`-Scope AllUsers`，默认会装到`$home\Documents\PowerShell\`，好处是不需要管理员权限
* `-Force`可避免从不信任的源安装的交互式警告
* `-AllowPrerelease`和指定最小版本的参数不允许一次装多个
* Get-Module显示已安装的模块，Update-Module更新所有或指定模块，Remove-Module与Import-Module相反，不是用来从硬盘上删除包的
* 这样配置下来每次启动要花1000多ms
* 因为breaking change，PSReadLine出现了严重的不兼容BUG，但是安装老版本时却永远说在运行中，因为它运行在所有会话中。必须先开CMD，用`pwsh -NoProfile -NonInteractive -Command "install-module psreadline -RequiredVersion 2.1.0 -Force"`才能安装

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Install-Module PSReadLine -AllowPrerelease -Scope AllUsers -Force -Proxy http://127.0.0.1:1080 # 好像现在自带？
Install-Module posh-git,oh-my-posh -Scope AllUsers -Force -Proxy http://127.0.0.1:1080

code $Profile # 在用户的Document/PowerShell中，一旦产生，再在地址栏里输powershell就会打开该我的文档里的文件夹，真的难以理解
Import-Module oh-my-posh # 自动导入posh-git
Set-Theme AgnosterPlus # 其实用这个也可以省略上一条导入
```

### 第三方模块

* Use-RawPipeline(Windows) 和 Use-PosixPipeline(Linux)

Profile文件
-----------

* http://www.pstips.net/powershell-auto-run-profile.html

## 与.Net的交互

### 当前目录

* 与pwd显示出来的不同，`[System.Environment]::CurrentDirectory`显示的是`C:\WINDOWS\system32`，只有主动对它赋值才会改变。使用相对路径时需要注意

## 在Linux上的兼容性

* 不支持作业控制，fg和bg不可用
* 其他不可用的cmdlet：https://docs.microsoft.com/zh-cn/powershell/scripting/whats-new/known-issues-ps6?view=powershell-6#command-availability
* 删除了别名：ls、cp、mv、rm、cat、man、mount、ps，使用的是本地命令。这会允许globbing，比如ls *.txt，但这将返回字符串

控制台（$host.ui.rawui）
-------------------------

* BackgroundColor：当前行的背景颜色，必须是System.ConsoleColor的枚举；ForegroundColor：字体的颜色；两者可以加横线作为日常参数
* CursorPosition：光标的位置
* WindowPosition：窗口的位置
* WindowSize：窗口的大小
* WindowTitle：窗口的标题

## 参考

* https://github.com/adambard/learnxinyminutes-docs/blob/master/powershell.html.markdown
* https://www.aloxaf.com/2019/03/powershell_miaoa/

### TODO

* https://www.jb51.net/article/115518.htm
* https://mva.microsoft.com/zh-cn/training-courses/powershell-18405
* https://www.pstips.net：看到 [Powershell错误处理](https://www.pstips.net/powershell-online-tutorials) 了
* https://www.cnblogs.com/sparkdev/tag/PowerShell/
* https://yuedu.baidu.com/ebook/d7b1a1767dd184254b35eefdc8d376eeafaa174e https://zhuanlan.zhihu.com/p/145043422
* https://stackoverflow.com/questions/47274532/difference-between-and
* https://leanpub.com/thebigbookofpowershellgotchas/read
* https://powershell.org/forums/forum/windows-powershell-qa/
* 粘贴数据时有bug，无法粘贴`1・`等，但可以单独粘贴那个点，或点前没内容也可以，注意不是`·`，好像键盘无法直接打出；又发现在Windows Terminal中无法用右键粘贴中文冒号分号等`1：`，但ctrl v可以
* 创建快捷方式：https://stackoverflow.com/questions/9701840

ISE无法识别utf8
