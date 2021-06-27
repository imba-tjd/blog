---
title: 反射、特性和动态类型
category: dotnet
---

TODO：
内存泄漏：https://zhuanlan.zhihu.com/p/269299903


## 序列化

```c#
[DataContract(Name="error"), Serializable]
public class Error
{
    [DataMember(Name="code")]
    public string Code{ get; set; }
    [DataMember]
    public string Message{ get; set; }
    [NonSerialized]
    public string others; // 逆序列化时会为null
}

var serializer = new DataContractJsonSerializer(typeof(Error));
var result = (T)serializer.ReadObject(stream); // 或用as；不能用string
```

不学二进制序列化，Core1的时候曾删掉了，也不安全。不学XML序列化，现在没人用XML。
