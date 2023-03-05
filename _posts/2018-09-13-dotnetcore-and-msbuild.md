---
title: .NET Core和MSBuild
category: dotnet
---

## 安装

* Windows：https://dot.net
* docker映像：https://hub.docker.com/_/microsoft-dotnet
* Linux，不支持x86：https://docs.microsoft.com/zh-cn/dotnet/core/install/linux-debian
* dotnet-install.sh自动安装脚本，支持无root权限安装，但只是用于CI环境临时使用，且需要手动添加Path：`export PATH=~/.dotnet:$PATH`、`$env:Path="$env:LocalAppdata\Microsoft\dotnet;"+$env:Path`
* 源代码：https://source.dot.net/，部分API只会显示Linux的。FX的源码：https://referencesource.microsoft.com

## [CSProj](https://docs.microsoft.com/zh-cn/dotnet/core/tools/csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">

<PropertyGroup>：
StartupObject：Namespace.Class # 存在多个Main时指定入口
OutputType：Exe # 默认Library
TargetFramework：net5.0 # 指定多个加s和分号
RuntimeIdentifier：win-x64 # https://docs.microsoft.com/zh-cn/dotnet/core/rid-catalog

PublishSingleFile：true
IncludeNativeLibrariesForSelfExtract=true；6.0添加了[RequiresAssemblyFiles]和EnableCompressionInSingleFile
PublishTrimmed：true # 删除未使用的成员，只有和自包含一起用才有意义和不报错。即使不开启它也可开启EnableTrimAnalyzer检测裁剪错误
PublishReadyToRun：true # 混合AOT，必须指定RID；提高启动速度，增加体积和编译时间，代码质量不如JIT不过运行后会自动分层编译
PublishReadyToRunComposite：显著增加体积和编译时间，稍微增加R2R效果。只能在自包含中启用。建议如果启用了分层编译就别开
InvariantGlobalization：true # 减少Linux下自包含的体积
DebugType：none # 默认portable，是一种跨平台格式。VS模板默认pdbonly，与full等价，在Win下使用专有格式。embedded嵌入文件内部，但直接用csc时不会报行号
ImplicitUsings：true 自动添加System Generic IO Linq Http Tasks的引用。启用Winform时还会添加Drawing Forms
PublishAot：隐式启用且必须启用PublishTrimmed。只在publish时生效，win下要装VC。一般再加StripSymbols否则大量体积是调试符号

LangVersion：latest/preview # 目标框架是net472时可加上
AllowUnsafeBlocks：true # 启用后才能写unsafe块，不是默认全局unsafe
OutputPath：Bin/ # 最好加上目录分隔符
CheckForOverflowUnderflow：true # 溢出时抛异常
Nullable：enable
NoWarn：NU1602,NU1604
WarningsAsErrors：$(WarningsAsErrors);CS8600;CS8602;CS8603;CS8618 # 几个nullable的视为error
GenerateAssemblyInfo：默认为true。自定义了Properties/AssemblyInfo.cs时可改为false否则会报重复声明，则还要定义RootNamespace和AssemblyName
DefineConstants：相当于-d/-define
AnalysisMode：默认已开启分析器，此项设为Minimum/Recommended/All可启用更多的Lint
ServerGarbageCollection：默认是false表示工作站类型，会GC更频繁以保持小内存占用。ASP会默认用true
SatelliteResourceLanguages：en 当存档多语言资源时，此项只保留指定的

WPF：
OutputType：WinExe # 存在下一条时本条设为exe也可，会自动替换，但不能不设置
UseWPF：true # 还有UseWindowsForms
TargetFramework：net5.0-windows
ApplicationIcon：favicon.ico
ApplicationManifest：app.manifest # 好像会自动使用

<ItemGroup>
  <Content Include="lib\**">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </Content>
  <Resource Include="logo.ico" />

  <PackageReference Include="Newtonsoft.Json" Version="12.*" />
  <ProjectReference Include="..\xxx\xxx.csproj" /> # FX中此项不是传递性的，而Core是，即Prj1引用Prj2，2引用3，则FX中1不能用3
  <Reference Hintpath="xxx.dll" />
  <Reference Include="System.Net.Http" /> # 目标框架是net472时可加上
</ItemGroup>

