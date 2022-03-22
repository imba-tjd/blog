---
title: C#库
---

## Newtonsoft.Json

```c#
using
JsonConvert.SerializeObject(obj [,Formatting.Indented]); // 支持基元类型、IEnumerable、IDictionary
JsonConvert.DeserializeObject<OBJ>(str);

using JsonWriter writer = new JsonTextWriter(sw);
var serializer = new JsonSerializer() {NullValueHandling = NullValueHandling.Ignore};
serializer.Serialize(writer, obj);

JObject.Parse(str); 之后当作dict用，其实一般用SerializeObject<dynamic>()
JObject.FromObject(匿名对象)
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
