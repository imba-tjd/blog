---
title: .NET 与数据库
category: dotnet
---

## EF Core

### 安装和CLI

* 生成的Migrations文件夹和数据库里留下的那个表都不要删
* 迁移可能很危险，可能造成数据丢失。比如可能真的想要删一个表，但也可能是误判，或者可能本地成功，线上出问题；还有现在用git，而连接到的数据库可不会自动改变；再有就是可能忘了迁移、
* VS中可以直接右键Update Model from Database和Generate Database from Medel

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design # VS中用.Tools
dotnet add package Microsoft.EntityFrameworkCore.SQLite # 用于开发环境
# 还有InMemory及MySql.Data.EntityFrameworkCore；以及另外两个Identity和Diagnostics包

dotnet tool install --global dotnet-ef # 用于“迁移”和查询DbContext信息
dotnet tool install --global dotnet-aspnet-codegenerator # 脚手架，依赖下面的Web和SqlServer包，否则三者都不用装
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

dotnet ef migrations add XXX # 尝试将当前Model和Context代码与数据库进行比较，生成设计结果和迁移脚本
dotnet ef database update # 运行迁移脚本，会自动修改数据库

Scaffold-DbContext <connectionstring> Microsoft.EntityFrameworkCore.Sqlserver -OutputDir Models -Context CustomerContext -DataAnnotations 可以反向从已经创建好的数据库中获取信息创建Model和上下文
```

### DbContext连接

* 加载导航属性的方式有三种：Eager loading进行查询的时候就加载导航属性的相关数据、Explicit loading查询后某一时刻通过代码显式地从数据库加载相关数据、Lazy loading仅当访问导航属性时，相关数据才会从数据库进行加载（由EF Core自动进行）
* 最简单的懒加载的方式是安装Microsoft.EntityFrameworkCore.Proxies，在DbContextOptionsBuilder中调用UseLazyLoadingProxies，再用virtual修饰想启用的导航属性
* 可以重写OnModelCreating方法：`builder.Entity<T>.HasData`可以添加种子数据；Property(x=>x.Name).IsRequired().HasMaxLength(10)可以代替标签进行限制
* 手动建立也可用ASP那样的DBContext的构造函数，需传DbContextOptionsBuilder的Options属性

```c#
class MyDBContext : DBContext {
    ...
    protected override void OnConfiguring(DbContextOptionsBuilder op) {
        op.UseSqlServer(@"connectionString");
    }
}

// appsettings.json
"ConnectionStrings": {
    "CustomerContext": "DataSource=Customers.db"
}

// Startup
// 还可在ctor中注入IWebHostEnvironment，用env.IsDevelopment()在开发环境和生产环境用不同的数据库
// 用AddDbContextPool可重用DbContext
var connectionString = builder.Configuration.GetConnectionString("CustomerContext");
builder.Services.AddDbContext<CustomerDbContext>(op => op.UseSqlServer(connectionString)/UseInMemoryDatabase("name"));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();
if (app.Environment.IsDevelopment()) app.UseMigrationsEndPoint();

// Data/CustomerDbContext.cs
using Microsoft.EntityFrameworkCore;
using RazorPagesContacts.Models;
namespace RazorPagesContacts.Data {
    public class CustomerDbContext : DbContext {
        public CustomerDbContext(DbContextOptions<CustomerDbContext> options) : base(options){ }
        public DbSet<Customer> Customers { get; set; } // 一个DbSet通常与一个表对应

```

### Model

* 两个Model之间的引用（包括`List<T>`）称作导航属性，会自动创建外键；如果另一个类没有ID，仍会有“卷影属性”

```c#
// Models/Customer.cs；与EF在架构上没有直接关系，除非用了标签限制
using System.ComponentModel.DataAnnotations; //还有个Schema命名空间，不知道区别
namespace RazorPagesContacts.Models {
    public class Customer {
        // 此时可以启用C#8的可空引用类型，迁移时会决定对应的数据库字段是否可空
        [Key] // 如果是以Id结尾就可以不加；类型也可以是Guid
        public int Id { get; set; }

