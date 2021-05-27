---
title: ASP.NET Core
category: dotnet
tags:
    - C#
    - ASP.NET
---

## 基本概念

### 初始化

```bash
dotnet new webapp [--no-https]
dotnet tool install --global dotnet-dev-certs
dotnet dev-certs https --clean/--trust # Linux下不可用，只会自动生成，无法安装，可以直接用Nginx反代；只会有localhost的证书，改成别的SNI会连接失败
```

### Startup类

* 还有一个IStartupFilter，没看懂有什么用

#### Configure和中间件

```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env) {
    // 异常/错误处理
    if (env.IsDevelopment()) {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    } else {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    // 可手动用appsettings的https_port或services.AddHttpsRedirection(op=>{op.HttpsPort});指定要重定向到的端口；3.0后不手动指定仍会自动重定向到第一个可用的https端口，如果没有可用的就不重定向。此选项并不决定kestrel监听哪个https端口
    app.UseHttpsRedirection(); // 用的307

    // app.UseDefaultFiles(); //测试下来只有Razor路由没有时才生效，增加wwwroot/Default.html和index.html的路由，但路径仍要对应，不会使用Pages下的html

    // Return static files and end the pipeline.
    // 使用时用波形符指向web根目录（默认为wwwroot）
    app.UseStaticFiles(); // 无参的就是设置web根目录，可多次调用有参重载设置别的
    // 一般要加OnPrepareResponse = ctx => ctx.Context.Response.Headers.Append("Cache-Control", ...)
    // 另外还有DirectoryBrowser目录浏览的中间件和FileServer中间件

    app.UseCookiePolicy(); // 添加符合欧洲的GDPR条例

    app.UseRouting();
    app.UseCors();

    app.UseResponseCompression(); // 不应压缩图片，不知道它是怎么处理的
    app.UseResponseCaching(); // 文档中还进行了其它调整，不知道只加这个是否有效

    // Authenticate before the user accesses secure resources.
    app.UseAuthentication();
    app.UseAuthorization(); // 授权，默认模板只有这一个，如果不用可以删掉

    app.UseEndpoints(endpoints => {
        endpoints.MapRazorPages();
        endpoints.MapFallbackToPage("/404");

        // endpoints.MapControllerRoute(...) // MVC
        // endpoints.MapControllers() // WebApi
        // endpoints.MapGet() // 一些自定义路由
    });
}
```

#### 自定义管道和中间件

```c#
app.Use(async (context, next) => {
    // Do work that doesn't write to the Response.
    await next.Invoke(); // 不调用它可使请求短路；向客户端发送响应后就不要再用这个方法了
    // Do logging or other work that doesn't write to the Response.
});

// 为特定的Url创建管道分支；签名和Configure一样
app.Map("/map1", HandleMapTest1); // 支持嵌套和同时匹配多个段

app.Run(async (context) => { // 管道的终点，没有next参数
    await context.Response.WriteAsync("Hello World!");
}

// 标准化的中间件，用UseMiddleware<T>()调用；可以把它添加进IApplicationBuilder的扩展方法，return前者，命名约定为UseXXX，就可以像普通的那样用了
public class XXXMiddleware {
    private readonly RequestDelegate _next;
    public RequestCultureMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context) {
        // Do sth related to context

        // Call the next delegate/middleware in the pipeline
        await _next(context);
    }
}
```

#### ConfigureServices和依赖注入

```c#
// 使用方法：using xxx；构造函数接受IT的参数；Razor页面用@inject；在有IApplicationBuilder的时候是app.ApplicationServices.GetService<T>()；不知道需不需要判断为null？
// DI的优点：没有强依赖，利于单元测试、不需要了解具体的服务类、不需要管理服务类的生命周期。
public void ConfigureServices(IServiceCollection services) {
    services.AddRazorPages();
    services.AddControllersWithView()//MVC;UseMvc()和UseRouter() Deprecated了
// 即时编译：在上面两者**紧接着**用.AddRazorRuntimeCompilation()，要先添加Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation包；但它只适用于View；另一种方式是用dotnet watch run跑
    services.AddController()[.AddXmlSerializerFormatters()]; // WebAPI

    services.AddResponseCompression();
    services.AddResponseCaching();

    services.AddTransient/AddScoped/AddSingleton<IT,T>(); // 自定义的类，意思是每次请求IT的时候返回T的实例
// Transient是每次请求都会新实例化，Scoped是在一次请求/连接中多次实例化都是同一个
// 另有TryAddXXX方法，用于某个接口未注册时就注册，已注册了就什么都不做
}
```

