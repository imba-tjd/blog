---
title: ".NET 与数据库"
---

EF Core
-------

```
// appsettings.json
  "ConnectionStrings": { // 对于sqlite，只要指定名字就好，数据库会自动创建
    "CustomerContext": "Data Source=Customers.db"
  }

// Startup
services.AddDbContext<CustomerDbContext>(op=>op.UseInMemoryDatabase("name"));
op=>op.UseSqlite(Configuration.GetConnectionString("CustomerContext");

// Models/Customer.cs；与EF在架构上没有直接关系，实际如果用了标签来限制就有关
using System.ComponentModel.DataAnnotations; //还有个Schema命名空间，不知道区别
namespace RazorPagesContacts.Models {
    public class Customer {
        // 此时可以启用C#8的可空引用类型，迁移时会决定对应的数据库字段是否可空
        [Key] // 如果是以Id结尾就可以不加；类型也可以是Guid
        public int Id { get; set; }

        [Required, StringLength(10)]
        public string Name { get; set; }

        [Display(Name = "Your BirthDate")]
        [DataType(DataType.Date)] // Password
        public DateTime BirthDate { get; set; }

        [Column(TypeName = "decimal(18, 2)")]
        public decimal Money { get; set; }

// [Compare(nameof(Password), ErrorMessage="xxx"] ConfirmPassword

// 可以有ICollection<T>，则是一对多的模型；迁移时如果另一个类有CustomerId，会自动创建外键；如果没有，仍然会有“卷影属性”
}}

// Data/CustomerDbContext.cs；对应一个数据库的连接
using Microsoft.EntityFrameworkCore;
using RazorPagesContacts.Models;
namespace RazorPagesContacts.Data {
    public class CustomerDbContext : DbContext {
        public CustomerDbContext(DbContextOptions<CustomerDbContext> options) : base(options){ }
        // 一个DbSet通常与一个表对应
        public DbSet<Customer> Customers { get; set; }

        // protected override void OnConfiguring(DbContextOptionsBuilder options){ options.UseSqlite(...) } // 在ASP.NET Core中不需要这样做，不在才要

// 可以重写OnModelCreating方法：builder.Entity<T>.HasData可以添加种子数据；Property(x=>x.Name).IsRequired().HasMaxLength(10)可以进行限制
}}

// 一般还需要建立一个IRepository和实现它，每个对应一个表。注入DbContext，有一些Async方法，对应行的增删改查。还需要用services.Addxxx注册

// 使用：Page/Customers.cshtml.cs；此处许多逻辑最好移到Repository里
using Microsoft.EntityFrameworkCore;
using RazorPagesContacts.Models;
...
private readonly RazorPagesContacts.Data.CustomerDbContext _context; // DI略
...
OnGet: Customers = _context.Customers.ToListAsync();
[BindProperty] public Customer Customer { get; set; } // BP表示能被修改
OnPost:_context.Customers.Add(Customer); await _context.SaveChangesAsync(); // 也可以直接_context.Add，ef会自动识别
OnPostDeleteAsync: await _context.Customers.FindAsync(id); if ... Remove ...
//查询还可使用Linq，但Find效率更高；增删改都要记得保存，查可用AsNoTracking提高性能

更改：_context.Attach(Customer).State=EntityState.Modified;但好像如果是跟踪的就不用显式指定了
try{保存}catch(DbUpdateConcurrencyException){if不存在，返回NotFound，否则throw}
Eager加载：使用Include方法；懒加载：需要...Proxies包，AddDbContext的选项中添加UseLazyLoadingProxies()，最后把需要懒加载的属性标记为virtual
重用DbContext：把AddDbcontext改成AddDbContextPool

// CLI
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SQLite
dotnet tool install --global dotnet-ef // 用于“迁移”
dotnet tool install --global dotnet-aspnet-codegenerator // 脚手架，依赖下面的Web和SqlServer包，否则不用装；使用方法具体参见文档或用-h
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
还存在InMemory、tools包；以及另外两个Identity和Diagnostics包

dotnet ef migrations add XXX 会尝试将当前Model和Context代码与数据库进行比较，生成设计结果和迁移脚本
dotnet ef database update 运行迁移脚本，会自动修改数据库
生成的Migrations文件夹和数据库里留下的那个表都不要删。
Scaffold-DbContext <connectionstring> Microsoft.EntityFrameworkCore.Sqlserver -OutputDir Models -Context CustomerContext -DataAnnotations 可以反向从已经创建好的数据库中获取信息创建Model和上下文
VS中可以直接右键Update Model from Database和Generate Database from Medel
```

* 迁移可能很危险，可能造成数据丢失。比如我可能真的想要删一个表，但也可能是误判，或者可能本地成功，线上出问题；还有现在用git，而连接到的数据库可不会自动改变；再有就是可能忘了迁移

