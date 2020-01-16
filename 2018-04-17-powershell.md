--- layout: post title: PowerShell date: 2018-04-17 14:33:30.000000000 -05:00 type: post parent\_id: '0' published: true password: '' status: publish categories: - 未分类 tags: [] meta: \_oembed\_5598e64745cf99a7600fa0857c665664: "{{unknown}}" \_oembed\_0e7378cefd1e57849c824630f4c9bab6: "{{unknown}}" \_oembed\_aaa26c01e15cb914b6e5a6de397947d8: "{{unknown}}" \_wpcom\_is\_markdown: '1' \_oembed\_d7338080ffbf24cab54638651582e086: "{{unknown}}" \_rest\_api\_published: '1' \_rest\_api\_client\_id: "-1" \_publicize\_job\_id: '16881917929' timeline\_notification: '1523946812' \_oembed\_8ac68fd9ed91958019183efbdb28c5a2: "{{unknown}}" \_oembed\_cae0ca79fc9a7ce657d3b7a6593f83c8: "{{unknown}}" \_oembed\_b7c865753eca74d97e4a44f0031f0ef9: "{{unknown}}" \_oembed\_0209514aa0e34b25bbdd275864aafb96: "{{unknown}}" \_oembed\_ca3b82ae8906ae3ad8954bf1d932b96a: "{{unknown}}" \_oembed\_fd273e661b44042974795b76bcbbfdcc: "{{unknown}}" \_edit\_last: '119115352' \_oembed\_eb47bb990532fa93fcc65c682ce6f11d: "{{unknown}}" \_oembed\_a69b2a1d4a1855b7fab22bf75105abdb: "{{unknown}}" author: login: imbalancedweb email: imba.tjd@gmail.com display\_name: imba-tjd first\_name: '' last\_name: '' permalink: "/2018/04/17/powershell/" ---

> https://www.jb51.net/article/115518.htm
> https://mva.microsoft.com/zh-cn/training-courses/powershell-18405
> https://www.pstips.net：看到 [Powershell错误处理](https://www.pstips.net/powershell-online-tutorials) 了

语法
====

变量
----

-   变量：\$变量名=值；因为是动态类型，下面的类型也可以这样复制给变量；声明但未赋值的变量的值等于（-eq）\$null；直接使用\$变量名就可以显示其值，无需用echo；如果需要用特殊字符做变量名，如\$和"，需要用大括号括起来；支持连续赋值；一次性多个赋值或交换两个变量：`$value1,$value2=$value2,$value1`；访问未声明的变量或数组越界不会报错，而是返回null，无输出
-   在\$前加[类型]可申明强类型变量，除了常见的类型以外，还支持：datetime、guid、nullable、psobject、regex、scriptblock、switch、timespan、type、XML；可以使用`(Get-Variable var).Attributes.Clear()`把强类型转换为弱类型；使用-is可以判断类型
-   声明变量还可以用New-Variable *name* -Value ...，name无需\$，-Option可指定readonly和constant，前者被赋值后不可修改但仍可被删除且可被-Force赋值；同作用域内在多次声明同一个变量会报错
-   验证变量或虚拟驱动器是否存在：Test-Path + 驱动器名和冒号/variable:*变量名*；使用del variable:*变量名*或Remove-Variable(rv)可清除变量，注意不能用del \$变量名，因为这样会删除变量的内容指向的东西
-   可以对变量进行一些限制，如不为空、范围等，详见：https://www.pstips.net/powershell-variable-management-behind-the-scenes.html
-   数组([array])：声明用@(元素1, ...)，或者直接用逗号；多行的数据实际上是数组；索引从零开始，但可为负，比如最后一个元素可用使用-1访问；访问用\$*name*[]，括号内用逗号表达式可一次获取多个元素，也可为序列，则`$books[($books.Count)..0]`即可倒序输出；元素类型可不同，如果要强类型，在\$*变量名*前面加`[int[]]`；附加元素用+=，但实际上是重新生成；是引用类型，需要副本时用Clone方法
-   哈希表([hashtable])：@{键=值; ...}，访问用方括号或点加键；值可直接用逗号表达式来声明数组；删除元素用Remove方法
-   强制类型转换用方括号，对于数字是四舍五入，比如[int] 1.7为2；定义函数的返回值或指定类型的变量时也用方括号
-   创建对象：New-Object 类名 参数；也可以用.net的方式
-   生成序列：a..b
-   获得变量类型：GetType()方法，大小写敏感；判断类型：`$name -is [array]`
-   动态生成环境变量：https://stackoverflow.com/questions/30911306/how-to-dynamically-create-an-environment-variable