### Configuration配置

```c#
// 默认命名：appsettings.json以及appsettings.Development.json
{
    "str_config": "value1_from_json",
    "simple_obj": { "value1": "subvalue1_from_json" }
}

// 如果是Razor页面，只用@using加上@inject IConfiguration config就好，类似于ctor
using Microsoft.Extension.Configuration
public Startup(IConfiguration config) => Configuration = config;
public IConfiguration Configuration { get; }

Configure(..., IConfiguration config) {
    var s1=config["str_config"];

    //POCO对象配置读取方式1
    var s2=config["simple_obj:value1"]; // 是array可跟冒号索引

    //POCO对象配置读取方式2
    var simple_object_config = new simple_object(); //Bind要先new，Get<T>不用
    config.GetSection("simple_obj").Bind(simple_object_config);
    var s3="POCO_config:" + simple_object_config.value1;
}

//simple_object类；也支持绑定整个对象图（属性是自定义类），文档用的是xml
public class simple_object {
    public string value1 { get; set; }
}
// 支持绑定数组，但如果中间有null，反序列化后会跳过那些null

config.Sources.Clear();
config.AddInMemoryCollection(dict);
config.AddJsonFile("json_array.json", optional: false, reloadOnChange: false);
config.AddXmlFile();、AddIniFile()
config.AddCommandLine(args); // 要在后面调用才会覆盖前面的；key1=value1或者--key2=value2或者--key3 value3；使用时key后必须直接跟等号，否则用的是空格的处理方式，也不要混用

config.GetValue<int>("Key", defaultVal); // 直接用索引器只会返回字符串
config.GetSection("KeyPath"); // 返回子节；永远不会返回null，如果找不到会返回{}
config.GetChildren(); // 返回IEnumerable<IConfigurationSection>
config.GetSection("...").Exists();
```

#### appsettings.json

* key不区分大小写，不同配置提供相同的key，后者会覆盖前者
* 修改后会立即生效（前提是支持动态加载，像绑端口就不行）
* 可以有注释和末尾的逗号
* 可以在dotnet run的时候用--key=value传进去

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information", // Development下为Debug
      "Microsoft": "Warning", // Key为命名空间
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.AspNetCore.Hosting.Diagnostics": "Information" // 收到请求时显示url和方法以及花费的时间
    }
  },
  "AllowedHosts": "*"
}
```

```c#
// 自定义选项：
public class MyOptions { public int Option {get; set;} = 1 }
// 注册；如果不在根中，要用GetSection()；参数也可为myOptions=>{}来手动设置值而不读取配置
services.Configure<MyOptions>(Configuration);
// 在类中使用：
using Microsoft.Extensions.Options;
_options = IOptionsMonitor/IOptionsSnapshot<MyOptions> optionsAccessor.CurrentValue/Value/Get("Key") // 用构造函数，此处略写
```

#### Properties/launchSettings.json

* 此文件仅用于本地开发，即使手动复制到publish里也没有用
* dotnet run会使用profiles中第一个"commandName"为"Project"的条目，代表使用Kestrel；VS里运行才使用那个IIS的配置
* Linux下绑443等低端口会报错

```json
"test": {
    "commandName": "Project",
    "launchBrowser": true, // 只对VS里启动有效，dotnet run是无效的
    "launchUrl": "api/companies", // 同上，启动时要打开的路径
    "applicationUrl": "https://localhost:5001;http://*:5000"//多域名用分号
    "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
    },
    "hotReloadProfile": "aspnetcore"
}
```

#### 手动读取配置

```c#
using Microsoft.Extensions.Configuration;
var config = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .AddEnvironmentVariables()
    .Build();