### 3.0的变化

* 3.0之后，如果代码不能在服务端执行（不能翻译成SQL），会直接报错（除了最后一个Select）；之前是放到本地执行的。可以显式用ToList或者AsEnumerable。
* 3.0之后，删除主体时会立刻删除依赖实体，之前SaveChanges时才会操作
* FromSqlRaw、ExecuteSqlRaw、FromSqlInterpolated、ExecuteSqlInterpolated，以及两个Async方法可以手动执行SQL语句，不过需要少用，因为这还是和数据库强耦合的

* 未读：https://zhuanlan.zhihu.com/p/35652195、https://www.bilibili.com/video/av34462368、缓存相关
* https://docs.microsoft.com/zh-cn/ef/core/

[Dapper](https://github.com/StackExchange/Dapper)
-------------------------------------------------

* https://www.youtube.com/watch?v=QVkpzuiiVtw SQL Transactions in C# using Dapper
* https://www.youtube.com/watch?v=Et2khGnrIqc How to connect C# to SQL (the easy way)

```
IEnumerable<Person> Fun(string name) =>
connection.Query<Person>($"select * from people where Name = '{name}'");
connection.Query<Person>("dbo.People_GetByName @Name", new { Name=name });

List<Person> people = ...;
people.Add(...);
connection.Execute("dbo.People_Insert @Name, @Phone", people);
```

LINQ to DB
----------

* https://linq2db.github.io/
* 比dapper的star少很多，但contributor有一半，而且仍然活着，所以也还可以看看；但现在刚好是3.0预览版，等正式版再说

SSDT
----

* https://www.youtube.com/watch?v=ijDcHGxyqE4

Microsoft.Data.SqlClient
------------------------

* https://github.com/dotnet/SqlClient
* 取代下面那个，不过应该大部分差不多。还没看的
* Linq to SQL微软已经不维护了，且只支持SQL Server

ADO.NET（System.Data.SqlClient）
--------------------------------

> 浙江省高等学校精品在线开放课程共享平台——Web应用程序设计（.NET）
> 未读：https://zhuanlan.zhihu.com/p/67492809

### .NET Framework数据提供程序

* SQL Server、OLE DB、ODBC、Oracle分别有System.Data下的命名空间，以下四种类的前面有不同的前缀
* Connection
* Command
* DataReader
* DataAdapter
* 它们都继承IDb开头的接口

### 连接模式

* 读取：数据库——Connection——Command——DataReader——页面
* 写入：页面——Command——Connection——数据库

### 断开模式（ADO.NET独有）

* 数据库——Connection——DataAdapter——DataSet，然后断开连接
* 之后的操作都是操作DataSet
* 完成后统一写回数据库

### SqlConnection

#### 属性

* ConnectionString：连接字符串
* ConnectionTimeOut：0表示不限制；超过时间产生异常
* Database：数据库名称
* DataSource：获取数据源的完整路径和文件名；如果是SQL Server，获取数据库名称
* State：ConnectionState的枚举

### ConnectionString参数

* 来源：https://www.connectionstrings.com
* Privider：用于设置数据源的OLE DB驱动程序。Access为`Microsoft.Jet.OLEDB.4.0`，SQL Server 6.5或之前的版本为`SQLOLEDB`
* Data Source或Server：需连接的数据库服务器名称；如果使用文件，需要转义；localhost表示本机的SQL Server默认服务器。
* Initial Catalog或Database：数据库名称
* AttachDBFilename：数据库的路径和文件名
* User ID或uid：账户
* Password或pwd：密码
* Integrated Security：true和SSPI表示使用Windows集成身份验证，false为不使用
* Connection Timeout：单位为秒，超时返回失败信息

可以使用SqlConnectionStringBuilder，设置以上属性；它的ConnectionString属性就是结果。

使用Web.config/App.config中的configuration节点可以全站统一配置。且可动态加载，修改后只需重启程序就可以变化，无需重新编译。

```
 <connectionStrings>
 <add name = "连接字符串名称" connectionString="连接字符串" providerName="System.Data.SqlClient" />
 </connectionStrings>
```

```
Using System.Configuration;
string strCnn = ConfigurationManager.ConnectionStrings["连接字符串名称"].connectionString;
```

###  常用方法

* Open
* Close
* BeginTransaction：开始一个数据库事物，可以制定事物名称和隔离级别
* ChangeDatabase：在打开连接的状态下，更改数据库
* CreateCommand：创建并返回与SqlConnection有关的SqlCommand对象
* Dispose

### SqlCommand

#### 属性

* CommandText：获取或设置要对数据源执行的SQL命令/储存过程/数据表名称
* CommandType：枚举，获取或设置命令类型，Text/StoredProcedure/TableDirect分别对应那三项
* Connection：获取或设置所使用的数据连接
* Parameters：SQL命令参数集合
* Transaction：设置所属的事物

#### 方法

* Cancel
* CreateParameter
* ExecuteNonQuery：只能执行Insert、Update和Delete，返回被影响的行数
* ExecuteReader：用于执行返回多条记录的Select命令，返回SqlDataReader对象
* ExecuteScalar：以object类型返回结果表第一行第一列的值，一般用于执行查询单值Select命令
* ExecuteXMLReader：返回XMLReader对象

#### SqlParameter

```
string UserID = textBox1.Trim();
SqlCommand cmd = new SqlCommand();
cmd.Connection = cnn;
cmd.CommandText = "select * from UserInfo where UserID = @UserID"

SqlParameter sp = new SqlParameter();
sp.ParameterName="@UserID";
sp.SqlDbType = SqlDbType.VarChar;
sp.Size = 20;
sp.Direction = ParameterDirection.Input;
sp.Value=UserID;
cmd.Parameters.Add(sp);

// 或者直接一条语句就行？
cmd.Parameters.AddWithValue("@UserID",UserID);
```

### 储存过程

```
string UserID = textBox1.Trim();
SqlCommand cmd = new SqlCommand();
cmd.Connection = cnn;

cmd.CommandText = "LoginProc";
cmd.CommandType = CommandType.StoredProcedure;
cmd.Parameters.AddWithValue("@UserID",UserID);
```

### SqlDataReader

* 一个向前只读的记录指针

#### 属性

* FieldCount：获取一行数据中的字段数
* IsClosed
* HasRows：是否包含数据

#### 方法

* Close
* Read：返回false表示读完了
* NextResult：当返回多个结果集时，指向下一个结果集。之后如果要读取仍然要用Read方法
* GetValue(int)：返回当前行指定索引列的值，object类型；如果知道类型，可以用GetInt32、GetString等方法
* GetName(int)：获得列名
* GetValues(object[])：会把当前行所有数据保存到一个数组里。可以根据FieldCount设定数组长度
* GetDataTypeName(int)：输入列索引，返回该列的类型
* IsDBNull(int)：输入当前行的列索引，判断是否为空

### 事务

* 数据库事务
* ADO.NET事务
* ASP.NET事务：@Page指令中加一个Transaction="Required"。
* COM+事务

### 数据集DataSet

* 相当于一个数据库
* 存放在内存中，可以使用Session保存
* 可以动态生成（非类型化数据集）；也可以先添加/编写好xsd文件，使用时就只需要添加数据了（类型化数据集）
* 每一行仍然需要用表的NewRow方法创建，但名字好像不同

#### 属性和方法

* DataSetName
* Tables：数据表的集合，使用索引器获取表，Add方法添加表
* Clear方法
* Copy方法

### DataTable

* Columns：所有字段的集合
* DataSet：DataTable所属的DataSet对象
* DefaultView：获取与数据表相关的DataView对象
* PrimarilyKey：获取或设置数据表的主键
* Rows：所有行的集合
* TableName：数据表名
* Clear方法
* NewRow方法

### DataColumn

* AllowDBNull
* Caption：字段标题。若未指定，与字段名相同
* ColumnName：字段名
* DataType = System.Type.GetType("数据类型")，不能直接设置？
* DefaultValue
* ReadOnly

### DataRow

* DataRow 对象名 = DataTable对象.NewRow();，不能直接new
* 使用索引器获取/设置字段的值
* RowState属性，属于DataRowState枚举型，分别为Add、Delete、Detached、Modified、Unchanged
* AcceptChanges方法
* BeginEdit方法：修改行之前要调用
* CancelEdit方法
* EndEdit方法
* Delete方法

### DataView

* DataView 对象名 = new DataView(数据表对象);
* string RowFilter：相当于Select，内容就是Select后的sql语句
* string Sort：相当于order by

### DataAdapter

* **用于DataSet跟和数据库交互**
* New的时候可以接受SqlCommand或接受Sql语句和SqlConnection/连接字符串；如果是后者，使用DataAdapter的方法时会自动连接数据库，好了以后自动关闭
* Fill方法（相当于Select \* from*TableName*）和SelectCommand方法，接受DataSet对象和数据库名，把内容复制到DataSet中
* InsertCommand、UpdateCommand、DeleteCommand方法：逐行检查RowState属性，进行写入数据库操作
* Update方法可以接受表，把数据从内存中写回数据库。因为DataAdapter也是以表进行操作的

### SqlCommandBuilder

* SqlCommandBuilder sb = new SqlCommandBuilder(DataAdapter);
* 可以根据SelectCommand属性自动设置其他三个Command属性