        [Required, StringLength(10)]
        public string Name { get; set; }

        [Display(Name = "Your BirthDate")]
        [DataType(DataType.Date)] // 还可以是Password、Time、Url等
        public DateTime BirthDate { get; set; }

        [Column(TypeName = "decimal(18, 2)")]
        public decimal Money { get; set; }

// [Compare(nameof(Password), ErrorMessage="xxx"] ConfirmPassword
}}
```

### 使用

* 查询可使用Linq，但Find效率更高；增删改都要记得保存，查可用AsNoTracking提高性能

```c#
using (var context = new MyDBContext()) {
    var models = MyDBContext.MyModels
        .Where(b => b.Id == "id")
        .ToList();
    context.MyModels.Add(new MyModel()); // 好像也可直接context.Add，会自动识别
    context.Attach(model).State=EntityState.Modified; // 修改后使用，但好像如果是跟踪的就不用显式指定了
    context.SaveChanges(); // 可能出现DbUpdateConcurrencyException
}

```

### ASP中的使用

* 也可以建立IRepository，一般每个对应一张表，注入DbContext封装成业务上的Async方法；要用services.Addxxx注册

```c#
// Page/Customers.cshtml.cs
using Microsoft.EntityFrameworkCore;
using RazorPagesContacts.Models;
...
private readonly RazorPagesContacts.Data.CustomerDbContext _context; // DI略
...
[BindProperty] public Customer Customer { get; set; } // BP表示能被修改
OnGet: Customers = _context.Customers.ToListAsync();
OnPost:_context.Customers.Add(Customer); await _context.SaveChangesAsync();
OnPostDeleteAsync: await _context.Customers.FindAsync(id); if ... Remove ...
```

### 3.0的变化

* 3.0之后，如果代码不能在服务端执行（不能翻译成SQL），会直接报错（除了最后一个Select）；之前是放到本地执行的。可以显式用ToList或者AsEnumerable
* 3.0之后，删除主体时会立刻删除依赖实体，之前SaveChanges时才会操作
* FromSqlRaw、ExecuteSqlRaw、FromSqlInterpolated、ExecuteSqlInterpolated，以及两个Async方法可以手动执行SQL语句，不过需要少用，因为这还是和数据库强耦合的

## [Dapper](https://github.com/StackExchange/Dapper)

* 给IDbConnection加了几个扩展方法，完全兼容和基于ADO
* 支持把单次查询映射到多个对象上，见文档的Multi Mapping
* 字符串默认期待unicode的，如果用varchar需要额外处理，比较麻烦不记录
* DapperAOT：还在早期，且很久没更新了
* Dapper.SqlBuilder：类似于Linq的方法，但只是用于生成SQL字符串，还不如自己手写
* Dapper.Contrib：又给connection添加了几个扩展方法，但看起来也就Insert()和Update()稍微有一点用，Get只支持按id搜索，要不就GetAll了，还可能需要给类加Attributes
* Dapper.Rainbow：需要继承抽象类，表面上类似于ORM不用写SQL，但添加的方法太少，估计必须加载所有数据
* FastCrud：第三方基于Dapper，类似于Linq的语法

```c#
// SELECT：Model无需任何处理，字段名称必须和SELECT的相同，必要时SQL中用AS指定，未SELECT到的类中的属性会为默认值
IEnumerable<Person> people = cnn.Query<Person>("SELECT ... name = @Name", new { Name = name });
var people2 = cnn.Query(...).AsList(); // 不加<T>返回Dynamic，类似于SELECT出来的匿名对象的数组，AsList()不会额外分配一遍
// 还有QueryFirst[OrDefault]()和QuerySingle()
// 参数：前缀必须用@，但对于数字和bool也支持{=xxx}且效率更高；支持in运算符：in @Names, new {Names=new[] {...}}

// DML，还支持存储过程
List<Person> people = ...;
cnn.Execute("INSERT INTO people(name, phone) VALUES (@Name, @Phone)", people); // 传集合自动插入多个

var dbParams = new DynamicParameters();
dbParams.AddDynamicParams(dic);
db.Query(query, dbParams);
```

## ADO.NET

* IDb前缀是通用接口，各个数据库也有自己的实现：Connection、Command、DataReader（类似于数据源Stream）
* 连接字符串的DataSource支持魔值`|DataDirectory|`，winform下表示bin/debug等
* DbProviderFactory：允许在多个数据库Provider之间切换，基本就是IDb接口的应用

```c#
string cnnstr = System.Configuration.ConfigurationManager.ConnectionStrings["连接字符串名称"].connectionString;
using var cnn = new SqliteConnection(cnnstr);
cnn.Open(); // 没Open时也可以创建Command，读取数据就必须Open了

using var cmd = cnn.CreateCommand(); // 或new SqliteCommand(sqltext,cnn)
cmd.CommandText ="INSERT INTO user (name) VALUES (@name)";
cmd.Parameters.AddWithValue("@name", name).Size = 30; // 添加参数并设置截断长度，这诡异的写法居然没问题。一般还是给AddWithValue的返回值赋一个变量再进一步设置

cmd.ExecuteNonQuery(); // 执行DML，返回被影响的行数
cmd.ExecuteScalar().ToString(); // 以object类型返回结果表第一行第一列的值，一般用于执行查询单值Select命令，无值时为null

var reader = cmd.ExecuteReader(); // 执行Select，一般没必要dispose()
while (reader.Read()) { // 读完时返回false
    string name = reader.GetString(0)/GetFieldValue<string>(0);
    int length = reader.GetInt32(1);
    object[] line = new object[reader.FieldCount]; reader.GetValues(line);
    reader[ndx/key]; // object类型
}

using var tran = cnn.BeginTransaction();
// 创建cmd，最好不要提前创建
tran.Commit()/Rollback();
```

### IDbConnection

* ConnectionStringBuilder：设置好属性后用ConnectionString获取组合后的连接字符串
* ConnectionString：获取设置连接字符串，一般在Connection的构造函数中设置
* ConnectionTimeOut：0为无限，超时抛异常
* Database：数据库名称
* State：Open、Connecting、Closed等
* IDb的没有DataSource属性
* ChangeDatabase()
* 连接字符串
  * https://www.connectionstrings.com
  * Server和Data Source同义，表示本机可用一个点，Database和Initial Catalog同义
  * OLEDB需要设置Provider，ODBC需要设置Driver和DBQ

### IDbCommand

* CommandText：获取或设置要执行的SQL命令/储存过程/数据表名称
* CommandType：Text（默认，SQLite只支持它）、StoredProcedure（使用时CommandText设为SP名，再添加参数即可，就像函数调用一样）、TableDirect
* Parameters：SQL命令参数集合
* Cancel()

### IDataReader

* 一个向前只读的记录指针
* GetSchemaTable()：获得元数据
* NextResult()：如果CommandText有多条SQL语句（批处理），此函数会继续执行下一条，指向下一个结果集，之后自己继续用Read()；Dispose时会自动执行完
* GetName(index)：获得列名；GetOrdinal()：根据列名返回它的列数
* GetDataTypeName(index)：输入列数，返回该列的类型名；转换成IDataRecord后有GetFieldType()返回Type对象

### DataSet

* 断开模式（ADO.NET独有）：数据库——Connection——DataAdapter——DataSet，然后断开连接。之后的操作都是操作DataSet，完成后统一写回数据库。数据集DataSet相当于一个内存数据库，有DataTables、DataRow、Linq to DataSet、CommandBuilder、DataAdapter等概念。DataAdapter能写回数据库，感觉不如直接用EF了，MS的SQLite不支持

```c#
var adapter = new SqlDataAdapter(sqlstr, cnn);
var ds = new DataSet();
adapter.Fill(ds, "Customers");
DataTable orders = ds.Tables["SalesOrderHeader"];
var query = from order in orders
    where order.Field<bool>("OnlineOrderFlag") == true
    select new {
        SalesOrderID = order.Field<int>("SalesOrderID"),
        SalesOrderNumber = order.Field<string>("SalesOrderNumber")
    };
foreach (var onlineOrder in query)
    WriteLine("Order ID: {0}", onlineOrder.SalesOrderID);
dataGridView1.DataSource = dt.DefaultView;

// 不使用Adapter，但仍表现为“内存表”，功能复杂
new DataTable().Load(reader);
for(int i=0;i<tb.Rows.Count;i++)
    tb.Rows[i]["Col1"]
```

### [Microsoft.Data.Sqlite](https://docs.microsoft.com/zh-cn/dotnet/standard/data/sqlite)

* 支持cnn.CreateFunction()注册由C#实现的能在SQL语句中使用的自定义函数
* con.DefaultTimeout默认30秒，设为0会永远等待
* 参数的DbType只有Sqlite的原生四种
* 不支持Async方法
* 连接字符串：`URI=file:db.sqlite`，这种情况下路径分隔符必须用斜杠；要不就也用`DataSource=...`
* EFCore上的默认启用了WAL
* 只支持命名参数，可用`: @ $`作为前缀
* 大型Blob有专门的方法读写，目前不学
* Win10自带winsqlite3.dll，目前版本3.34，可安装Microsoft.Data.Sqlite.Core和SQLitePCLRaw.bundle_winsqlite3来使用

### System.Data.SQLite

* 不是内置库，不是微软出的，是SQLite官方出的
* 一般用.Core的包，与普通版相比没有Linq和EF6。MSIL的包缺少二进制依赖，无法直接使用
* 无参构造函数无法使用

### [Microsoft.Data.SqlClient](https://docs.microsoft.com/zh-cn/sql/connect/ado-net/sql)

* 取代System.Data.SqlClient

## Oracle

* FX用Oracle.ManagedDataAccess包，Core用.Core包，还有VS和VSC的扩展。最好不要用System.Data.OracleClient，它也只能在装了Oracle实例的环境中用
* 连接字符串：Server=xxx;User Id=xxx;Password=xxx

## 其它项目

* [LINQ to DB](https://linq2db.github.io/)：比dapper的star少很多，但contributor有一半，而且仍然活着，所以也还可以看看。比Dapper重。Wiki有内容
* SSDT：https://www.youtube.com/watch?v=ijDcHGxyqE4
* SqlSugar：国产的ORM，号称简单，star数还算可以但贡献者极少；FreeSql：国产，支持的数据库多一点
* Linq to SQL：已经不维护了，且只支持SQL Server
* [AutoMapper](https://docs.automapper.org/en/latest/Getting-started.html)：用于类与类之间的映射，或者数据库模型与实体之间的映射。https://zhuanlan.zhihu.com/p/89550593 https://zhuanlan.zhihu.com/p/136602715
* mysql-connector-net：官方ADO驱动，支持X协议
* [sqlite-net](https://github.com/praeclarum/sqlite-net)：不遵循ADO规范，更易用也更专用，内置简单ORM。也依赖SQLitePCLRaw，但不支持winsqlite3
* https://github.com/CollaboratingPlatypus/PetaPoco 轻量级ORM

## 参考

* https://zhuanlan.zhihu.com/p/67492809
* https://zhuanlan.zhihu.com/p/35652195
* 浙江省高等学校精品在线开放课程共享平台——Web应用程序设计（.NET）
* https://www.youtube.com/channel/UC-ptWR16ITQyYOglXyQmpzw
* https://zhuanlan.zhihu.com/p/36608131

### TODO

* https://www.entityframeworktutorial.net/
* https://www.youtube.com/watch?v=qkJ9keBmQWo
* https://www.devart.com/dotconnect/ ADO.NET Provider，可以玩玩看
* https://www.youtube.com/watch?list=PLLWMQd6PeGY3b89Ni7xsNZddi9wD5Esv2