```

### 多环境（environment）

* 默认会读取ASPNETCORE_ENVIRONMENT环境变量，默认有Development、Staging和Production；如果不设就是Production
* 在代码中，IHostingEnvironment env，env.IsDevelopment()
* 也可以自定义，用env.IsEnvironment("your_value")
* 不同环境下会自动使用不同的appsettings.json，如appsettings.Development.json。但它只是会**覆盖**普通的，即如果有条目没覆盖，普通的仍有效
* 可以手动创建不同的Startup，但要用“接受程序集名称的UseStartup重载”
* ConfigureXX和ConfigureXXServices也有不同环境版本，且不用特殊调整
* Page中可以用environment标记控制，include/exclude属性指定环境名，可以用来添加不压缩的css

### 日志

```c#
// 自定义太复杂了，这里仅记录使用的方法；Application Insides可以图形化查看日志
services.AddLogging(); // 然而这几个不做也能获得依赖注入和控制台日志
Configure(ILoggerFactory loggerFac){ // 应该有选项能记录到文件
loggerFac.AddConsole(); loggerFac.AddDebug(); } // 不清楚Debug是什么

using Microsoft.Extensions.Logging;
public class TodoController : ControllerBase {
    private readonly ILogger _logger;
    public TodoController(ILogger<TodoController> logger) => _logger=logger;

    [HttpGet("{id}", Name = "GetTodo")]
    public ActionResult<TodoItem> GetById(string id) {
        _logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id); // 还有LogCritical等方法
        // Item lookup code removed.
        if (item == null) {
            _logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);
            return NotFound();
        }
        return item;
    }
}
// 未看：https://www.youtube.com/watch?v=oXNslgIXIbQ
```

## 主机和Main

* 现在应使用Host静态类的方法，不要用WebHost类
* Run()实际上用了RunAsync().GetAwaiter().GetResult();所以它跟直接await RunAsync()是差不多的。会阻止调用线程，直到关闭主机
* RunConsoleAsync()会启用控制台支持？
* Start()是同步运行的，StartAsync()可用于延迟启动，不阻塞；直接使用它们会不响应Ctrl+C，导致直接走到退出的地方然后卡住，要用using包裹host。一般用于非Web的程序使用host，不是单独用的
* 如果不用UseStartup，也可以手动webBuilder.UseConfiguration().ConfigureServices().Configure()
* 配置终结点：默认只会监听localhost的5000和5001，开发者模式下会用launchSettings，然而发布后不会。可在appsettings或命令行中用`"urls": "http://localhost;https://localhost"`或在代码中用UseUrls或UseSetting。在Docker中要用0.0.0.0或[::]或*。端口也可以用*

```c#
static void Main(string[] args) => CreateHostBuilder(args).Build().Run();

static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)[.ConfigureAppConfiguration]
        .ConfigureWebHostDefaults(webBuilder => { // 以下也可以链式调用
            // webBuilder.CaptureStartupErrors(true); // false表示启动时出错就自动退出，true表示捕获异常并尝试启动；默认为flase，除非在IIS“后方”时默认为true
            webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
            webBuilder.UseStartup<Startup>(); // 未知顺序是否有影响
        });
```

### [Kestrel](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/servers/kestrel)

* 配置HTTPS使用证书：https://devblogs.microsoft.com/aspnet/configuring-https-in-asp-net-core-across-different-platforms/

```c#
//CreateDefaultBuilder内部调用了UseKestrel；没有用那个才用webBuilder.UseKestrel
ConfigureWebHostDefaults(webBuilder => {
    webBuilder.ConfigureKestrel(serverOptions => {
        serverOptions.Limits.MaxConcurrentConnections = 100;
        serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
        serverOptions.Limits.MaxRequestBodySize = 10 * 1024;
        serverOptions.Limits.MinRequestBodyDataRate = new MinDataRate(
            bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
        serverOptions.Limits.MinResponseDataRate = 同上
        serverOptions.Listen(IPAddress.Loopback, 5000); // url前缀和端口
        serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions => {
            listenOptions.UseHttps("testCert.pfx", "testPassword");
            // listenOptions.Protocols 现在已经默认启用了http2
        });
        serverOptions.AllowSynchronousIO = false; // 默认是true
    })
    .UseStartup<Startup>();
});

也可以用appsettings.json：

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    },
    "DisableStringReuse": true
  }
}
ConfigureServices(IServiceCollection services) {
    services.Configure<KestrelServerOptions>(
        Configuration.GetSection("Kestrel")); // 见上面的配置Configuration
}
```

## 会话和应用状态

```c#
// SignalR 应用不应使用会话状态来存储信息

