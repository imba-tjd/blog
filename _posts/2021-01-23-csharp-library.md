---
title: C#库
---

## Newtonsoft.Json

* 部分功能依赖Microsoft.CSharp命名空间

```c#
using
JsonConvert.SerializeObject(obj [,Formatting.Indented]); 支持基元类型、IEnumerable、IDictionary
JsonConvert.DeserializeObject<OBJ>(str);

using JsonWriter writer = new JsonTextWriter(sw);
var serializer = new JsonSerializer() {NullValueHandling = NullValueHandling.Ignore};
serializer.Serialize(writer, obj);

JObject.Parse(str); 之后当作dict用，还可SelectToken(jsonpath)
JObject.FromObject(匿名对象)
JArray 当作List用，长度用Count

SerializeObject<dynamic>()，之后object可用.xxx或同JObject，数组同JArray
反序列化时json字面量支持单引号
```

## Config.Net

* 支持多种数据源：app.config 命令行 环境变量 ini json
* ini
  * `[Option(Alias = "SectionOne.MyOption")]`将产生Section
  * 支持数组，表示法：Numbers="1 2 3"
  * 值最好不要有分号，否则会与注释混淆。Key不能有等号

```c#
using Config.Net;
public interface IMySettings { // 可选继承INotifyPropertyChanged，使用者订阅PropertyChanged事件
   [DefaultValue("n/a")]
   string AuthClientId { get; }
}

IMySettings settings = new ConfigurationBuilder<IMySettings>()
   .UseIniFile(filePath)
   .Build();
```

## Windows.Networking.Sockets

```c#
var socket = new StreamSocket();
await socket.ConnectAsync(new HostName("localhost"), "8888");
var w = new DataWriter(socket.OutputStream);
w.WriteString(s);
await w.StoreAsync();

var serversocket = new StreamSocketListener();
await serversocket.BindEndpointAsync(new HostName("localhost"), "8888");
serversocket.ConnectionReceived += async (sender, e) => { // 一次连接
   var reader = new DataReader(e.Socket.InputStream);
   while(true) {
         await reader.LoadAsync(4); // 必须读满指定长度才继续
         string s = reader.ReadString(4);
         Console.WriteLine(s);
   }
};
```

## 第三方库

* https://www.hangfire.io/
* https://github.com/icsharpcode/SharpZipLib
* https://masstransit-project.com/
* https://github.com/App-vNext/Polly 弹性及瞬态故障处理库，方便指定重试次数、超时、熔断等
* https://github.com/pythonnet/pythonnet
* https://github.com/dotnet/command-line-api：System.CommandLine；https://dotnetcoretutorials.com/2021/01/16/creating-modern-and-helpful-command-line-utilities-with-system-commandline/
* https://github.com/dotnet/reactive
* https://github.com/natemcmaster/CommandLineUtils
* https://github.com/autofac/Autofac 轻量依赖注入
* https://github.com/SixLabors/ImageSharp 处理图片，跨平台，替代System.Drawing
* https://github.com/microsoft/FASTER 包括一个日志库和一个KV数据库
* https://github.com/bilal-fazlani/commanddotnet
* 混淆器：https://github.com/NotPrab/.NET-Obfuscator
