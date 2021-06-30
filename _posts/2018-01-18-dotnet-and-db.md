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
"ConnectionStrings": { // 对于sqlite，只要指定名字就好，数据库会自动创建
    "CustomerContext": "Data Source=Customers.db"
}

// Startup
services.AddDbContext<CustomerDbContext>(op=>op.UseInMemoryDatabase("name"));
op=>op.UseSqlite(Configuration.GetConnectionString("CustomerContext");
// 还可在ctor中注入IWebHostEnvironment，用env.IsDevelopment()在开发环境和生产环境用不同的数据库
// 用AddDbContextPool可重用DbContext

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

## Microsoft.Data.Sqlite

* 能通过cnn.CreateFunction()方便地创建自定义函数，因为那些实际上是用当前的编程语言执行的
* cmd.CommandTimeout默认30秒
* 参数的DbType只有Sqlite的原生四种
* 不要用Async方法，不支持
* 连接字符串：`"DataSource=db.sqlite/:memory:;Mode=ReadWriteCreate（默）/ReadWrite/ReadOnly"`，SqliteConnectionStringBuilder略；不加参数会创建临时磁盘数据库，在连接关闭时删除；路径为相对CWD的；还支持url但应该是只读的；默认如果文件不存在会自动创建但无法自创文件夹
* 事务默认隔离级别为可序列化，设置`Cache=Shared`再更改事务隔离级别才允许读取未提交内容
* 支持cnn.BackupDatabase()，目前只有阻塞的
* 默认启用了WAL

```c#
// 排序规则默认支持 RTRIM忽略尾随空格、NOCASE英文字符不区分大小写、BINARY二进制比较 三种。可自定义支持Unicode的排序：
cnn.CreateCollation("NOCASE", (x, y) => string.Compare(x, y, ignoreCase: true));
```

## [Microsoft.Data.SqlClient](https://docs.microsoft.com/zh-cn/sql/connect/ado-net/sql)

TODO。取代System.Data.SqlClient。

## [Dapper](https://github.com/StackExchange/Dapper)

* https://www.youtube.com/watch?v=QVkpzuiiVtw SQL Transactions in C# using Dapper
* 扩展了IDbConnection和IDataReader，因此连接部分和普通的ADO.NET没区别
* 数据库字段不区分大小写；查询参数必须和匿名对象名称相同，选取结果的名称也必须和映射类相同，否则参数需用DynamicParameters，选取结果在SQL语句中用as
* 支持Ansi字符串（varchar），用new DbString；`reader.GetRowParser<T>(typeof(class))`支持同一列不同类型的读取；`new DataTable().Load(reader)`
* Sqlite测试：当连接未打开时，创建表不报错，插入时才报错。当select xxx忘写from时，报的错是`no such column xxx`
* DapperAOT，还在早期

```c#
// 强类型查询，这样做Model类无需任何注释绑定；如果类中存在数据库没有的字段，会保留默认值，不会抛异常
IEnumerable<Person> = cnn.Query<Person>("SELECT ... name = @Name", new { Name = name }); // 还可 in @Names, new {Names=new[] {...,}}
// 还有QueryFirst[OrDefault]()和QuerySingle()；不加<T>返回的是dynamic，差不多是SELECT出来的匿名对象数组，必要时需强转；还支持{=XXX}的bool和数字字面量替换；Query还支持多个泛型参数，见文档的Multi Mapping

// 插入等
List<Person> people = ...;
cnn.Execute("INSERT INTO people(name, phone) VALUES (@Name, @Phone)", people); // 自动插入多个
cnn.Execute("dbo.People_Insert @Name, @Phone", people); // 储存过程自动识别，也可指定commandType位置参数；输出参数要用动态参数

// 查询语句含有多个SQL语句
using (var multi = cnn.QueryMultiple(sql)) {
    var invoice = multi.Read<Invoice>().First();
    var invoiceItems = multi.Read<InvoiceItem>().AsList(); // 与ToList相比不会重新分配空间
}

// Dapper.Contrib和.Contrib.Extensions。就只有这一点东西，没法根据普通字段查询
[Table("students")] // 表名与类名相同可不加
public class Student{
    [Key] public int Id {get; set;} // 用于自增字段，名称完全等于Id也可不加，使用Insert()会忽略对象的值；非自增用ExplicitKey
    // [Write(false)]、[Computed]
}
cnn.Insert(stus) // 插入一或多条数据，表要自己建好；Update、Delete、DeleteAll略
cnn.Get<Student>(1) // 根据主键查找数据
cnn.GetAll<Student>();
```

## 其它项目

* [LINQ to DB](https://linq2db.github.io/)：比dapper的star少很多，但contributor有一半，而且仍然活着，所以也还可以看看。比Dapper重
* SSDT：https://www.youtube.com/watch?v=ijDcHGxyqE4
* SqlSugar：国产的ORM，号称简单，star数还算可以但贡献者极少；FreeSql：国产，支持的数据库多一点，产生时间不长，但有人说已经比前者更好了
* Linq to SQL：已经不维护了，且只支持SQL Server

## ADO.NET

SQL Server、OLE DB、ODBC、Oracle分别有System.Data下的命名空间，以下四种类的前面有不同的前缀，也可用它们实现了的IDb前缀的通用接口：

* Connection
* Command
* DataReader：数据源Stream

连接模式：读取：数据库——Connection——Command——DataReader——页面。写入：页面——Command——Connection——数据库

断开模式（ADO.NET独有）：数据库——Connection——DataAdapter——DataSet，然后断开连接。之后的操作都是操作DataSet，完成后统一写回数据库。数据集DataSet相当于一个内存数据库，有DataTables、DataRow、Linq to DataSet、CommandBuilder、DataAdapter等概念。但是感觉不如用EF。

### IDbConnection

* 具体的构造函数接受具体的ConnectionStringBuilder设置好属性后用ConnectionString获取连接字符串，也可用https://www.connectionstrings.com
* ConnectionString：可获取或设置
* ConnectionTimeOut：0为无限，超时抛异常
* Database：数据库名称
* State：Open、Connecting、Closed等
* IDb的没有DataSource属性
* ChangeDatabase()
* CreateCommand()、BeginTransaction()
* 还有个DbProviderFactory用于从app.config中获得DbConnection，不过没什么必要

```c#
app.config中的configuration节点可以全站统一配置，且修改后只需重启程序就可以变化，无需重新编译。但Core不使用此文件。
<connectionStrings>
  <add name = "连接字符串名称" connectionString="连接字符串" providerName="System.Data.SqlClient" />
</connectionStrings>

string cnnstr = System.Configuration.ConfigurationManager.ConnectionStrings["连接字符串名称"].connectionString;
using var cnn = new SqliteConnection(cnnstr);
cnn.Open(); // Command的构建在连接没Open时也可以，读取数据就必须Open了
```

### IDbCommand

* CommandText：获取或设置要执行的SQL命令/储存过程/数据表名称
* CommandType：Text（默认）、StoredProcedure、TableDirect。Sqlite只支持Text
* Parameters：SQL命令参数集合
* CommandTimeout
* Cancel()
* ExecuteNonQuery()：只能执行Insert、Update和Delete，返回被影响的行数
* ExecuteReader()：用于执行返回多条记录的Select命令，返回SqlDataReader对象
* ExecuteScalar()：以object类型返回结果表第一行第一列的值，一般用于执行查询单值Select命令

```c#
using var cmd = cnn.CreateCommand(); // 或new SqliteCommand(sqltext,cnn)

cmd.CommandText ="INSERT INTO user (name) VALUES (@name)";
cmd.Parameters.AddWithValue("@name", name).Size = 30; // 添加参数并设置截断长度，这诡异的写法居然没问题。一般还是给AddWithValue的返回值赋一个变量再进一步设置
var param=cmd.CreateParameter(); param.ParameterName="@name"; param.Value=name; param.Direction=ParameterDirection.Input; param.Size = 30; cmd.Parameters.Add(param); // 正常写法
cmd.ExecuteNonQuery();

using var tran = cnn.BeginTransaction(); // ADO.NET事务，不是数据库事务？
tran.Commit()/Rollback();
```

### IDataReader

* 一个向前只读的记录指针
* FieldCount：一行数据中的字段数
* HasRows：是否包含数据，一般只在最初的时候使用
* GetSchemaTable()：获得元数据
* NextResult()：如果CommandText有多条SQL语句（批处理），此函数会继续执行下一条，指向下一个结果集，之后自己继续用Read()；Dispose时会自动执行完
* GetValue(index)：返回当前行指定索引列的值，object类型，与直接对reader取索引一样；也可用GetInt32、GetString等方法，有的还有`GetFieldValue<T>()`
* GetName(index)：获得列名；GetOrdinal()：根据列名返回它的索引
* GetValues(object[])：会把当前行所有数据保存到一个数组里。可以根据FieldCount设定数组长度
* GetDataTypeName(index)：输入列索引，返回该列的类型名；SqliteDataReader有`GetFieldType()`返回Type对象
* IsDBNull(index)：输入当前行的列索引，判断是否为空

```c#
cmd.CommandText = "SELECT * FROM DB";
using var reader = cmd.ExecuteReader();
while (reader.Read()) { // 读完时返回false
    string name = reader.GetString(0); // 把第一列当作string读取
    int length = reader.GetInt32(1);
    }
```

## DataSet

```c#
var adapter = new SqlDataAdapter(queryString, cnn);
var ds = new DataSet();
ds.Locale = CultureInfo.InvariantCulture;
adapter.Fill(ds, "Customers");
DataTable orders = ds.Tables["SalesOrderHeader"];
var query = from order in orders
    where order.Field<bool>("OnlineOrderFlag") == true
    select new {
        SalesOrderID = order.Field<int>("SalesOrderID"),
        SalesOrderNumber = order.Field<string>("SalesOrderNumber")
    };
foreach (var onlineOrder in query)
    WriteLine("Order ID: {0} Order number: {1}",
        onlineOrder.SalesOrderID, onlineOrder.SalesOrderNumber);
```

## [AutoMapper](https://docs.automapper.org/en/latest/Getting-started.html)

TODO。用于类与类之间的映射，或者数据库模型与实体之间的映射

* https://zhuanlan.zhihu.com/p/89550593
* https://zhuanlan.zhihu.com/p/136602715

## 参考

* https://zhuanlan.zhihu.com/p/67492809
* https://zhuanlan.zhihu.com/p/35652195
* 浙江省高等学校精品在线开放课程共享平台——Web应用程序设计（.NET）
* https://www.youtube.com/channel/UC-ptWR16ITQyYOglXyQmpzw
* https://zhuanlan.zhihu.com/p/36608131

### TODO

* https://www.entityframeworktutorial.net/
* https://www.youtube.com/watch?v=qkJ9keBmQWo
