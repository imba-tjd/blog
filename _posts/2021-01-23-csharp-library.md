---
title: C#库
---

## System.Text.Json

* 默认只序列化公共属性
* 包含System.Net.Http.Json后HttpClient可用GetFromJsonAsyn和PostAsJsonAsync，且具有Web选项
* json-everything：第三方库，包括JsonSchema、JsonPath等

```c#
using System.Text.Json;
using System.Text.Json.Serialization;

public class Product {
    public string ID { get; set; }

    [JsonPropertyName("img")]
    public string Image { get; set; }

    public int[] Rating{ get; set; }

    public override string ToString() => JsonSerializer.Serialize<Product>(this);
}

class JsonFileProductService {
    string JsonFileName = Path.Combine(root, "data", "products.json");
    public IEnumerable<Product> GetProducts() {
        using var instream = File.OpenRead(JsonFileName);
        return JsonSerializer.Deserialize<Product[]>(instream.ReadToEnd());
    }
    public void SaveProducts(IEnumerable<Product> products) {
        using var outstream = File.OpenWrite(JsonFileName);
        JsonSerializer.Serialize(new Utf8JsonWriter(outstream), products); // 也可用Async方法
    }
}

// 直接操纵JSON
using var doc = JsonDocument.Parse(json);
doc.RootElement...
对象：EnumerateObject()返回JsonProperty(k-v)列表，TryGetProperty()，也可以直接[]
数组：EnumerateArray()返回JsonElement列表，GetArrayLength()
值：GetInt32()，GetDouble()，GetString()，GetBoolean()

// System.Text.Json.Node
JsonNode.Parse(jsonstr)
jNode["prop"].GetValue<int>()
new JsonObject {
    ["key"] = new JsonObject {...}
}.ToJsonString()

using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream);
writer.WriteStartObject();
writer.WriteNumber("temp", 42);
writer.WriteEndObject();
```

### JsonSerializerOptions

* 有性能开销，循环序列化应构造实例复用
* 有拷贝构造函数
* WriteIndented = true：格式化
* PropertyNamingPolicy = JsonNamingPolicy.CamelCase：目前只有这一个选项，且是Web的默认值；另有DictionaryKeyPolicy
* ReadCommentHandling = JsonCommentHandling.Skip：允许注释
* AllowTrailingCommas = true：允许尾随逗号
* PropertyNameCaseInsensitive = true：默认false但Web中默认为true
* IncludeFields：也序列化字段
* IgnoreReadOnlyProperties/Fields：忽略只读或只有get的；还有[JsonIgnore]
* Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping：默认会转义所有非ASCII字符，此选项是最大限度的放松
* MaxDepth：默认64

## 第三方库

* https://www.hangfire.io/
* https://github.com/icsharpcode/SharpZipLib
* https://masstransit-project.com/
* https://github.com/App-vNext/Polly 弹性及瞬态故障处理库，方便指定重试次数、超时、熔断等
* https://github.com/pythonnet/pythonnet
