# 各语言JSON库

## 功能

* 映射规则：CamelCase对应SnakeCase全局设置、单个字段设置
* 类型转换器
* 序列化
  * 忽略某字段
* 反序列化
  * 额外字段：一般默认忽略
  * 缺少字段：一种策略是设为默认值，另一种策略是调用构造函数
* 引用类型null值的处理
  * 序列化：若value是null，是否序列化key
  * 反序列化：key存在但value为null
  * 整个jsonstr或对象是null
* 源：字符串、数据流、树形结点
* 注释、结尾的逗号

## [moshi](https://github.com/square/moshi)

* com.squareup.moshi:moshi
* okhttp组织出的，是原gson开发者做的

```java
Moshi moshi = new Moshi.Builder().build(); // 此处添加类型转换器
JsonAdapter<Model> ada = moshi.adapter(Model.class); // 此处指定处理null、缩进

String jsonstr = ada.toJson(m);
Model m = ada.fromJson(jsonstr);
```

## Jackson

* com.fasterxml.jackson.core:jackson-databind
  * 处理LocalDateTime：jackson-datatype-jsr310
* TODO: Spring中的配置

```java
ObjectMapper mapper = JsonMapper.builder()
    .findAndAddModules()
    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false) // 默认为true，设为false后当JSON存在Bean没有的字段时不报错
    .configure(MapperFeature.ACCEPT_CASE_INSENSITIVE_PROPERTIES, true) // 便于反序列化record
    .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false) // 默认true将Date和TS序列化为unix时间戳毫秒，设为false后默认为ISO格式0时区字符串；LocalDateTime分别为内部结构表示和当前时区ISO格式。Spring默认false
    .build();

mapper.setPropertyNamingStrategy(PropertyNamingStrategies.LOWER_CASE) // 默认不改变大小写。还有SNAKE_CASE
    .setTimeZone(TimeZone.getDefault()) // 仅当不设置DateFormat且WRITE_DATES_AS_TIMESTAMPS=false时考虑使用，设置了DateFormat似乎默认就变成当前时区了
    .setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")); // 对LocalDateTime无效

mapper.writeValueAsString(obj); writeValue(os, obj)
mapper.readValue(str/in_stream, clazz); // 一般的类要求存在无参ctor，record不必
类字段注解：@JsonIgnore、@JsonProperty("重命名", access = Access.WRITE_ONLY)、@JsonFormat(pattern="yyyy-MM-dd",timezone = "GMT+8")
类注解：@JsonIgnoreProperties(ignoreUnknown = true)
前端无法直接处理Long，使用：@JsonSerialize(using=ToStringSerializer.class)，反序列化默认就支持传入字符串
将一般的对象变为Map：mapper.convertValue(obj, Map<String,Object>.class)
```

## FastJson2

* com.alibaba.fastjson2:fastjson2、fastjson2-extension-spring6

```java
JSON.toJSONString(o)
JSON.parseObject(str, clazz)
JSONArray.from(List)
jsonarr.toList(clazz)
对于LocalDateTime，默认无T
```

## Python

* dumps/loads：字典/对象与字符串互转，后者还支持直接从bytes转换。对于纯英文的可用`b'{...}'`定义
* dump/load(file)：从filelike中写入/读取json；如果文件为空会抛异常
* 序列化的参数：indent=4缩进格式化，sort_keys=True进行排序；默认会把中文变成\u的转义，用ensure_ascii=False可以保留中文
* 不允许有注释
* 有些东西不可自动json序列化，如datetime；有的东西不能序列化，如线程锁
* 简单的自定义dataclass：序列化指定default=vars，每一层都会使用。单层反序列化loads后`**`解包到构造函数里或者直接设置`__dict__`属性，多层时每个类创建from_dict类方法，处理好数据后返回cls()构造函数
* echo '{"json": "obj"}' | python -m json.tool：命令行工具验证与格式化
* 二进制序列化/反序列化可用pickle，第二个参数可选协议版本，目前默认4最新5，-1永远用最新的；可序列化几乎任何对象，因此要保证来源可信。使得自定义类能序列化：实现reduce魔术方法。扩展了pickle，能自动序列化更多类型的第三方库：cloudpickle，不保证持久兼容性，可用于多进程共享对象
* 持久化字典：`with shelve.open('data') as db`。内部使用了pickle，默认可能不是最新版协议。当需要原地改变里面的值时需设置writeback=True，否则默认False无需更改。内部使用了dbm，没必要单独用它。Win下会生成data.bak .dat .dir三个文件，Linux下就是data。第三方有sqlitedict
* 第三方库：ujson(ultrajson,C)、simplejson(Py)、pyjson5(很慢)、orjson(rust，支持序列化dataclass和日期)。tomllib：3.11自带，只能解析

