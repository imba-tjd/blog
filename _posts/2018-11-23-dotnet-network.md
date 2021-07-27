---
title: ".NET网络编程"
---

## System.Net.Http

### HttpClient

* 构造函数可以直接用无参的，也可以接受HttpClientHandler
* BaseAddress：获取或设置基地址， 使用URN时会拼接上
* DefaultRequestHeaders：返回一个HttpRequestHeaders对象，具体见下面；无法用初始化器初始化因为它只是一个引用，无法set
* Timeout：接受TimeSpan，默认100秒，DNS查询15秒，需要无限可用System.Threading.Timeout.InfiniteTimeSpan；超时时是在获取结果时抛TaskCanceledException
* DefaultRequestVersion = new Version(2, 0)：使用HTTP/2
* GetByteArrayAsync、GetStreamAsync、GetStringAsync：只接受Uri对象或字符串，编码不清楚
* PostAsync、PutAsync、PatchAsync、SendAsync、DeleteAsync、GetAsync：发送对应的请求，都支持取消；其中前三者需要HttpContent，Send就是自定义其它所有请求，需要HttpRequestMessage；返回都是`Task<HttpResponseMessage>`
* `System.Net.ServicePointManager.DefaultConnectionLimit`限制最大并发数，默认是两个

### HttpClientHandler

* 用于更改client的默认行为，传给它的构造函数
* 设置缓存行为、自动压缩、认证、代理、重定向（是否允许、最大次数）、SSL协议等
* UseCookies默认为true，用CookieContainer可以获取/设置cookies
* UseProxy默认为true，Proxy默认为null，会使用系统的代理设置；new Proxy可手动设置代理
* AllowAutoRedirect默认为true（从https自动重定向http好像会抛异常）
* AutomaticDecompression枚举默认为None不使用压缩，可以用GZip和Flags特性
* 可以使用链式处理：自定义一个继承DelegatingHandler的类，然后把下一个handler传给InnerHandler属性，最后一个通常是无参的HttpClientHandler。发送请求时是从前往后处理，接收时从后往前
* core2.1后会自动使用更高级的SocketsHttpHandler，使用了Span

### HttpContent

* 既是响应信息，也作为请求的body
* 请求时实际一般用new StringContent/StreamContent，因为它本身的构造函数是protected的
* 响应时一般用ReadAsStringAsync或ReadAsByteArrayAsync，内容大于80KB时最好用ReadAsStreamAsync或CopyToAsync
* 5之后可用JsonContent

#### FormUrlEncodedContent

提交表单，不知道是否会自动设置ContentType。

```c#
var content = new FormUrlEncodedContent(new[] {
    new KeyValuePair<string, string>("email", "xxxx"),
    new KeyValuePair<string, string>("password", "xxxx"),
});
content.Headers.
var result = await client.PostAsync("https://www.xxxx.com/login", content);
```

### HttpResponseMessage

* StatusCode、IsSuccessStatusCode、EnsureSuccessStatusCode()
* Content.ReadAsStringAsync()
* RequestMessage获取导致此响应消息的请求消息，如果发生过跳转就会与最初的不同；可以获取Method和RequestUri等

### HttpRequestHeaders

* 本身位于System.Net.Http.Headers命名空间中，但是一般用`HttpClient.DefaultRequestHeaders`获取，回复用`response.Headers`；是引用类型，修改它就好，不需要也不能再赋值回来；或者每次请求也可以单独使用它
* 用于设置HTTP请求头
* 修改UA：`Add("User-Agent", "...");`或`UserAgent.ParseAdd`，默认不存在
* ContentType = new MediaTypeHeaderValue("application/json") { CharSet = "utf-8" }
* TryGetValues可以获取它的值

## 给网址进行编码

* 最合适的方法是先分别把键值对用Uri.EscapeDataString编码再手动合并起来，保留以下符号：`!'()*-._~0-9a-zA-Z`，会把`://`编码掉，5之后不再限制长度；解码用Uri.UnescapeDataString
* Uri.EscapeUriString过时了。适合把URL中的中文编码掉；会保留url中的以下符号：`!#$&'()*+,/:;=?@-._~0-9a-zA-Z`，所以会保留`://`；如果参数本身需要那些符号，此方法就会出错；只支持编码成utf8的转义，没有UnEscapeUriString方法
* System.Net.WebUtility与System.Web.HttpUtility类似，但后者在.Net Core上是在Web命名空间里唯一的一个类了；HttpUtility.UrlEncode可以指定编码方式，WebUtility.UrlEncode与Uri.EscapeDataString相比会编码波浪号
* 曾经空格会被编码成加号，后来弃用了；但表单的提交不符合最新标准，仍用加号，许多实现也支持把加号解码成空格
* WebUtility.HtmlEncode：此方法用于把Html特殊字符转换成Html实体，比如大于小于符号，与url编码无关
* HttpUtility.UrlPathEncode：不要使用此方法，已弃用
* System.Text.Encodings.Web.UrlEncoder.Default.Encode：Core新增
* 几种编码方式的结果对比：https://stackoverflow.com/a/21771206/9606292；Uri的两个方法的对比：https://stackoverflow.com/questions/4396598；

