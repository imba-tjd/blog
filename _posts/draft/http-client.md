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

## Ktor

插件化：日志（默认依赖了Slf4j，会写TRACE）、超时、缓存（内存、持久化） 等。添加依赖后“install”

```kotlin
// implementation(platform("io.ktor:ktor-bom:$ktor_version"))
// implementation("io.ktor:ktor-client-cio") 只支持HTTP1.1但跨全平台，如果要2和WebSocket可改okhttp

import io.ktor.client.*
import io.ktor.client.request.*
import io.ktor.client.statement.*
import kotlinx.coroutines.*

val client = HttpClient() { // 自动选择依赖中可用的，此处是CIO。需要复用
    expectSuccess = 默认false // 返回非200不抛异常
    followRedirects = 默认true
}

fun main() = runBlocking {
    val resp = client.get("https://example.com") {
        headers {
            appendAll(
                HttpHeaders.Accept to "text/html",
            )
        }

        contentType(ContentType.Application.Json)
        setBody(obj) // 需ktor-client-content-negotiation
    }
    println(resp.bodyAsText())
    val cus: Customer = client.get(...).body() // 泛型函数，根据变量类型自动反序列化，需negotiation插件
}
```

### ktor-network

* TCP、UDP、TLS
* tcp noDelay 默认true

```kotlin
import io.ktor.network.selector.*
import io.ktor.network.sockets.*
import io.ktor.utils.io.*
import kotlinx.coroutines.*

val selectorManager = SelectorManager(Dispatchers.IO) // ktor的库跑在这上面，下面示例里的launch如果有自己的库要再加
val serverSocket = aSocket(selectorManager).tcp().bind("0.0.0.0", 9002) { reuseAddress = true; typeOfService = TypeOfService.IPTOS_LOWDELAY }
while (true) {
    val socket = serverSocket.accept()
    println("Server accepted: ${socket.remoteAddress}")
    launch {
        val receiveChannel = socket.openReadChannel()
        val sendChannel = socket.openWriteChannel(autoFlush = true)

        try {
            while (true) {
                val name = receiveChannel.readline() // 不会读入\n；客户端关闭前的最后一次视为发了\n
                sendChannel.writeStringUtf8("Hello, $name!\n")
            }
        } catch (e: Throwable) {
            socket.close()
        }
    }
}

val clientSocket = aSocket(selectorManager).tcp().connect("127.0.0.1", 8443) { keepAlive = true }

客户端：bind()
UDP广播：udp().bind {broadcast = true}，不要写udp().configure{broadcast = true}，安卓上失败
```

## Windows API

### Windows.Networking.Sockets

* 读写器从下层处理字符串默认就是U8的。端序实测此处默认为大端，x.ByteOrder()不赋值就是读取，传参就是修改

```cpp
#include <winrt/Windows.Foundation.h>
#include <winrt/Windows.Networking.h>
#include <winrt/Windows.Networking.Sockets.h>
#include <winrt/Windows.Storage.Streams.h>

using namespace winrt;
using namespace Windows::Foundation;
using namespace Windows::Networking;
using namespace Windows::Networking::Sockets;
using namespace Windows::Storage::Streams;
```

UDP服务端：

```cpp
DatagramSocket g_socket = DatagramSocket();
g_socket.MessageReceived(OnMessageReceived); // 每次收到都会触发，每条请求独立，不会收到一次就结束了；自动在线程池中处理请求
co_await g_socket.BindServiceNameAsync(L"7777"); // 如果要指定ip，用BindEndpointAsync

void OnMessageReceived(DatagramSocket const&, DatagramSocketMessageReceivedEventArgs const& args) {
    // 第一个参数是触发的Socket，因为一个Handler可以绑定到多个Socket上。此处没必要用
    auto reader = args.GetDataReader();
    uint32_t len = reader.UnconsumedBufferLength();
    string msg(len, '\0');

    reader.ReadBytes(array_view<uint8_t>(
        reinterpret_cast<uint8_t*>(&msg[0]),
        reinterpret_cast<uint8_t*>(&msg[0]) + len));

    cout << "Received: " << msg << endl;

    SendReplyUnwait(args.RemoteAddress(), args.RemotePort());
}

fire_and_forget SendReplyUnwait(HostName remoteAddr, hstring remotePort) {
    auto stream = co_await g_socket.GetOutputStreamAsync(remoteAddr, remotePort); // 如果多个线程同时向同一个客户端回复
    DataWriter writer(stream);
    writer.WriteString(L"pong");
    co_await writer.StoreAsync(); // 写入底层流
    co_await writer.FlushAsync();
    writer.DetachStream(); // 分离生命周期，让writer析构时，底层流能复用
}
```