### 预定义变量

-   单独使用Get-Variable(gv)可以看到所有变量
-   \$null，\$true，\$false
-   \$\_管道中当前对象
-   \$args所有参数的数组
-   \$pid当前ps进程的pid
-   \$home用户目录
-   \$host显示ps的基本信息
-   \$?显示上一次命令是否成功，为布尔类型，\$LastExitCode为上一次命令返回的数字值
-   \$\$：上一次执行的代码的最后一个参数字符串，\$\^：第一个参数字符串
-   \$MyInvocation：具有有关当前命令的信息，如脚本的路径和文件名\$myinvocation.mycommand.path或函数的名称\$myinvocation.mycommand.name
-   \$Profile显示配置文件路径
-   \$PSHome：PS的安装路径
-   \$PSScriptRoot：正在执行的脚本的路径
-   \$PsCmdlet：正在运行的cmdlet或函数

### 虚拟驱动器

以冒号结尾，使用ls可以打印它们所有的值，此时不需要加\$。key可用dos通配符。只有使用里面的变量内容的时候才用\$，如`$env:path`。使用Get-PSDrive可看到所有的驱动器，Get-Volume只看实际的卷。

-   env：环境变量。永久修改需要用`[environment]::SetEnvironmentvariable`
-   alias：所有的别名。仅仅打印，如果要执行可用&
-   variable：所有的变量
-   \${c:/123.txt}：直接取得路径指示的文件里的内容，必须要用大括号，必须是具体的路径，不能用变量替换
-   HKCU、HKLM：注册表，可以直接cd进去。其实左边全是key，右边的是value。key用操纵文件夹的方式，而value要用Get/Set/Clear/Remove-ItemProperty [-name] ([-value])
-   修改path可以用+=，但是仍要加分号，且是临时修改；永久修改用上面的或者`setx /M PATH "%PATH%;…"`
-   cd Registry::：这个可能不是虚拟驱动器，因为有两个冒号

### 变量的作用域

-   变量的作用域：全局\$global、本地\$local、私有\$private和脚本\$script
-   默认\$local，其他三个都需要用修饰符加冒号做前缀声明，再跟不加\$的变量名，使用时可加但不必修饰符
-   同一作用域只能有一个类型的变量，用别的修饰符赋值可以覆盖，不同修饰符访问到的可能是同一个变量
-   进入新的子域后可以读取父域的\$local，但赋值修改时会生成自己的\$local变量，不会影响父域
-   父域的\$global子域可以读取和写入，自己声明新的\$global父域也能看到
-   \$private的变量子域无法读取，给它重新给它赋值成\$global时会影响父域
-   使用点命令运行脚本会把\$script扩展到\$local，暂时来说\$script和\$local一样
-   使用`gv 变量名 | select Visibility`可以看到作用域​

### 条件操作符

``` {.wp-block-code}
(3,4,5) -contains 2 # False
(3,4,5) -contains 5 # True
(3,4,5) -notcontains 6 # True
2 -eq 10 # False
"A" -eq "a" # True
"A" -ieq "a" # True
"A" -ceq "a" # False
1gb -lt 1gb+1 # True
1gb -lt 1gb-1 # False
$a= 2 -eq 3; $a # False
-not $a # True
!($a) # True
# -and ：和
# -or ：或
# -xor ：异或

# 过滤数组中的元素
1,2,3,4,3,2,1 -eq 3
3
3

1,2,3,4,3,2,1 -ne 3
1
2
4
2
1
```

选择和循环
----------

-   if(){...}elseif(){...}else{...}
-   for (\$i = 0; \$i -lt \$array.Count; \$i++) {...}
-   foreach (\$item in \$collection) {...}
-   while、do while、do until：略，与C一致，只是不能用大于小于的符号