<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="echo Output written to $(OutDir)" /> # 默认CWD是Temp
</Target>
```

### 默认编译项

* 取消默认的Compile项：EnableDefaultCompileItems false
* 最小的全自定义Compile项：`<Compile Include="**/*.cs" Exclude="**/*.csproj; **/*.sln; bin/**; obj/**" />`，不能只写bin，必须加/**
* 自定义排除内容：`<DefaultItemExcludes>$(DefaultItemExcludes);xxx</DefaultItemExcludes>`
* Core中.settings和.resx仍需手动加入：https://docs.microsoft.com/zh-cn/dotnet/desktop/winforms/migration/#resources-and-settings

## dotnet CLI

* dotnet new list 列出可用的模板
* dotnet add/remove package xxx；dotnet list package --outdated --include-transitive
* dotnet add app/app.csproj reference **/*.csproj；dotnet list/remove reference
* dotnet sln todo.sln add **/*.csproj；dotnet sln remove
* dotnet run -- -flag：--后的是传递给运行的程序，不是传递给dotnet程序的。无法在sln目录下运行，必须加-p csproj的目录
* dotnet watch：任何文件修改后就重新编译一遍，程序退出后有修改又会自动再启动运行；对ASP比较有用。VSC修改后有一直重启的BUG
* 未读：dotnet user-secrets
* --configuration/c Release：默认Debug。.NET8 publish时默认Release
* --runtime/-r rid
* --output/-o dir：指定存放目录，默认是./bin/Debug/net5.0
* SCD独立部署(自包含)带运行时体积大，必须指定rid且一旦指定就默认加了--self-contained
* FDD和FDE依赖框架的部署，前者运行必须用dotnet xxx.dll，3.0默认后者，能生成.exe但实际程序仍是dll；要么不指定rid，要么指定rid且加--self-contained=false
* --roll-forward LatestMajor：使用最新runtime跑老应用。也可指定DOTNET_ROLL_FORWARD环境变量，用于某个非自己开发的程序没更新
* 设置COREHOST_TRACE=1环境变量可详细显示编译过程
* 修改显示语言：DOTNET_CLI_UI_LANGUAGE
* dotnet msbuild /pp | code -：查看完整生成的csproj
* .NET7在所有必须指定RID的地方，如果没有手动指定则会隐式使用当前的。包括R2R 自包含 单文件 AOT

### 全局工具

* 安装：dotnet tool install -g PackageName --add-source ./
* 列出、卸载、升级：dotnet tool list、dotnet tool uninstall、dotnet tool update
* 自己创建：https://docs.microsoft.com/zh-cn/dotnet/core/tools/global-tools-how-to-create
* 收集：https://www.nuget.org/packages?packagetype=dotnettool
* dotnet-script

## Docker

* Linux的latest是debian10，Win是nanoserver-1909

```
dotnet new webapi --no-https

FROM mcr.microsoft.com/dotnet/sdk:5.0-buster-slim AS build
WORKDIR /src
COPY *.csproj . # 或用 *.sln，但好像csproj的文件夹没法通配复制啊
RUN dotnet restore
COPY . .
RUN dotnet publish -c release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:5.0-alpine # 默认已放行80和443
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "myapp.dll"]

