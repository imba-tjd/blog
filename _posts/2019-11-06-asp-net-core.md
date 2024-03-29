---
title: ASP.NET Core
category: dotnet
tags:
    - C#
    - ASP.NET
---

## 初始化

```bash
dotnet new webapp --no-https
dotnet watch
dotnet dev-certs https --trust/--clean # 安装sni为localhost的证书；Linux下只会生成不会自动安装
隐式引用：Microsoft.AspNetCore.Builder、Hosting、Http、Routing; Microsoft.Extensions.Configuration、DependencyInjection、Hosting、Logging;
```

## Builder

```c#
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddRazorPages(); // WebApi用AddController()，MVC用AddControllersWithView()
var app = builder.Build();

app.UseForwardedHeaders(new() { ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto }); // 仍仅信任本地环回发的
if (!app.Environment.IsDevelopment()) { // .NET6 IsDevelopment()下自动UseDeveloperExceptionPage()
    app.UseExceptionHandler("/Error"); // 出错时重定向到指定路由
    app.UseHsts();
}
app.UseHttpsRedirection(); // 307
app.UseStaticFiles(); // 使用时用~表示web根目录，默认wwwroot；另外还有DirectoryBrowser目录浏览和FileServer中间件
app.UseCookiePolicy(); // 添加欧洲GDPR的提示
app.UseRouting();

app.UseCors(); // 还必须Services.AddCors(op=>op.AddDefaultPolicy(builder=>builder.WithOrigins(...)))
app.UseAuthentication(); // 验证，确定用户身份，常见手段如JWT；默认模板存在它是因为某些MVC标签可能用到，如[RequireHttps]表示此资源只能通过HTTPS访问
app.UseAuthorization(); // 授权，判断用户是否可执行操作，比如管理员和普通用户不同
app.UseResponseCompression(); // 因为放在UseStaticFiles()后，不会压缩静态文件；还必须Services.AddResponseCompression();
app.UseResponseCaching(); // 处理客户端发的Cache等Header；7.0预计会出不管客户端怎么要求都进行缓存的中间件

app.MapRazorPages();
app.MapFallbackToPage("/404");
app.MapControllers() // WebApi
app.MapControllerRoute() // MVC

app.Run(); // 与await RunAsync()差不多，会阻塞直到关闭主机。Start()和StartAsync()一般用于非Web程序，不学。RunConsoleAsync()应该是Run()的stub，不支持IIS
```

### Services

* 依赖注入的优点：没有强依赖，利于单元测试、不需要了解具体的服务类、不需要管理服务类的生命周期
* 创建：Services.AddTransient/AddScoped/AddSingleton<IT,T>() 表示每次请求IT时返回T的实例，也可用单参数的T
* 使用：cshtml.cs的构造函数接受IT t，赋值给类字段，或handler的参数也可以直接用。cshtml用@inject It t
* Transient表示每次请求都新实例化，Scoped表示在一次连接中多次实例化都是同一个。另有TryAddXXX方法用于某个接口若已注册了就什么都不做
* 在builder中使用：`using var scope = app.Services.CreateScope(); scope.ServiceProvider.GetRequiredService<SampleService>();`

## Microsoft.Extension.Configuration

* key不区分大小写
* 允许注释和行尾逗号
* 命令行指定：dotnet run --key=val
* 使用：依赖注入IConfiguration config

```c#
{
    "str_config": "value1_from_json",
    "simple_obj": { "value1": "subvalue1_from_json" }
}

builder.Configuration["simple_obj:value1"] // 取数组元素用:[n]
var obj = config.GetSection("simple_obj").Get<SimpleObject>();

var config = new ConfigurationBuilder() // 手动指定要加载的数据源
    .AddJsonFile("appsettings.json") // 一重载支持reloadOnChange
    .AddEnvironmentVariables().AddCommandLine(args) // 允许命令行调用时传--key=val
    .AddInMemoryCollection(dict)
    .Build();

config.GetValue<int>("Key", defaultVal); // 直接用索引器只会返回字符串
config.GetSection("KeyPath"); // 返回子节，永远不会返回null，如果找不到会返回{}，或用.Exists()判断是否存在，或用GetRequiredSection()则不存在时抛异常
config.GetChildren(); // 返回IEnumerable<IConfigurationSection>
```