### switch

``` {.wp-block-code}
$value=2
switch [-wildcard/-regex/-case] ($value) # 比较字符串时默认-eq不区分大小写，-case后变成-ceq区分
{
    {$_ -lt 5 }  { echo 小于5 } # 会输出
    25 { echo 等于25 } # 因匹配失败不输出
    {$_ -gt 0 }  { echo 大于0; break; } # 也会输出，即满足多个条件时都处理
    {$_ -lt 100} { echo 小于100 } # 因break不输出
    Default {"没有匹配条件"}
}
```

函数
----

-   function 函数名 { Test-Connection -Count 2 -ComputerName \$args }
-   如果不显式声明参数，仍可使用\$args获取所有参数；否则可在函数名后加括号或在函数体中用param (\$var, ...)，可加等号设定默认值
-   .NET原生的函数，调用时不加圆括号会返回函数自己的信息（重载）；PS声明的函数调用无需括号，参数只用空格分隔就行
-   删除函数用`del function:xxx`
-   [switch]类型的参数：使用时只需指定参数名即可，而bool类型的还要传\$True
-   函数会将所有的表达式结果都算在返回值里，如果有多个就是object数组，接下来如果不赋给变量，就会显示在终端里；如果用了return，前面的表达式的仍会返回，return后就结束函数了；使用Write-Host才会只显示在终端、不算在返回值里
-   标准输入存放在\$input数组中，但这样效率低
-   如果使用filter替换function关键字，会变成流模式，每次只需处理\$\_；还可用begin{}process{}end{}块进行统一处理

处理字符串
----------

-   PS的字符串是个字符串类型的值，bash的是个命令
-   +：合并字符串，但必须赋给一个变量或者加括号，否则加号可能被认为是普通的字符串的一部分；数字运算同理
-   -eq：判断字符串相等
-   替换：源文本 -replace *匹配模式*, *替换内容*；支持且必须用正则所以有的符号要转义，逗号不可忽略
-   -like使用dos通配符，\*代表多个字符
-   -match使用正则（成功时填充\$Matches）
-   -contains完全匹配
-   这几个命令最前面都可以加no来取相反的集合，可以再在前面加c表示区分大小写（默认不区分）
-   -is/-isnot "System.String"可以判断类型
-   -join：127, 0, 0, 1 -join '.'；注意优先级，最好加括号
-   -split "\`n"：以换行符分隔字符串
-   -f可以使用跟String.Format类似的用法，如`"{0:N0}" -f 1000`
-   在字符串前加&，可以把它当作命令处理，如有参数必须在外面
-   Select-String(sls)可使用正则搜索，但正常情况下只会搜索\$\_.ToString()后的结果而不是默认的输出形式，此时可以用`out-string -stream`，或者不用sls而用where的`$_.属性 -match`
-   sls的-Path参数可指定多个文件且可用通配符；如果传进来的不是路径，用`sls .*`可以替代%{\$\_.ToString()}，否则就会显示文件中所有的内容
-   提取字串：用[..]会变成object[]，搜过大概只能用SubString

命令行参数
----------

-   param(\$a, \$b)：表示接受两个命令行参数参数；前面可以加类型，传实参时如果不匹配会报错；后面加等号可以设定默认值
-   执行脚本时，可以用-b和-a指定命名实参
-   停止解析符号：`--%`：放在程序和参数之间，防止参数被解析成PS指令。比如参数中含有括号

对象
----

-   new-object声明对象
-   添加成员时，对象可用管道传，则不用指定-InputObject(In) \$ObjectName；其他参数默认顺序是：MemberType、Name、[Value]，后面可直接跟函数体
-   PSMemberTypes可以是NoteProperty：“随后增加的属性”、ScriptMethod：函数和命令、AliasProperty：另外一个属性的别名、CodeProperty：通过静态的.Net方法返回属性的内容、CodeMethod：映射到静态的.NET方法、Property：真正的属性、Method：真正的方法
-   Get-Member(gm)：查看对象的属性和方法，可用管道传给它；-MemberType(me)可指定类型