docker build -t myapp
docker run -it --rm -p 3000:80 --name myappcontainer myapp
```

## MSBuild

### 命令行参数

* `/property:xxx=yyy;aaa=bbb`或/p：重写属性值
* `/target:xxx;yyy`或/t：生成这些目标
* `/maxcpucount[:n]`或/m：指定要使用的cpu个数并行生成，不加为1，加了但不指定n就是最大
* `/consoleloggerparameters:ShowTimestamp`或/clp：每条命令前显示时间；还有一些其它的选项
* `/fileLogger[n]`：输出到msbuild[n].log文件中
* `/warnaserror[:code;code2]`：将(某些)警告视为错误
* `/restore`或/r：先执行名为Restore的目标
* `/verbosity`或/v：q[uiet], m[inimal], n[ormal], d[etailed], and diag[nostic]，一般指定-m

### 元素

* Task元素是最小的元素，Target元素是Task元素的有序组合
* 元素名和Attribute都区分大小写
* 最顶层是Project元素，可以声明InitialTargets和DefaultTargets，前者即使手动指定其它目标也会执行，后者就是不手动指定时的默认值了
* Target元素可以有DependsOnTargets，多个依赖用分号分隔
* 使用Target元素的Inputs和Outputs属性可以增量生成：https://docs.microsoft.com/zh-cn/visualstudio/msbuild/how-to-build-incrementally

#### 可能的常用元素

* 处理文件：Copy、Delete、Move、Touch、Exec、DownloadFile
* 处理文件夹：MakeDir、RemoveDir、Unzip、ZipDirectory
* 其它：CombinePath、ConvertToAbsolutePath、Message、SignFile、Csc、CallTarget

### 属性

* 所有的属性都在`<PropertyGroup>`中，后声明的会覆盖前面的
* 使用`$(PropertyName)`使用属性，属性名不区分大小写
* 预定义属性：https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-reserved-and-well-known-properties
* 条件属性：`<Configuration Condition=" '$(Configuration)' == ''">Debug</Configuration>`：当Configuration未指定时，会等于""；还可在MakeDir中用!Exist判断文件夹不存在
* 字符串里如果含有[特殊字符](https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-special-characters)，使用反斜杠是无法转义的，要写%+十六进制
* 获取注册表值：`$(registry:Hive\MyKey\MySubKey@Value)`，去掉@Value可获取子键默认值
* $()中也可以使用类型PS与.NET交互的语法，字符串要用反引号
* OutDir与TargetDir的区别是后者是相对路径；会自动根据rid进行变化但不会根据publish进行变化

```
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
```

### 项

* 处于`<ItemGroup>`下，用作系统的输入，通常是文件
* 获取项的值用`@()`
* 可以有多个相同名字的项，其中Exclude仅对同一句中的Include生效
* 项还可能有元数据，如`<FileName>`；大概是项的属性；用%(项.metaname)；预定义的参见：https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-well-known-item-metadata
* EmbeddedResource：内嵌到程序集内，运行时不更改
* Content：程序集中只包含一些元数据，VS里选择“包含在项目中”后就是此项，默认不会复制到生成目录。另外文件夹是Folder
* None：允许编译时文件不存在
* Resource：仅WPF，内嵌到程序集内，VS里将文件包含时默认变为此项。使用时用new Uri("/xxx",UriKind.Relative)，绝对路径是pack://application:,,,/xxx这样的，若文件路径中有井号百分号则需转义。Page是特殊的资源，会编译

### Csc示例

```
<ItemGroup>
    <CSFile Include="**/*.cs" / Exclude="./bin;./obj" />
</ItemGroup>

<Target Name="Compile" DependsOnTarget="Resources" >
    <Csc Sources="@(CSFile)"
            TargetType="library"
            Resources="@(CompiledResources)"
            EmitDebugInformation="$(includeDebugInformation)"
            References="@(Reference)"
            DebugType="$(debuggingType)" >
            <Output TaskParameter="OutputAssembly"
                ItemName="FinalAssemblyName" />
    </Csc>
    <Message Text="The output file is @(FinalAssemblyName)"/>