TCP服务端：

* 默认NoDelay

```cpp
class TcpServer {
private:
    StreamSocketListener listener;
    vector<StreamSocket> clients;
    mutex clientsMutex;

public:
    void Start(uint16_t port) {
        listener.ConnectionReceived({ this, &TcpServer::OnConnectionReceived });
        listener.Control().KeepAlive(true);
        listener.Control().QualityOfService(SocketQualityOfService::LowLatency);

        listener.BindServiceNameAsync(to_hstring(port)).get();;
        cout << "TCP Server Listening: " << port << endl;
    }

    void Close() {
        listener.Close(); // 停止继续接收，不影响已有Socket

        lock_guard<mutex> lock(clientsMutex);
        for (auto& client : clients) {
            client.Close();
        }
        clients.clear();
    }

private:
    void OnConnectionReceived(StreamSocketListener sender,
                              StreamSocketListenerConnectionReceivedEventArgs args) {
        try {
            StreamSocket socket = args.Socket();

            HostName remoteHost = socket.Information().RemoteAddress();
            hstring remotePort = socket.Information().RemotePort();
            cout << "新客户端连接："
                      << to_string(remoteHost.CanonicalName())
                      << ":" << ansi(remotePort.c_str()) << endl;

            {
                lock_guard<mutex> lock(clientsMutex);
                clients.push_back(socket);
            }

            HandleClientUnwait(socket);
        }
        catch (const hresult_error& ex) {
            cerr << "连接处理错误：" << to_string(ex.message()) << endl;
        }
    }

    fire_and_forget HandleClientUnwait(StreamSocket socket) {
        DataReader reader{ socket.InputStream() };
        DataWriter writer{ socket.OutputStream() };
        reader.InputStreamOptions(InputStreamOptions::Partial); // 默认None，完全匹配LoadAsync（除非EOF）。Partial允许提前返回，但不保证立即返回。ReadAhead允许系统内部提前读数据。后两者可以组合使用

        try {
            while (true) {
                uint32_t loaded = co_await reader.LoadAsync(1024);

                if (loaded == 0) {
                    cout << "客户端断开连接" << endl;
                    break;
                }

                while (reader.UnconsumedBufferLength() > 0) {
                    co_await process_data(reader, writer);
                }

                reader.DetachBuffer();
            }
        }
        catch (const hresult_error& ex) {
            cerr << "客户端通信错误：" << to_string(ex.message()) << endl;
            // 转换为网络错误枚举：SocketError::GetStatus(ex.code()) 但没有自动转字符串
        }

        reader.DetachBuffer();
        socket.Close();

        lock_guard<mutex> lock(clientsMutex);
        erase_if(clients, [&socket](const StreamSocket& s) {
            return s == socket;
        });
    }
}

并发调用socket.OutputStream().WriteAsync(iBuffer)是安全的，但从它创建DataWriter后并发调用writer不安全
```

#### async

```
#include <winrt/Windows.Foundation.h>

IAsyncOperation<int> FooAsync() {
    int x = co_await BarAsync();
    co_return x;
}

无返回值：IAsyncAction
不允许调用者等待：fire_and_forget返回类型
对于调用方，不等待：auto lambda = [=]() -> winrt::fire_and_forget { try { co_await ... } catch(...) }; lambda();
AI：不能声明为IAsyncAction又不co_await/get()它，该协程对象可能会在执行完毕前就被销毁。Gemini说是安全的，不等待自动相当于co_await，但一般用fire_and_forget包一层处理异常

co_await winrt::resume_on_signal(m_event);
```

### WSA API

* 从17063(可能是2018)支持Unix socket（AF_UNIX）。提供双向关闭语义，而命名管道没有。类型仅支持流(SOCK_STREAM)，寻址格式支持pathname；abstract实际上不支持，unnamed因为socketpair不存在而基本不支持
* WSAAsyncSelect：用于GUI的

```c
#include <winsock2.h> // -lws2_32。不要用wsock32
#include <ws2tcpip.h>
#include <mswsock.h> // IOCP
#include <windows.h> // 必须后写，或用WIN32_LEAN_AND_MEAN

WSAStartup(MAKEWORD(2, 2), &(WSADATA){}); // WSACleanup()
SOCKET so = socket(AF_INET, SOCK_STREAM/SOCK_DGRAM, IPPROTO_TCP/IPPROTO_UDP); // closesocket(so)

SOCKADDR_IN addr;
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = INADDR_ANY; // inet_addr("127.0.0.1")
addr.sin_port = htons(12345); // sin_port内部为大端序而非普通uint16_t，此函数转换
bind(so, (SOCKADDR*)&addr, sizeof(addr));
```