### appsettings.json

* 修改后会立即生效，前提是支持动态加载，像绑端口就不行
* 多环境（Environment）
  * 发布后运行会读取ASPNETCORE_ENVIRONMENT环境变量，支持Development Staging Production(默认)
  * 获得当前是哪种环境：IWebHostEnvironment env; env.IsDevelopment()
  * 不同环境下会加载完普通的appsettings.json后再加载appsettings.xxx.json，与普通的**覆盖合并**
  * cshtml中用environment include/exclude指定不同环境下的行为
* 实现自定义选项用`Microsoft.Extensions.Options`，要先定义一个类，到服务中注册一下，使用时`IOptionsMonitor/IOptionsSnapshot<MyOptions> optionsAccessor.CurrentValue/Value/Get("Key")`
* Properties/launchSettings.json
  * 仅用于本地开发，即使手动复制到publish里也没有用
  * dotnet run会使用profiles中第一个"commandName"为"Project"的条目，代表使用Kestrel；用--launch-profile xxx使用指定的配置。VS里运行才使用那个IIS的配置
* 配置终结点：默认只会监听localhost:port，其中端口不同版本不同。可指定`"urls": "http://localhost"`或命令行--urls=...或用Run()的重载，支持设为`*`和`[::]`
* Kestrel对象：能对KestrelServerOptions选项进行设置，包括限制流量、指定绑定的端口

## 日志

* Application Insides可以图形化查看日志
* TODO: https://www.youtube.com/watch?v=oXNslgIXIbQ

```c#
builder.Logging.AddJsonConsole();
services.AddLogging(); // 然而这几个不做也能获得依赖注入和控制台日志
Configure(ILoggerFactory loggerFac){ // 应该有选项能记录到文件
loggerFac.AddConsole();
loggerFac.AddDebug();
} // 不清楚Debug是什么

ILogger<XXXController> logger; // DI进来
logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id);
logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);

// appsettings.json
"Logging": {
  "LogLevel": {
    "Default": "Information", // 可考虑Development下设为Debug
    "Microsoft": "Warning", // Key为命名空间
    "Microsoft.Hosting.Lifetime": "Information",
    "Microsoft.AspNetCore.Hosting.Diagnostics": "Information" // 收到请求时显示url和方法以及花费的时间
  }
}
```

## 会话和应用状态

