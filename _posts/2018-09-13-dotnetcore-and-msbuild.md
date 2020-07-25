---
title: ".NET Core和MSBuild"
---

安装
----

* Linux上的运行时：https://docs.microsoft.com/zh-cn/dotnet/core/install/runtime?pivots=os-linux，全都要先做准备工作；Windows上的SDK：https://dotnet.microsoft.com/download；也有自动安装脚本，且支持在没有root权限的情况下安装
* Linux全部不支持x86
* DOTNET_SKIP_FIRST_TIME_EXPERIENCE=1：环境变量，跳过第一次使用的欢迎信息
* docker映像：https://hub.docker.com/_/microsoft-dotnet-core/

[CSProj](https://docs.microsoft.com/zh-cn/dotnet/core/tools/csproj)的PropertyGroup
----------------------------------------------------------------------------------

* 从2015的格式迁移到新格式：https://natemcmaster.com/blog/2017/03/09/vs2015-to-vs2017-upgrade/；https://github.com/hvanbakel/CsprojToVs2017

```xml
<StartupObject>Namespace.Class // 指定入口
//<OutputType>Exe // 生成EXE；3.0前还必需指定RID，现在默认FDE不需要它了；WinExe为窗体程序
<TargetFramework>netcoreapp3.0</TargetFramework> // 分号指定多目标
<RuntimeIdentifier>... // RID
<PublishSingleFile>true // 单文件发布，但比较大，100多M？
<PublishTrimmed>true // 裁剪体积，但就不要用反射了
<PublishReadyToRun>true // AOT模式，必须指定RID，可与Trimmed一起用
<LangVersion>latest // C#版本
<AllowUnsafeBlocks>true
<Configuration>Debug</Configuration>
<AssemblyName>MSBuildSample</AssemblyName>
<OutputPath>Bin/ // 最好加目录分隔符
<OS>Windows_NT
<IsPackable> // 与dotnet pack有关
<CheckForOverflowUnderflow>true //全局溢出时抛异常
<NoWarn>NU1602,NU1604</NoWarn>
<Nullable>enable
<WarningsAsErrors>$(WarningsAsErrors);CS8600;CS8602;CS8603;CS8618</WarningsAsErrors> //把几个nullable的视为error
<ApplicationIcon>favicon.ico
未看：GenerateAssemblyInfo、AssemblyName、RootNamespace、ApplicationManifest、DefineConstants
还存在一个全球化固定模式，可以减少Linux下安装包的体积：github.com/dotnet/corefx/blob/master/Documentation/architecture/globalization-invariant-mode.md

WPF：
<OutputType>WinExe；<UseWPF>true
```

### 默认编译项

```
EnableDefaultCompileItems属性设为false后可取消默认的Compile项。
最小的全自定义Compile项：<Compile Include="**/*.cs" Exclude="**/*.csproj; **/*.sln; bin/**; obj/**" />，不能只写bin，必须加/**
所有默认排除项可用$(DefaultItemExcludes)引用。
```

### [Runtime IDentifier(RID)](https://docs.microsoft.com/zh-cn/dotnet/core/rid-catalog)

* win-x86、win-x64、win10-x64；linux-x64、osx-x64；win-x64-corert

dotnet CLI
----------

无论是全写还是缩写，都使用空格分隔开关和值。

* dotnet run -- -flag：--后的是传递给运行的程序，不是传递给dotnet程序的
* 未读：dotnet watch、dotnet user-secrets

### dotnet build、clean、publish共有

* --configuration/c {Debug|Release}
* --framework/f \<FRAMEWORK\>
* --output \<OUTPUT_DIRECTORY\>/-o：指定存放目录，默认是./bin/Dubug/netcoreapp2.2
* --runtime \<RUNTIME_IDENTIFIER\>/-r
* --no-restore：强制不自动还原

### dotnet publish

* SCD独立部署：dotnet publish -c Release -r win-x64；带运行时，体积大
* FDD和FDE：不自带runtime。区别是前者必须用dotnet xxx.dll才能运行程序，后者自带.exe（当前平台的程序），但实际程序仍是dll。3.0默认后者。
* --self-contained：一同发布运行时。如果指定了RID，就默认加上了这个参数，否则必须用`--self-contained false`手动关闭
* /p:PublishSingleFile=true：打包为单文件
* 不清楚FDE中的dll是否是跨平台的

### dotnet sln、reference、package

* dotnet sln todo.sln add **/*.csproj
* dotnet sln remove
* dotnet add app/app.csproj reference **/*.csproj
* dotnet list reference、dotnet remove reference
* dotnet add package Newtonsoft.Json [--version 1.0.0]、dotnet remove package

### 其他

* dotnet nuget locals --clear all：清除所有本地缓存目录的文件
* 暂时还没有dotnet update和list package，Feature Request在：https://github.com/NuGet/Home/issues/4103，可能的脚本解决办法：https://gist.github.com/JonCanning/a083e80c53eb68fac32fe1bfe8e63c48
* ~~要添加DotNetCliToolReference现在只能手动编辑csproj，Feature Request在：https://github.com/NuGet/Home/issues/4901~~ 此条目被deprecated了

全局工具
--------

* 安装：dotnet tool install -g *PackageName* --add-source ./
* 列出、卸载、升级：dotnet tool list、dotnet tool uninstall、dotnet tool update
* 自己创建/打包：dotnet pack、https://docs.microsoft.com/zh-cn/dotnet/core/tools/global-tools-how-to-create
* 有名的可用的：https://github.com/natemcmaster/dotnet-tools
* dotnet watch

Docker
------

* Linux的latest是debian10，Win是nanoserver-1909

```
dotnet new webapi --no-https

FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine AS build
WORKDIR /src
COPY *.csproj . # 或用 *.sln，但好像csproj的文件夹没法通配复制啊
RUN dotnet restore # 教程如此，说是as distinct layers
COPY . .
RUN dotnet publish -c release -o /app

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-alpine # 默认已放行80和443
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "myapp.dll"]

docker build -t myapp
docker run -it --rm -p 3000:80 --name myappcontainer myapp
```

MSBuild
-------

### 命令行参数

* `/property:xxx=yyy`或/p：重写属性值
* `/target:xxx;yyy`或/t：生成这些目标
* `/maxcpucount[:n]`或/m：指定要使用的cpu个数并行生成，不加为1，加了但不指定n就是最大
* `/consoleloggerparameters:ShowTimestamp`或/clp：每条命令前显示时间；还有一些其它的选项
* `/fileLogger[n]`：输出到msbuild[n].log文件中
* `/warnaserror[:code[;code2]]`：将警告视为错误
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
* 字符串里如果含有[特殊字符](https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-special-characters)，使用反斜杠是无法转义的，要写%+16进制
* 获取注册表值：`$(registry:Hive\MyKey\MySubKey@Value)`，去掉@Value可获取子键默认值
* $()中也可以使用类型PS与.NET交互的语法，字符串要用反引号

```
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
```

### 项

* 处于`<ItemGroup>`下，用作系统的输入，通常是文件
* 获取项的值用`@()`
* 可以有多个相同名字的项，其中Exclude仅对同一句中的Include生效
* 项还可能有元数据，如`<FileName>`；大概是项的属性；用%(项.metaname)；预定义的参见：https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-well-known-item-metadata

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


