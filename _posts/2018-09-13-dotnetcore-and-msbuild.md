---
title: .NET Core和MSBuild
category: dotnet
---

## 安装

* Windows：https://dot.net
* docker映像：https://hub.docker.com/_/microsoft-dotnet
* Linux(不支持x86)：https://docs.microsoft.com/zh-cn/dotnet/core/install/linux-debian；
* dotnet-install.sh自动安装脚本，支持无root权限安装，但只是用于CI环境临时使用。且需要手动添加Path：`export PATH=~/.dotnet:$PATH`、`$env:Path="$env:LocalAppdata\Microsoft\dotnet;"+$env:Path`
* 源代码：https://source.dot.net/，部分API只会显示Linux的。FX的源码：https://referencesource.microsoft.com

## [CSProj](https://docs.microsoft.com/zh-cn/dotnet/core/tools/csproj)

* CoreRT必须要装VC
* dotnet msbuild /pp | code -：查看完整生成的csproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

<PropertyGroup>：
StartupObject：Namespace.Class # 存在多个Main时指定入口
OutputType：Exe # 默认Library
TargetFramework：net5.0 # 指定多个加s和分号
RuntimeIdentifier：win-x64 # https://docs.microsoft.com/zh-cn/dotnet/core/rid-catalog

PublishSingleFile：true # 自包含最好加上IncludeNativeLibrariesForSelfExtract=true；6.0添加了[RequiresAssemblyFiles]和EnableCompressionInSingleFile
PublishTrimmed：true # 删除未使用的成员，只有和自包含一起用才有意义和不报错，小心反射失败除非确定目标能静态检测到；现在默认TrimMode为link且开启了分析警告
PublishReadyToRun：true # 混合AOT，必须指定RID，可与Trimmed一起用；提高启动速度，减少JIT数量，但代码质量不如JIT，不过会自动分层编译
InvariantGlobalization：true # 减少Linux下自包含的体积
DebugType：none # 默认portable，是一种跨平台格式。VS模板默认pdbonly，与full等价，在Win下使用专有格式。embedded嵌入文件内部，但直接用csc时不会报行号
Prefer32Bit：默认false，但VS模板默认true

LangVersion：latest/preview
AllowUnsafeBlocks：true # 启用后才能写unsafe块，不是默认全局unsafe
AssemblyName：MSBuildSample # 还有个RootNamespace
OutputPath：Bin/ # 最好加上目录分隔符
CheckForOverflowUnderflow：true # 溢出时抛异常
Nullable：enable
NoWarn：NU1602,NU1604
WarningsAsErrors：$(WarningsAsErrors);CS8600;CS8602;CS8603;CS8618 # 几个nullable的视为error
GenerateAssemblyInfo：默认为true，自定义了Properties/AssemblyInfo.cs时可改为false否则会报重复声明
DefineConstants：未看
AnalysisMode：AllEnabledByDefault启用更多的Lint，但可能太多了，比如public filed都会有警告

WPF：
OutputType：WinExe # 存在下一条时设置为exe也可，会自动替换，但不能不设置；有选项关闭自动替换
UseWPF：true # 还有UseWindowsForms
TargetFramework：net5.0-windows
ApplicationIcon：favicon.ico
ApplicationManifest：app.manifest

<ItemGroup>
  <Content Include="lib\**">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </Content>
  <Resource Include="logo.ico" />

  <PackageReference Include="Newtonsoft.Json" Version="12.*" />
  <ProjectReference Include="..\xxx\xxx.csproj" />
  <Reference Hintpath="xxx.dll" />
</ItemGroup>

<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="echo Output written to $(OutDir)" /> # 默认CWD是Temp
</Target>
```

### 默认编译项

EnableDefaultCompileItems属性设为false后可取消默认的Compile项。
最小的全自定义Compile项：`<Compile Include="**/*.cs" Exclude="**/*.csproj; **/*.sln; bin/**; obj/**" />`，不能只写bin，必须加/**
所有默认排除项可用$(DefaultItemExcludes)引用。

## dotnet CLI

* dotnet add/remove package xxx；dotnet list package --outdated
* dotnet add app/app.csproj reference **/*.csproj；dotnet list/remove reference
* dotnet sln todo.sln add **/*.csproj；dotnet sln remove
* dotnet run -- -flag：--后的是传递给运行的程序，不是传递给dotnet程序的。无法在sln目录下运行，必须加-p csproj的目录
* dotnet watch：任何文件修改后就重新编译一遍，程序退出后有修改又会自动再启动运行；对ASP比较有用。VSC修改后有一直重启的BUG
* 未读：dotnet user-secrets
* --configuration/c Release：默认Debug
* --runtime/-r rid
* --output/-o dir：指定存放目录，默认是./bin/Debug/net5.0
* SCD独立部署(自包含)带运行时体积大，必须指定rid且一旦指定就默认加了--self-contained
* FDD和FDE依赖框架的部署，前者运行必须用dotnet xxx.dll，3.0默认后者，能生成.exe但实际程序仍是dll；要么不指定rid，要么指定rid且加--self-contained=false
* --roll-forward LatestMajor：使用最新runtime跑老应用。也可指定DOTNET_ROLL_FORWARD环境变量，用于某个非自己开发的程序没更新
* 设置COREHOST_TRACE=1环境变量可详细显示编译过程

### 全局工具

* 安装：dotnet tool install -g PackageName --add-source ./
* 列出、卸载、升级：dotnet tool list、dotnet tool uninstall、dotnet tool update
* 自己创建：https://docs.microsoft.com/zh-cn/dotnet/core/tools/global-tools-how-to-create
* 收集：https://github.com/natemcmaster/dotnet-tools

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

* `/property:xxx=yyy`或/p：重写属性值
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
* 字符串里如果含有[特殊字符](https://docs.microsoft.com/zh-cn/visualstudio/msbuild/msbuild-special-characters)，使用反斜杠是无法转义的，要写%+16进制
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

## Upgrade Assistant

* https://devblogs.microsoft.com/dotnet/introducing-the-net-upgrade-assistant-preview/
* dotnet tool install -g try-convert upgrade-assistant
* upgrade-assistant <MySolution.sln>
* 老的方式：https://natemcmaster.com/blog/2017/03/09/vs2015-to-vs2017-upgrade/ https://github.com/hvanbakel/CsprojToVs2017
* Microsoft.Windows.Compatibility：对于FX项目，添加此包可直接迁移到Core，它包含了那些被移除的API

## 参考

* https://zhuanlan.zhihu.com/p/35979897

### TODO

```
ItemGroup:
<Folder Include="datafiles\" />
	<None Update="datafiles\datafile.db">
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
         </None>
```