```c#
services.AddDistributedMemoryCache(); // 不加也行？也可使用数据库真正的分布式后备缓存，此时使用Session前必须要用LoadAsync，否则同步获取会造成性能损失；可注册DistributedSession，则不调用LoadAsync时抛异常
services.AddSession(options => {
    options.Cookie.Name = ".AdventureWorks.Session";
    options.IdleTimeout = TimeSpan.FromSeconds(10); // 默认20分钟
    options.Cookie.HttpOnly = true; // 默认true
    options.Cookie.IsEssential = true; // 默认false
});
// 在UseAuthorization之后添加：
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

## 其它对象

* HttpContext.Connection.RemoteIpAddress
* HttpRequest HttpResponse
* 处理出错时的信息：`var ctx = HttpContext.Features.Get<IExceptionHandlerFeature>(); ctx.Error`

## Razor和MVC和WebApi

### Razor

* cshtml，第一行必须是@page
* @using引用命名空间，@inject IT t使用服务
* @model：提供Model属性用于访问传递到视图的模型
* HTML代码中`@表达式`或`@(表达式)`可使用变量的值、属性、索引器、函数、await
  * 如果表达式是字符串，里面的内容会经过HTML编码，显示出来的就是字符串原来的样子；如果想把字符串当作HTML，用HtmlHelper.Raw；非IHtmlContent的表达式会自动ToString
* 两个@会转义一个，email链接中单个@就为字面量
* `@* *@`为razor的注释，最优先，不会发送到客户端
* `@{}`里可以写C#代码，可以声明变量和函数，可以调用函数，可以写C#的注释。关键是里面也可以写HTML和普通的@：函数可以返回void，函数体只有HTML（Core3）；如果编译器无法分辨语言而报错或者不想有空格，可用text标记把HTML括起来，或者用`@:`表示该行后面都是HTML
* 以@开头的命令：if（else和else if就不用@了）、switch、for、foreach、while、dowhile、using、try,catch,finally、lock
* @functions没看懂有什么用。好像是不用就只能写本地函数，用了能用属性和public以及写OnGet，也可用@Functions.xxx调用；不用PM时可用
* View Component：属于高级用法，PartialView不能添加业务逻辑，Controller无法到处复用
* @helper在3中无法使用了
* 启用运行时编译，与watch run不兼容，仅限View层，编辑后刷新能重新编译：添加Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation包，services.AddRazorPages().AddRazorRuntimeCompilation()或在launchSettings中加"ASPNETCORE_HOSTINGSTARTUPASSEMBLIES":"Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation"
* VSC："emmet.includeLanguages": { "aspnetcorerazor": "html" }

### Razor Page

* https://www.learnrazorpages.com
* ViewData：弱类型字典，在分布页面的时候需要用到，比如默认在_Layout中获取了`@ViewData['Title']`设置为title
* 直接访问域名的根会使用**Pages**文件夹（不是Page）下的Index；直接访问其它不带后缀的路径，如果存在文件会用文件，否则会用对应文件夹下的Index；如果还不存在，中间件会往下走，如果还没有，就返回空白页面
* dotnet new page --name Pizza --namespace RazorPagesPizza.Pages --output Pages

```c#
.cshtml代码，Page：
@page "..." //后面可跟路由规则。/或~/开头是以Page开始，否则是相对路径；{id}和{参数名:int}可添加参数，后者是约束，还可以是alpha:minlength(4)；{id?}中的问号表示参数可选，能把?id=xxx变成路径；View中可用RouteData.Values["id"];
@model IndexModel // 可以是IEnumerable<T>，但不知道怎么用；一个viewmodel可以对应多个view

.cshtml.cs代码，PageModel：
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
namespace MyWebapp.Pages; // 子文件夹用别的命名空间应该也可以
// 大部分内容都要是public的，依赖注入放在构造函数里
public class IndexModel : PageModel { // 继承了HttpContext属性
    [BindProperty(SupportsGet=true)] public int Id {get;set;} // View中可以用@Model.Id获取或赋值，但只有想修改时才添加BindProperty，默认只在Post时有效；对于Get添加后可以写在OnGet参数中，能直接赋进去；但它在Mvc命名空间下
    public void OnGet(int id){ } --or-- public async Task OnGetAsync(){ }
    public async Task<PageResult> OnPostAsync() { // 可以有Model类型的参数
        if (!ModelState.IsValid)
            return Page();
        ...
        return RedirectToPage("..."); // 什么都不加就是首页
    }
}