services.AddDistributedMemoryCache(); // 不加也行？也可使用数据库真正的分布式后备缓存，此时使用Session前必须要用LoadAsync，否则同步获取会造成性能损失；可注册DistributedSession，则不调用LoadAsync时抛异常
services.AddSession(options => {
    options.Cookie.Name = ".AdventureWorks.Session";
    options.IdleTimeout = TimeSpan.FromSeconds(10); // 默认20分钟
    options.Cookie.HttpOnly = true; // 默认true
    options.Cookie.IsEssential = true; // 默认false
});
// 必须在CookiePolicy之后，终结点之前之前添加本中间件
app.UseSession();

using Microsoft.AspNetCore.Http; // 提供了session的扩展方法
HttpContext.Session.Set/GetString/Int32；还可自己写Get<T>的扩展方法，反序列化

[TempData] // 属于mvc命名空间，在其它地方做同样的声明能获得相同的数据；默认用cookie存，只适合小数据（小于500字节），当用session时没有性能损失
public string Message { get; set; }
// 在Page中，使用@TempData.Peek()获取数据而不删除；使用索引器访问，则在会话结束后内容会被删除，除非又使用了Keep()
```

### Session的问题

* 占服务器的内存、会把服务器变得有状态。当有集群的时候另一台服务器就没有对应的数据，就无法横向扩展，负载均衡也无法直接用轮转法
* 一般需要在Cookie里存SessionId，恶意客户端可能一直发空的Cookie
* 对于多页面只在最后一次提交数据，应该使用localstorage，因为每次访问都会携带所有的cookie，而服务端什么也不会处理；但它的安全性比cookie还差，连httponly都没有，所有JS代码都能读取；或者可以只用前端的方法，只是把多页数据隐藏了，点下一步不发数据

## 其它类

### HttpContext

* HttpContext.Connection.RemoteIpAddress

## Razor和MVC和WebApi

### Razor

* cshtml，必须以`@page`开头
* `@using`引用命名空间，`@inject`使用服务，后面类似于构造函数的参数
* HTML代码中`@表达式`可以使用变量的值，不用加分号。可以是属性，可以调用索引器，可用函数，可用await。其它的一些问题可以加小括号解决
* 如果表达式是字符串，里面的内容会经过HTML编码，显示出来的就是字符串原来的样子；如果想把字符串当作HTML，用HtmlHelper.Raw；非IHtmlContent的表达式会自动ToString
* 两个@会转义一个，email中的会自动处理；@星号 星号@是最优先注释
* 用`@{}`、`@xxx`开头，中间可以写C#代码，可以声明变量和函数，可以调用函数，可以写C#的注释。关键是里面也可以写HTML和普通的@：函数可以返回void，函数体只有HTML（Core3）；如果编译器无法分辨语言而报错或者不想有空格，可用text标记把HTML括起来，或者用`@:`表示该行后面都是HTML
* 支持的以@开头的：if（else和else if就不用@了）、switch、for、foreach、while、dowhile、using、try,catch,finally、lock
* @functions没看懂有什么用。好像是不用就只能写本地函数，用了能用属性和public以及写OnGet，也可用@Functions.xxx调用；不用PM时可用
* model：提供Model属性用于访问传递到视图的模型
* View Component：属于高级用法，PartialView不能添加业务逻辑，Controller无法到处复用
* @helper在3中无法使用了

### Razor Page

* 用的MVVM思想
* https://www.learnrazorpages.com
* 看起来和JSP差不多，都是在HTML里写非前端代码，前后端不分离；而且MVC也一样：https://www.zhihu.com/question/328713931
* ViewData是个字典，基本上和Model用处差不多，只是是弱类型的；在分布页面的时候需要用到，比如默认在_Layout中获取了`@ViewData['Title']`设置为title
* 直接访问域名的根会使用**Pages**文件夹（不是Page）下的Index；直接访问其它不带后缀的路径，如果存在文件会用文件，否则会用对应文件夹下的Index；如果还不存在，中间件会往下走，如果还没有，就返回空白页面

```c#
.cshtml代码，Page：
@page "..."//后面可跟路由规则。/或~/开头是以Page开始，否则是相对路径；{id}和{参数名:int}可添加参数，后者是约束，还可以是alpha:minlength(4)；{id?}中的问号表示参数可选，能把?id=xxx变成路径；View中可用RouteData.Values["id"];
@model IndexModel // 可以是IEnumerable<T>，但不知道怎么用；一个viewmodel可以对应多个view