## Newtonsoft.Json

* 部分功能依赖Microsoft.CSharp命名空间引用
* 反序列化时json字面量支持单引号

```c#
JsonConvert.SerializeObject(obj [,Formatting.Indented]); 支持基元类型、IEnumerable、IDictionary
JsonConvert.DeserializeObject<T>(str);
SerializeObject<dynamic>()，之后object可用.xxx或同JObject，数组同JArray
// 文件
var serializer = new JsonSerializer() {NullValueHandling = NullValueHandling.Ignore};
serializer.Serialize(File.CreateText(...), obj)/Deserialize<T>(new JsonTextReader(File.OpenText(...)));
// Newtonsoft.Json.Linq
JObject.Parse(str); 之后当作dict用，还可SelectToken(jsonpath)
JObject.FromObject(匿名对象)
JArray 当作List用，长度用Count
```

## System.Text.Json

* 默认只序列化public的Property，反序列化时自动忽略只读属性
* 包含System.Net.Http.Json后HttpClient可用GetFromJsonAsyn和PostAsJsonAsync，且选项为Web的默认值
* 支持序列化一维和锯齿数组，不支持多维数组。支持许多集合
* 处理溢出内容：反序列化时JSON的内容比类多或操作失误未匹配上，默认会忽略。添加`[JsonExtensionData] Dictionary<string, JsonElement>`可捕获额外内容。如果JSON更少，也不会报错，而Newtonsoft.Json就支持Required属性
* 不支持多态序列化
* json-everything：第三方库，包括JsonSchema、JsonPath等
* FX自带System.Runtime.Serialization.Json，但与本库不兼容，不学，不如直接用Newtonsoft.Json

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

// 直接解析JSON，只读。反序列化成dynamic无意义
using var doc = JsonDocument.Parse(json);
doc.RootElement...
对象：GetProperty()。EnumerateObject()返回JsonProperty(k-v)列表，TryGetProperty()。没有字符串indexer是为了提醒不是O(1)操作
数组：[]。EnumerateArray()返回JsonElement列表，GetArrayLength()
值：GetInt32()，GetDouble()，GetString()，GetBoolean()

// System.Text.Json.Nodes，可写
JsonNode.Parse(jsonstr)
jNode["prop"].GetValue<int>()
new JsonObject {
    ["key"] = new JsonObject {...}
}.ToJsonString()

// 保存
using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream);
writer.WriteStartObject();
writer.WriteNumber("temp", 42);
writer.WriteEndObject();
```

### JsonSerializerOptions

* 有性能开销，循环序列化应构造实例复用
* 有拷贝构造函数
* WriteIndented = true：格式化输出
* ReadCommentHandling = JsonCommentHandling.Skip：允许注释
* AllowTrailingCommas = true：允许尾随逗号
* PropertyNameCaseInsensitive：是否不区分大小写，默认false但Web中默认为true
* IncludeFields：也序列化字段。或对单个想要序列化的字段标识`[JsonInclude]`；忽略用JsonIgnore
* Encoder = JavaScriptEncoder.All/UnsafeRelaxedJsonEscaping：默认会转义所有非ASCII字符，设为All会不再转义所有语言，但仍会转义<和&等，后者仅当确定客户端将内容解释为JSON时才考虑使用
* MaxDepth：默认64
* JsonNamingPolicy：Web中默认为CamelCase
* IgnoreReadOnlyProperties：序列化时忽略只读属性；忽略只读字段用IgnoreReadOnlyFields
* DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull：序列化时忽略值为null的

## Golang

### encoding/json

* 序列化：byte[],err := json.Marshal(对象)，只编码导出成员。缩进：MarshalIndent(data, "", "\t")
* 反序列化：var m Message; err := json.Unmarshal(byte[], &m)。如果不知道结构可以设为map[string]any或json.RawMessage（延迟反序列化）
* 自定义JSON键名：使用struct的Tag，在字段后加`反引号json:"自定义名"反引号`，用"-"表示忽略；如果不指定，则序列化时大写（因为只会序列化公开的），反序列化时大小写不敏感。omitempty当值为零值时忽略
* 序列化进实现了Writer的对象：err=json.NewEncoder(w).Encode(o)。读取实现了Reader的对象反序列化：NewDecoder(r).Decode(&o)
* Decoder还允许反序列化不在JSON数组中的对象流（"{}{}"），用for dec.More(){doc.Decode()}每次反序列化一个
* 高性能第三方库：bytedance/sonic
* 不反序列化，用某种JSONPATH直接取值：github.com/tidwall/gjson