## 安全性

* https://docs.microsoft.com/zh-cn/dotnet/framework/network-programming/security-in-network-programming
* ServicePointManager.SecurityProtocol |= SecurityProtocolType.Tls12;：fx4.8/core3.0才支持1.3，win7必须设置这个否则只会使用1.0
* `ServicePointManager.Expect100Continue = true;`

## Cookie

```c#
Cookie cookie = new Cookie("Name", Value);
cookie.Secure = true;
cookie.Domain = “www.domain.com”;

var cookies = new CookieContainer();
var handler = new HttpClientHandler() {CookieContainer = cookies};
HttpClient client = new HttpClient(handler);

response.Headers.TryGetValues("set-cookie", out cookies)

cookieContainer.Add(cookie);
cookieContainer.Add(new Uri("..."), "a=a,b=b"/response.Cookies); // 省略第一个参数通常会失败；分隔符要是逗号，不能是分号；不清楚和SetCookies方法的区别
```

## System.Net.Socket

```c#
using System;
using System.Net.Security;
using System.Net.Sockets;
using System.Threading.Tasks;

static class TLS {
    static async Task ConnectCloudFlare() {
        var targetHost = "www.cloudflare.com";

        using TcpClient tcpClient = new TcpClient();

        await tcpClient.ConnectAsync(targetHost, 443);

        using SslStream sslStream = new SslStream(tcpClient.GetStream());

        await sslStream.AuthenticateAsClientAsync(targetHost);
        await Console.Out.WriteLineAsync($"Connected to {targetHost} with {sslStream.SslProtocol}");
    }
}
```

## 例子

* https://github.com/wangqiang3311/HttpRequestDemo

### 只获取响应头

```c#
// 一旦获取消息头即可完成相关的请求操作，在获取源的大小或者验证的源的可用性时很好用
public static System.Net.Http.Extension{
    public static async Task<long> GetContentLengthAsync(this HttpClient client, string src) {
        using (var request = new HttpRequestMessage(HttpMethod.Head, src))
        using (var response = await client.SendAsync(request, HttpCompletionOption.ResponseHeadersRead).ConfigureAwait(false))
            return response.Content.Headers.ContentLength ?? -1;
}}
```

## HttpClientFactory

HttpClient一般用单例模式/复用，不要用using，否则会耗尽socket。但这样无法反映DNS变动。解决方法：用`DefaultRequestHeaders.ConnectionClose = true;`，相当于Http头的keep-alive为false，但socket还是没有复用。或者修改`ServicePointManager.FindServicePoint(baseUri).ConnectionLeaseTimeout`（默认-1永不关闭）和`ServicePointManager.DnsRefreshTimeout`（默认两分钟）。

.NET Core 2.1后加入了HttpClientFactory，用于统一管理HttpClient实例。但是太复杂了看不懂。而且在FX上用需要装两个包，几百兆。

## 其它

如果 ThreadPool 设置了最大并行数量，一旦超过最大并行数，CLR会先挂起所有线程，然后在排队进行，但是Http是不支持挂起的，就会直接终止。

指定**发送**的ip和端口，用于多网卡的情形：`ServicePointManager.FindServicePoint(new Uri(url)).BindIPEndPointDelegate = (a, b, c) => new IPEndPoint(IPAddress.Parse(ip), 0);`

FindServicePoint的参数只会考虑scheme、host和port

Ping：new System.Net.NetworkInformation.Ping().SendPingAsync(host, 5); p.Status = IPStatus.Success

## 过时的技术

### HttpWebRequest

* 默认已KeepAlive
* 用完后释放Response或Stream都可以
* POST：设置Method和ContentLength，往GetRequestStream()里写
* 有一些Async方法，但不能和同步的混用

```c#
var request = WebRequest.CreateHttp(ADDR);
using WebResponse response = request.GetResponse(); // 必要时强转
var receiveStream = response.GetResponseStream(); // 另一种用法是CopyTo到内存流
var reader = new StreamReader(receiveStream);
string content = reader.ReadToEnd();
```

### WebClient

* var client = new System.Net.WebClient()
* client.DownloadString()、DownloadFile()

## 其它

* IIS Express：https://zhuanlan.zhihu.com/p/64424475
* 代理：https://docs.microsoft.com/zh-cn/dotnet/framework/network-programming/accessing-the-internet-through-a-proxy
* "C:\Program Files (x86)\IIS Express\iisexpress.exe" /path:C:\MyWeb /port:8000

## 参考

* https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/async/
* https://zhuanlan.zhihu.com/p/89106847
* https://www.cnblogs.com/dudu/archive/2011/02/25/asp_net_UrlEncode.html

### TODO

* https://github.com/tmenier/Flurl 最近才发了3.0，但依赖Newtonsoft.Json