.cs代码，PageModel：
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
namespace MyWebapp.Pages { // 子文件夹用别的命名空间应该也可以
// 大部分内容都要是public的，依赖注入放在构造函数里
public class IndexModel : PageModel { // 继承了HttpContext属性
    [BindProperty(SupportGet=true)] public int Id {get;set;} // View中可以用@Model.Id获取或赋值，但只有想修改时才添加BindProperty，默认只在Post时有效；对于Get添加后可以写在OnGet参数中，能直接赋进去；但它在Mvc命名空间下
    public void OnGet(int id){ } --or-- public async Task OnGetAsync(){ }
    public async Task<PageResult> OnPostAsync() { // 可以有Model类型的参数
        if (!ModelState.IsValid)
            return Page();
        ...
        return RedirectToPage("..."); // 什么都不加就是首页
    }
}}

// 改变路由，具体见：https://docs.microsoft.com/zh-cn/aspnet/core/razor-pages/razor-pages-conventions
.AddRazorPagesOptions(options => { // 不是RazorOptions
options.Conventions.AddPageRoute("/extras/products", "product");});
//xxx.com/product映射到extras/products，注意顺序；第二个参数可以是"{*url}"或""
// options.RootDirectory可更改默认的Pages文件夹
```

### 结构

* js，css，lib，favicon放在wwwroot下作为静态文件，可以用IWebHostEnvironment.WebRootPath获取
* 内容根：Services、Models、Data、Repositories、Web根(wwwroot)；内容根和Web根在构建主机时可以修改

### 布局

* 使用时路径可以写相对路径或绝对路径，如果是相对路径，寻找顺序为本文件夹、递归父文件夹、/Shared、/Page/Shared（至少分部视图是这样）；所以这些文件可以在子文件夹里创建，用来覆盖父文件夹的设置
* Pages/_ViewImports.cshtml中添加@using和@inject语句，且这个using也具有“自动路由”，后缀部分按它和目标之间的文件夹以点分隔，模板生成的前缀是项目名.Pages
* Pages/_ViewStart.cshtml中添加每个Page渲染前运行的语句（分部视图除外），默认只有`@{Layout = "_Layout";}`，在具体Page中可手动设为null或其它的
* Pages/Shared/_Layout.cshtml中添加大部分的模板，必须使用`@RenderBody()`渲染具体的部分；但又有个IgnoreBody方法，看不懂
* 区域：在具体Page中写`@section Scripts {...}`，在Layout中写`{@RenderSection("Scripts", required: false)}`，其中Scripts是标识符，required如果不为false，效果是找不到对应区域时报错
* 分部视图：约定命名为Shared/_XXXPartitial.cshtml，使用@{await HTML.RenderPartialAsync}渲染，此方法无返回值，性能更好

### Tag Helper

* @add/removeTagHelper：用于简化表单和验证，如asp-for
* https://github.com/52ABP/Documents/blob/V1.0.0/src/mvc/35.Tag-helpers-in-asp.net-core.md、https://github.com/52ABP/Documents/blob/V1.0.0/src/mvc/36.Why-use-tag-helpers.md
* asp-page-handler：可以使用OnPostDeleteAsync这样的函数
* asp-page/asp-controller/asp-action/asp-route-xxx：MVC的
* asp-href-include
* 好像是替代htmlhelper的

#### HTML Helper

* @Html.DisplayNameFor/DisplayFor/DisplayForModel：HTML Helper

### 路由

* https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/routing
* https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/routing
* 不区分大小写
* 可以是某个cshtml文件，也可以是那个文件夹下的Index.cshtml
* action的Async后缀现在无需在url中添加

### MVC

* 默认路由为{controller=Home}/{action=Index}/{id?}，代表默认找HomeController的Index方法，Index方法有一个名字叫做id的参数，但问号代表可以不传，则实际会传默认值
* 建立Controllers文件夹，里面放名字以Controller结尾的类，继承Controller，都放xxx.Controllers命名空间下。文件夹的路径或者是否在子文件夹里不重要。
* 类里的方法就是Action，方法的返回类型为Task IActionResult，实际可返回View(…)或RedirectToAction(nameof(Index))和Json等
* View需要和Action的前缀对应，要不就在View()中传参数指定返回的是哪个；控制器和View的数据用ViewBag交互，无intellisense
* 也可以用ViewModel，返回View()的时候传对象进去，View中用和Razor Page一样的用法
* 方法的参数默认是用QueryString查询字符串?key=val传递的，但可在路由规则中写{id?}变成路径；注意名字必须对应
* 自定义路由，使用[Route("xxx/[controller]")]，其中中括号代表类会以Controller结尾，实际url不需要加这部分，而xxx好像就直接跟在根目录后了；函数也可用这个特性，则是子路由，在控制器下，不会在根目录下
* WinForm采用事件响应机制，无可避免地难以做到View层和Controller层分离，也就无法真正实现MVC架构

### WebApi

* https://docs.microsoft.com/zh-cn/aspnet/core/web-api
* 命名空间使用MVC
* 类继承ControllerBase，必须使用属性路由[Route("api/[controller]")]，最好再用[ApiController]；不过因为api不应该更改而类名可能更改，可以考虑不用中括号改用绝对路径
* 方法添加[HttpPost]等特性，不加就是Get，可以加逗号应用多个。方法名不重要，只会路由到控制器，然后看Verb；特性支持路由参数，如[HttpGet("{id}")]，就可以传递到方法的参数；如果方法的重载无法区分，可以用特性的name属性
* NoContent、Ok、CreatedAtRoute
* [FromBody]等：用在函数形参上，推断绑定源，这样参数可以直接是自定义的类，有时不加也行
* 返回值是Task IActionResult T，return可以直接返回对象，会自动序列化成JSON
* 创建带有验证的WebApi：https://www.youtube.com/watch?v=_LdiqQ13NBo；官方文档用的是IdentityServer4
* 默认输入和输出的都是JSON(Content-Type: application/json)，可以用AddControllers().AddXmlDataContractSerializerFormatters();添加XML的支持；收到不支持的Accept: xxx可以在AddControllers的选项里用ReturnHttpNotAcceptable=true，则会返回406 Not Acceptable

## IIS Express

* https://github.com/aspnet/AspNetCore/issues/1012
* https://stackoverflow.com/questions/47899494
* https://stackoverflow.com/questions/36282550
* https://blog.maartenballiauw.be/post/2019/02/26/asp-net-core-iis-express-empty-error-starting-application.html
* https://docs.microsoft.com/iis/extensions/using-iis-express/running-iis-express-from-the-command-line
* UseIISIntegration、UseIIS
* https://docs.microsoft.com/zh-cn/aspnet/core/host-and-deploy/iis

## 示例项目

* https://github.com/AiursoftWeb/Tracer
* 获取请求发生的时间：https://stackoverflow.com/questions/50589179

## 其它

* libman：vs自带的管理前端库的工具，类似于npm，可以直接放到wwwroot中；bundleconfig.json：把css合并起来，需要安装BuildBundlerMinifier，也可以minify。感觉都没什么用
* 限制IP访问：https://github.com/stefanprodan/AspNetCoreRateLimit

### 验证

* 需要想的问题：身份验证，登录
* Authentication是证明你是你，Authorization是证明你有这个权限
* 多服务的验证的一般流程：服务器A用私钥签名信息存到cookie里，客户端访问服务器B，B用A的公钥进行验证，这样A和B不需要通信。用Https是没用的
* 一般只能在一个模块中处理密码，验证通过以后生成一个Token令牌；密码会一直有效，而Token有有效期。这就是JWT

### 未读

* 后端API学习指南：https://zhuanlan.zhihu.com/p/38215531
* SOA和微服务：https://www.zhihu.com/question/37808426、https://zhuanlan.zhihu.com/p/88095798、https://www.zhihu.com/question/37808426
* UseCors
* Redis：https://www.bilibili.com/video/av41724451
* .NET 微服务：适用于容器化 .NET 应用程序的体系结构（书，2.2）：https://docs.microsoft.com/zh-cn/dotnet/architecture/microservices/
* https://zhuanlan.zhihu.com/p/46894251
* https://zhuanlan.zhihu.com/p/127415186

## 参考

* https://zhuanlan.zhihu.com/p/82903204
* https://zhuanlan.zhihu.com/p/39692934
* https://www.bilibili.com/video/av65313713
* https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/index

### TODO

* https://github.com/davidfowl/AspNetCoreDiagnosticScenarios
