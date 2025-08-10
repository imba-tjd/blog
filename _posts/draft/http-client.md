# Http Client

## 功能

* Path：模板
* Header：MultiMap
* Body：纯数据、application/x-www-form-urlencoded、multipart/form-data
* 透明gzip压缩
* 跟随跳转
* 重试：根据DNS解析结果尝试不同IP
* 超时：连接、发送、接受、总
* 缓存：支持过期、限制大小
* Cookie
* Proxy
* TLS：信任哪些证书（默认系统，支持自定义）、使用的加密套件
* 连接池：每主机最大连接数
* Basic Auth
* HTTP/2、Websockets、SSE
* 可观测性、拦截器

## [OkHttp](https://square.github.io/okhttp/)

* 其他库
  * retrofit 同一组织出的，用于把restapi封装成易使用的类
  * feign：在多种http客户端上的封装，受retrofit启发。另有Spring Cloud OpenFeign但只维护了

```java
var client = new OkHttpClient();
或 new OkHttpClient.Builder().connectionTimeout(60, TimeUnit.SECONDS).cache(...).build()
public static final MediaType JSON = MediaType.get("application/json; charset=utf-8");
var body = RequestBody.create(json, JSON);
var req = new Request.Builder().url(url).post(body).build();
try (var resp = client.newCall(request).execute()) {
  return resp.body().string();
}
异步：call.enqueue(new Callback(){ onFailure(); onResponse(); })
```

## Java HttpClient

* keepalive时间：JDK21默认30秒，之前默认20分钟

```java
var client = HttpClient.newHttpClient(); // JDK11
var request = HttpRequest.newBuilder() // URI也可传进这里面
    .uri(URI.create("https://javastack.cn")).header(k,v).timeout(duration).GET().build();
// 同步
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
// 异步
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```