别名
----

### 操作别名本身

-   Get-Alias(alias)：查看别名，如果要查找指定的，可以遍历时用\$\_.Name/Definition.Contains
-   Set-Alias设置别名，需要指定-Name和-Value，或者new-item -path alias:xx -value xxx
-   删除别名：del alias:*别名*
-   导出和导入别名：Export-Alias alias.ps1、Import-Alias [-Force] alias.ps1

### 规则

-   Get: g, Set: s, Item: i, Location: l, Command: cm, Alias: al
-   PS6取消的别名：sc——Set-Content；curl/wget——Invoke-WebRequest

其他
----

-   使用反引号转义，比如`` `n ``是换行符，在末尾加可以在下一行继续输入等；`` `$ ``会把\$解释为普通字符；正则中仍用反斜杠
-   PS可以直接进行加减乘除取模运算，可以识别容量（KB、MB、GB)
-   任何平台的PS都不支持小于号的输入重定向，但Get-Content可以把文件中的内容写入管道
-   单纯使用括号貌似就能实现子表达式，不知道和\$()有无区别

CMDLET
======

> https://ss64.com/ps/

基本信息
--------

-   Get-Location(pwd)：（以表格形式）显示当前路径；Get-Location | select -ExpandProperty Path的输出就和pwd一样了，或者先赋给变量，再用属性
-   Get-Command \*：获取所有的cmd-let，不用\*就是一般关键的，加上-ListImported只列出特别关键的，-CommandType指定命令类型。但只会显示一些属性，不会有用法
-   Get-History、Invoke-History *id*

未分类
------

-   Invoke-WebRequest(iwr)：下载网页
-   Get-ChildItem(gci, ls)：获取文件夹内容，-r递归
-   Invoke-Expression(iex)：调用字符串代表的命令，iex "\$args"可直接带参数地执行传过来的命令；而`&`把后面的字符串仅解析成命令本身，在字符串外才可以跟那个命令的参数；比如要执行的程序文件名带空格，只能用&，iex和直接输没区别；如果命令本身又在引号里，就需要`iex "& $command"`，直接用&会把引号当作命令的一部分
-   Get-Random：随机获取对象数组中的一个元素
-   Get-Content(cat)
-   Get-Process(ps)：获取的对象数组的元素具有kill()方法
-   Start-Transaction：运行后会把当前会话的所有内容记录到文件中
-   创建软硬连接：https://docs.microsoft.com/zh-cn/powershell/wmf/5.0/feedback\_symbolic；或用cmd -c
-   prompt：改变每条命令前面的提示字符
-   Start-Transcript：运行后会把当前会话的所有内容保存起来

管道
----

-   ForEach-Object(foreach和%)：1..3 | % { echo \$\_ }；可指定-Begin、-Process、-End，或直接用三个大括号代替，\$ForEach表示索引
-   Where-Object(where和?)：'You', 'Me' | ? { \$\_ -match 'u' }
-   Select-Object(select)：选择属性（投影），支持通配符，单用星号相当于fl \*；可指定-First(f)、-Last、-Skip、-SkipLast、-Index、-Unique、-Property（不加时默认用的这个）、-ExpandProperty（只显示属性的值，不显示属性名）；自定义列：@{Name=...;Expression={\$\_...}}
-   Sort-Object(sort)：-Descending降序；如果某个对象不具有所指定的属性之一，则 cmdlet 会将该对象的属性值解释为 NULL，并将其放置在排序顺序的末尾；如果要多字段排序需要传哈希表对象
-   Tee-Object(tee)：保存并显示管道输入的内容，会先创建文件再运行前面的命令；-Variable 变量名（不用加\$）可以把结果保存到变量里
-   Group-Object：进行分组，依据可为表达式；分组后有Count属性用于排序，Name属性为key，Group属性为内容
-   Get-Unique(gu/unique): 从排序列表返回唯一项目。但其实只会比较前一个，所以如果没有先排序就无法发挥作用，只能保证相邻不重复
-   Measure-Object(measure): 计算对象的数字属性以及字符串对象（如文本文件）中的字符数、单词数、行数、最大最小总和
-   Compare-Object(compare/diff): 比较两个对象数组，SideIndicator的=\>表示新增的对象；使用-Property参数可以比较对象的指定属性