// 改变路由，具体见：https://docs.microsoft.com/zh-cn/aspnet/core/razor-pages/razor-pages-conventions
.AddRazorPagesOptions(options => { // 不是RazorOptions
options.Conventions.AddPageRoute("/extras/products", "product");});
//xxx.com/product映射到extras/products，注意顺序；第二个参数可以是"{*url}"或""
// options.RootDirectory可更改默认的Pages文件夹
```

### 结构

* Services、Models、Data、Web根(wwwroot)
  * Data类似于Java的Service，放IxxxDataStore.cs和某数据库xxxDataStore.cs
* js，css，lib，favicon放在wwwroot下作为静态文件，可以用IWebHostEnvironment.WebRootPath获取
  * 在 Razor.cshtml 文件中，~/ 指向 Web 根。 以 ~/ 开头的路径称为虚拟路径

### 布局

* 使用时路径可以写相对路径或绝对路径，如果是相对路径，寻找顺序为本文件夹、递归父文件夹、/Shared、/Page/Shared（至少分部视图是这样）；所以这些文件可以在子文件夹里创建，用来覆盖父文件夹的设置
* Pages/_ViewImports.cshtml中添加@using和@inject语句，且这个using也具有“自动路由”，后缀部分按它和目标之间的文件夹以点分隔，模板生成的前缀是项目名.Pages
* Pages/_ViewStart.cshtml中添加每个Page渲染前运行的语句（分部视图除外），默认只有`@{Layout = "_Layout";}`，在具体Page中可手动设为null或其它的
* Pages/Shared/_Layout.cshtml中添加大部分的模板，必须使用`@RenderBody()`渲染具体的部分；但又有个IgnoreBody方法，看不懂
* 区域：在具体Page中写`@section Scripts {...}`，在Layout中写`{@RenderSection("Scripts", required: false)}`，其中Scripts是标识符，required如果不为false，效果是找不到对应区域时报错
* 分部视图：约定命名为Shared/_XXXPartitial.cshtml，使用`<partial name="_ValidationScriptsPartial" />`或@{await HTML.RenderPartialAsync()}渲染，优先前者

### Tag Helper

* @add/removeTagHelper：用于简化表单和验证，如asp-for
* https://github.com/52ABP/Documents/blob/V1.0.0/src/mvc/35.Tag-helpers-in-asp.net-core.md、https://github.com/52ABP/Documents/blob/V1.0.0/src/mvc/36.Why-use-tag-helpers.md
* asp-page-handler：可以使用OnPostDeleteAsync这样的函数
* asp-page/asp-controller/asp-action/asp-route-xxx：MVC的
* asp-href-include
* 好像是替代htmlhelper的
*  asp-validation-summary="All" asp-validation-for
* partial
* `<input asp-for="PageModel.属性">`：自动添加id和name，根据PageModel指定的属性类型自动设置input的类型
* asp-items：与asp-for配合用在select元素中

#### HTML Helper

* @Html.DisplayNameFor/DisplayFor/DisplayForModel
* @Html.ActionLink：好像仅MVC

### 路由

* https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/routing
* https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/routing
* 不区分大小写
* 可以是某个cshtml文件，也可以是那个文件夹下的Index.cshtml
* action的Async后缀现在无需在url中添加
* 命名路由，区分大小写：MapGet().WithName("xxx"); 另一终结点(LinkGenerator linker) => linker.GetPathByName("hi", values: null)
* {*rest}捕获剩余全部路径
* {slug:regex(^[a-z0-9_-]+$)}
* 显式参数绑定
* 可选参数

### Blazor Server

* 以.razor位后缀的组件，在@code{}里写C#代码，能起到JS的作用，元素属性加 @事件="handler名"

### MVC

* 默认路由为{controller=Home}/{action=Index}/{id?}，代表默认找HomeController的Index方法，Index方法有一个名字叫做id的参数，问号允许不传时用默认值
* 建立Controllers文件夹，里面放名字以Controller结尾的类，继承Controller，放xxx.Controllers命名空间下。文件夹的路径或者是否在子文件夹里不重要
* 类里的方法就是Action，方法的返回类型为Task IActionResult，实际可返回View() RedirectToAction(nameof(Index)) Json()等
* View需和Action的前缀对应；控制器和View的数据用ViewBag交互
* 也可以用ViewModel，返回View()的时候传对象进去，View中用和Razor Page一样的用法
* 方法的参数默认是用QueryString查询字符串?key=val传递的，但可在路由规则中写{id?}变成路径；注意名字必须对应
* WinForm采用事件响应机制，难以做到View层和Controller层分离，也就无法真正实现MVC架构

### [WebApi](https://docs.microsoft.com/zh-cn/aspnet/core/web-api)

* dotnet new webapi --no-openapi
* 命名空间使用Mvc
* 继承ControllerBase，添加[ApiController]和[Route("api/[controller]")]特性。其中[controller]表示去掉结尾的Controller的类名
* 方法
  * 添加[HttpPost]等特性，不加就是Get，可以加逗号应用多个
  * 方法名不重要，只会路由到类，然后看Verb。但也可以在[HttpGet(...)]中添加子路由
  * 特性支持路由参数传递到方法的参数，如[HttpGet("{id}")]
  * 返回值可以是具体类型，会自动json序列化。可以是IEnumerable。可以是`ActionResult<T>`
  * 形参上可加[FromBody] FromHeader推断绑定源，这样参数可以直接是自定义类，有时不加也行
* 默认输入和输出的都是JSON(Content-Type: application/json)。在AddControllers的选项里用ReturnHttpNotAcceptable=true 对其它的返回406 Not Acceptable
* dotnet tool install -g Microsoft.dotnet-httprepl; httprepl localhost:port 之后可以输入ls cd get命令

### Minimal API

```c#
// dotnet new web
var app = WebApplication.Create(args);
app.MapGet("/", () => "Hello World!");
app.MapGet("/{id}", async (int id, DB db) => await db.FindAsync(id) is C c ? Results.Json(c) : Results.NotFound()); // DB是依赖注入的
app.Run("http://localhost:3000");
```

### IActionResult

* 首先现在一般用`ActionResult<T>`，因为IActionResult是非泛型的
* 作为方法的返回值
* T可以隐式转换成`ActionResult<T>`
* 方法可以是Async的，返回值可以再包一层Task
* 预定义的ActionResult的子类对象：NoContent()用在PUT和DELETE中、Ok()、BadRequest()、NotFound()
* Created()、CreatedAtRoute()、CreatedAtAction()：表示201资源创建成功。区别：https://ochzhen.com/blog/created-createdataction-createdatroute-methods-explained-aspnet-core

### Swagger/OpenAPI

```c#
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
if (app.Environment.IsDevelopment()) {
    app.UseSwagger(); app.UseSwaggerUI();
}
```

## 其它

* libman：vs自带的管理前端库的工具，类似于npm，可以直接放到wwwroot中；bundleconfig.json：把css合并起来，需要安装BuildBundlerMinifier，也可以minify。感觉都没什么用
* 限制IP访问：https://github.com/stefanprodan/AspNetCoreRateLimit

### 验证

* 需要想的问题：身份验证，登录
* Authentication是证明你是你，Authorization是证明你有这个权限
* 多服务的验证的一般流程：服务器A用私钥签名信息存到cookie里，客户端访问服务器B，B用A的公钥进行验证，这样A和B不需要通信。用Https是没用的
* 一般只能在一个模块中处理密码，验证通过以后生成一个Token令牌；密码会一直有效，而Token有有效期。这就是JWT
* `builder.Services.AddDefaultIdentity<IdentityUser>(op => op.SignIn.RequireConfirmedAccount = true).AddEntityFrameworkStores<ApplicationDbContext>();`
* 创建带有验证的WebApi：https://www.youtube.com/watch?v=_LdiqQ13NBo；官方文档用的是IdentityServer4，不学

## TODO

* 后端API学习指南：https://zhuanlan.zhihu.com/p/38215531
* SOA和微服务：https://www.zhihu.com/question/37808426、https://zhuanlan.zhihu.com/p/88095798、https://www.zhihu.com/question/37808426
* .NET 微服务：适用于容器化 .NET 应用程序的体系结构：https://docs.microsoft.com/zh-cn/dotnet/architecture/microservices/
* https://zhuanlan.zhihu.com/p/46894251
* https://zhuanlan.zhihu.com/p/127415186
* https://zhuanlan.zhihu.com/p/82903204
* https://zhuanlan.zhihu.com/p/39692934
* https://www.bilibili.com/video/av65313713
* https://github.com/davidfowl/AspNetCoreDiagnosticScenarios
* https://zhuanlan.zhihu.com/p/460624318 WebApplicationBuilder
* https://github.com/AiursoftWeb/Tracer
* https://github.com/aspnet/AspNetCore/issues/1012
* 自动扫描 https://github.com/khellang/Scrutor
* https://code-maze.com/net-core-series/