</Target>
```

## nuget及打包

* 在PropertyGroup中添加PackageId、PackageVersion、Title、Authors等属性，替代.nuspec，dotnet pack打包
* 进入.nupkg，dotnet nuget push name.nupkg -k API_Key -s https://api.nuget.org/v3/index.json
* 上传后不能删，即使没有任何包依赖也不行
* 403错误可能是和别人的包重名了
* dotnet nuget locals --clear all：清除所有本地缓存目录的文件，包括%LocalAppData%\NuGet\v3-cache和~/.nuget等
* 暂时还没有dotnet update，Feature Request在：https://github.com/NuGet/Home/issues/4103
* nuget.config能指定第三方源，包括本地文件夹
* Version指定的是最低兼容版本，用*可指定最高版本
* 国内镜像，VS中使用时要删掉原来的，不知是不是BUG：https://nuget.cdn.azure.cn/v3/index.json
* `error NU1100: Unable to resolve xxx for 'net6.0'`：删除%AppData%\NuGet\NuGet.Config

## [manifest](https://docs.microsoft.com/windows/win32/sbscs/application-manifests)

* 如果程序内的资源没有，也支持放在同一位置下
* heapType:SegmentHeap 需要2004
* longPathAware 需要1607
* activeCodePage:UTF-8 需要1903
* requestedExecutionLevel
  * asInvoker：调用者是什么权限就是什么权限，一般Explorer运行就是标准权限
  * requireAdministrator：在Explorer中双击会弹UAC
  * highestAvailable：在管理员账户里弹UAC，在标准账户里以受限权限运行
  * UWP只能以受限权限运行，关闭UAC后会都闪退
* highResolutionScrollingAware：还有一个ultraHigh的，用于触摸板

## Upgrade Assistant

* https://devblogs.microsoft.com/dotnet/introducing-the-net-upgrade-assistant-preview/
* dotnet tool install -g try-convert upgrade-assistant
* upgrade-assistant <MySolution.sln>
* 老的方式：https://natemcmaster.com/blog/2017/03/09/vs2015-to-vs2017-upgrade/ https://github.com/hvanbakel/CsprojToVs2017
* Microsoft.Windows.Compatibility：对于FX项目，添加此包可直接迁移到Core，它包含了那些被移除的API
* https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer VS的插件，分析从FX迁移到Core的兼容性问题
* https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.upgradeassistant 官方VS插件

## infersharp

* https://github.com/microsoft/infersharp
* 能检测代码中的空引用、资源泄露、线程安全
* 可用Actions和Docker映像，但好像无法在Win下运行

## .NET Framework

* `Your project does not reference ".NETFramework,Version=v4.7.2" framework. Add a reference to ".NETFramework,Version=v4.7.2" in the "TargetFrameworks" property of your project file and then re-run NuGet restore.`：不要混用Core和FX的生成，删除bin和obj即可
* 获得已安装的版本：`reg query "HKLM\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full" /v version`
* LTSB2015启用更新后所带版本为4.6.2

### NGen

* C:\Windows\Microsoft.NET\Framework64\v4.0.30319\ngen.exe
* ngen install/display xxx.exe
* ngen update
* 需要管理员权限
* 本机映像生成在C:\Windows\assembly中

## bflat

* https://github.com/MichalStrehovsky/bflat 删去lib里的Linux和arm64，不要删pdb
* bflat build src.cs lib.cs -Ot或-Os --no-debug-info --no-globalization --no-reflection --no-stacktrace-data --no-exception-messages
* 支持C#10的“顶级语句”
* 如果源文件没有Main，自动视为库生成dll，导出函数要指定`[UnmanagedCallersOnly(EntryPoint="xxx")]`，异常必须捕获，之后可当作C函数调用。若用在另一个bflat的cs中，要用-i:库名 --ldflags:库名.lib
* 仅x64，不支持csproj但可以不加源文件名表示CWD所有cs文件

## csc

* /o开启优化
* 默认不生成调试内容，/debug开启。默认生成外部pdb文件，有个内嵌版但实测无法显示行号
* /r:System.Net.Http.dll
* /platform
  * 默认anycpu，产生32位程序，在32位下运行是32位的，64位下运行是64位的
  * FX的csproj默认为Prefer32Bit=true，对应csc的anycpu32bitpreferred，导致64位下运行还是32位的
  * dll不支持anycpu32bitpreferred，不受Prefer32Bit影响，anycpu的dll能被anycpu和x86和x64的exe使用
  * Core完全不支持anycpu，相当于仅有x86和x64，64位SDK默认x64

## 反编译

* ILSpy：SharpDevelop开发者出的，还有VS扩展
* dnSpy：用了VS Shell，更美观，好像还能调试，但是Archive了。现在有dnSpyEx的Fork
* dotPeek：JB的，免费
* https://www.telerik.com/products/decompiler.aspx：Telerik的，开源免费但有一段时间不更新了，只有在线安装包

## 混淆和反混淆

* https://github.com/XenocodeRCE/neo-ConfuserEx
* https://github.com/de4dot/de4dot
* https://github.com/SychicBoy/NetReactorSlayer
* https://github.com/obfuscar/obfuscar

## VS社区版协议

* 个人：任意使用
* 开发 OSI批准的开源协议的程序、教育教学、Win驱动 允许任意数量的人使用。若不是开发这些，且也不是企业，最多5人使用
* 企业，不允许使用：指超过250台电脑或250位用户或年收入超过100万美元

## JetBrains协议

* IDEA社区版和PyCharm社区版可以商业使用，只是不能将社区版IDE商业化
* 可以使用个人授权许可证在商业开发上，只是个人的只能本人用，不能分享，也不能向公司报销
* 永久回退许可证：订阅一年以上，若停止订阅，仍可以一直使用订阅到期日往回推一年前时的正式版本
* 开源项目免费许可证：需正在积极开发、不提供付费版本和付费支持等、未获得商业公司的资助
* EAP：无限制，无需订阅，可以商用。但每个build只有Release后的30天内有效。CLion和Rider都有

## TODO

* https://zhuanlan.zhihu.com/p/35979897
* https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-concepts
* PackageReference的PrivateAssets="All"