格式化或输入输出转换
--------------------

一旦用了这些管道，就无法进一步处理了，也必须放到最后。默认会自动加Out-Default，等于ft/fl|oh。

-   ConvertTo-Html: 将 Microsoft .NET Framework 对象转换为可在 Web 浏览器中显示的 HTML
-   Export/Import-Clixml: 创建对象的基于 XML 的表示形式并将其存储在文件中
-   Export/Import-Csv: 将 Microsoft .NET Framework 对象转换为一系列以逗号分隔的、长度可变的 (CSV) 字符串，并将这些字符串保存到一个 CSV 文件中
-   Format-List(fl): 将输出的格式设置为属性列表，其中每个属性均各占一行显示。适合属性类型不同的情况。未知对象属性大于4个时会采用；fl \*会显示所有属性
-   Format-Table(ft): 将输出的格式设置为表，适合用于显示键值对，后可跟KeyName和对每一项的计算（比如/1KB）；-Autosize(auto) -Wrap：强制显示所有内容，如果不用，过长的字符串会显示...，这是因为PS是流模式，下一条命令的长度未知；如果要手动指定宽度和对齐方式等，每个键需要传一个哈希表给Property属性：`@{expression="KeyName"; width=40;label="Header"; alignment="center"}`；有一个-GroupBy参数可以在保持表格的情况下分组
-   Format-Wide: 将对象的格式设置为只能显示每个对象的一个属性的宽表，样式类似于bash默认的ls

### 输出

-   Write-Host：向控制台写文本，可用字符串插值（双引号内直接用\$变量名），或者用类似`"asdf" $_ "asdf"`这样的式子，但空格还是会保留且不可省，除非加括号用字符串的处理方式；与echo(write-output)的区别是echo会把以空格分开的每个参数（当作\$args数组？）在单独的一行输出，而它会把每个参数只用一个空格分隔后在一行输出
-   Write-Debug：默认情况下（"SilentlyContinue"）不会往终端输出，\$DebugPreference为"stop"时不允许使用此命令，为"Continue"时会显示，为"Inqure"时会显示并询问是否继续
-   \$ErrorActionPreference与debug类似，"SilentlyContinue"为隐藏，"Continue"为输出
-   \$host.UI.WriteErrorLine，WriteVerboseLine，WriteWarningLine：不受\$DebugPreference控制
-   Out-File: 将输出发送到文件；Add-Content附加到文件
-   Out-Null: 与\>\$null等效，删除输出，不将其发送到控制台
-   Out-Printer: 将输出发送到打印机
-   Out-String -stream: 将对象作为一列字符串发送到主机；非常重要，使用之后再用sls才有grep的效果；如果不加-stream会变成一整个字符串
-   Out-Host -Paging：分页显示，功能比more更强大，而且后者在PS中是非流的

非语法
======

Profile文件
-----------

-   http://www.pstips.net/powershell-auto-run-profile.html

与.Net的交互
------------

### 当前目录

-   与pwd显示出来的不同，`[System.Environment]::CurrentDirectory`显示的是`C:\WINDOWS\system32`，只有主动对它赋值才会改变。使用相对路径时需要注意

在Linux上的兼容性
-----------------

-   不支持作业控制，fg和bg不可用
-   其他不可用的cmdlet：https://docs.microsoft.com/zh-cn/powershell/scripting/whats-new/known-issues-ps6?view=powershell-6\#command-availability
-   删除了别名：ls、cp、mv、rm、cat、man、mount、ps，使用的是本地命令。这会允许globbing，比如ls \*.txt，但这将返回字符串

控制台（\$host.ui.rawui）
-------------------------

-   BackgroundColor：当前行的背景颜色，必须是System.ConsoleColor的枚举；ForegroundColor：字体的颜色；两者可以加横线作为日常参数
-   CursorPosition：光标的位置
-   WindowPosition：窗口的位置
-   WindowSize：窗口的大小
-   WindowTitle：窗口的标题


