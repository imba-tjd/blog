---
title: SQL Server
---

安装
----

* 启用协议：打开SQL Server 配置管理器，网络协议中named pips和TCP/IP启用；命名实例的端口是动态的，如果只有一个，可能为1433，不过最好手动[调成静态的](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/configure-a-server-to-listen-on-a-specific-tcp-port?view=sql-server-2017)
* 打开防火墙端口：`netsh firewall set portopening protocol = TCP port = 1433 name = SQLPort mode = ENABLE scope = SUBNET profile = CURRENT`；其他端口参见[安装文档](https://docs.microsoft.com/zh-cn/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access?view=sql-server-2017)
* Windows系统需要打开网络发现，否则连ping都不让
* 连接服务器名=[协议名加冒号]+计算机名或ip+反斜杠加实例名或逗号加端口（默认1433）；有端口时即使指定实例名也会忽略
* 如果不设置实例名，则默认实例名为MSSQLSERVER但连接时不需也不能用到，命名实例默认为SQLEXPRESS；实例名不区分大小写，实例无法重命名但可以建立别名，不能有除下划线和$以外的特殊字符
* 启用SQL Server Browser服务和UDP 1434端口，再在防火墙中允许Sqlservr.exe入站，则可用实例名访问且可用动态端口；否则无法用实例名访问命名实例（本地计算机名除外，但包括本地ip），但可用静态端口访问
* 如果安装时选择了区分大小写的排序规则，则登录名也将区分大小写
* 实例默认路径：C:\Program Files\Microsoft SQL Server\MSSQL14.MyInstance\
* 修改SQL的身份验证模式：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL14.SQLEXPRESS\MSSQLServer`节点下有个LoginMode的键，把值从1（代表Windows验证模式）修改为2（代表混合验证模式）即可。如果这里没有，到Wow6432Node下面去看看有没有

命令行工具
----------

### [sqlcmd](https://docs.microsoft.com/zh-cn/sql/tools/sqlcmd-utility)

* sqlcmd -S 服务器名称 -U 帐户 -P 密码 -d 数据库
* -Q 查询语句，完成后退出，小写不退出
* -i 脚本名，-o 输出到文件
* -l 超时时间，默认8秒
* -Z 新密码，更改后退出，-z不退出
* -f 指定代码页
* -A：使用专用管理员连接(DAC)登陆，或者用admin:*实例名*来连接；只能使用有限的资源，详情参考[文档](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/diagnostic-connection-for-database-administrators)
* –N –C：对通信加密并信任服务器证书
* -M：在连接到 SQL Server 可用性组或 SQL Server 故障转移群集实例的可用性组侦听程序时，应始终指定此项，能更快检测和连接
* [:]Reset：清空输入缓存，:List：显示输入缓存
* [:]!!：在运行sqlcmd的计算机上执行shell命令
* :On Error[exit | ignore]：在脚本或批处理执行过程中发生错误时要执行的操作
* [:]EXIT[ (statement) ]：把语句的结果作为sqlcmd的返回值
* :Connect：连接到另一个实例同时关闭当前连接
* 环境变量：SQLCMDSERVER、SQLCMDUSER、SQLCMDPASSWORD
* 其他：https://docs.microsoft.com/zh-cn/sql/tools/sqlcmd-utility

### [mssql-cli](https://github.com/dbcli/mssql-cli)

* pip install mssql-cli

### DBCC

Transact-SQL
------------

### 变量

* declare @变量名 变量类型 [, ...]
* select/set @变量名 = 变量值
* select可以一次给多个变量赋值，如果有多个返回值，保存的是最后一个；如果赋值号右边是子查询且没有返回值，变量设为NULL
* set一次只能给一个变量赋值，但有很多选项
* 类型不能为text、ntext、image
* 记得指定长度

### 运算符优先级

> https://docs.microsoft.com/zh-cn/sql/t-sql/language-elements/operator-precedence-transact-sql

### 流程控制

* begin ... end
* if ... else
* if [not] exsits
* case when then
* while ... continue ... break：用到begin ... end
* waitfor [delay|time] '*时间*' ...：delay是倒计时，time是“闹钟”
* goto
* return：保留-1~-99作为系统使用

### 常用命令

* backup
* checkpoint
* dbcc：后面必须加参数
* kill
* print
* raiserror
* restore
* shutdown [with nowait]
* use：切换数据库
* go：用于批处理

### 常用函数

* stdev：标准差
* stdevp：总体标准差
* var：统计变异数
* varp：总体变异数
* sin、cos、tan、cot；asin、acos、atan；degrees、radians
* exp、log、log10、sqrt
* ceiling、floor、round：取近似值函数
* abs、sign
* pi、rand

* * * * *

* char：把ASCII码转换为字符
* lower、upper
* str：把数值型数据转换为字符型数据

* * * * *

* ltrim、rtrim：去空格
* left、right、substring：取字串
* charindex、patindex、soundex、difference
* quotename、replicate、reverse、replace、space、stuff

* * * * *

* cast( Expression as DataType )
* convert( DataType, Expression )

* * * * *

* day、month、year
* dateadd、datediff、datename、datepart、getdate

* * * * *

* readtext/writetext：用于从text、ntext、image中读取/写入数据
* textptr、textvalid

* * * * *

* 自定义函数：create/alter/drop function FunctionName
* (@Parameter...)
* returns DataType
* as begin ..
* return @变量
* end
* 还可以返回一个表（内联表值函数），或返回语句前还有其他的T-SQL语句（多语句表值函数）

数据库设计
----------

### 排序规则

* 服务器级别的只能在建立时选择，建立后无法修改；数据库级别的修改用`alter database db1 collate xxx`，查询用`select name, collation_name from sys.databases`
* LocalDB默认的是Latin1_General，无法显示中文，用nvarchar也不行；正常的默认是Chinese_PRC_CI_AS

### 预定义数据库角色

* db_accessadmin
* db_backupoperator
* db_datareader
* db_datawriter
* db_ddladmin
* db_denydatareader
* db_denydatawriter
* db_owner：拥有全部权限
* db_securityadmin
* public：所有用户都有

### 查看数据库角色

* sp_addrolemember
* sys.database_role_memebers
* sys.database_principals

#### 应用程序角色

* create/alter/drop application role
* sp_setapprole、sp_unsetapprole

其他
----

* 保存有所有数据库的基本信息的数据库：sys.databases
* 字符串前加N表示NVARCHAR，系统自带的部分都是这样的？
* 对象的标准写法是databasename.databaseownername.objectname，默认是当前数据库和登陆的用户，而每个数据库都会有一个叫dbo的schema，相当于全局的吧
* 清除SSMS登陆的用户名和密码：删除%AppData%\Microsoft\SQL Server Management Studio\18.0\SqlStudio.bin
* 启用账户：alter login sa enable；修改密码：alter login sa with password='sa'，启用后要重启服务？
* Select @@ServerName：计算机名
* 控制Windows服务：用net还是sc?
* print函数
* [查看或更改服务器属性](https://docs.microsoft.com/zh-cn/sql/database-engine/configure-windows/view-or-change-server-properties-sql-server)
* 单用户模式连接，则本地所有用户都可作为sysadmin连接
* 创建表时，在名称前加#可创建会话临时表，结束时自动销毁；加##可创建全局临时表，没有引用时自动销毁
* 不要使用ntext、text、image，改用nvarchar(max)、varchar、varbinary(max)替代
* 分离数据库：sp_detach_db，附加数据库：sp_attach_single_file_db
* 清理日志文件：https://blog.csdn.net/slimboy123/article/details/54575592
* 自定义函数要加as，和mysql不一样；可以返回表：returns table，之后return (select)，如果要自定义表，用`returns @TableName Table(...)`，之后insert再直接return就行
* 存储过程中用set nocount on可以不显示”n行受影响“信息

查询连接方式：

```
SELECT net_transport, auth_scheme, encrypt_option
FROM sys.dm_exec_connections
WHERE session_id = @@SPID;
```

问题
----

### 无法访问数据库XXX (objectexplorer)

* 权限不够
* 数据库文件没有了
* 日志文件满了/储存的磁盘满了

### 错误40/53

* 账号/密码/端口错了
* 服务没有运行

### 消除孤立用户

* https://www.cnblogs.com/13590/archive/2007/09/28/909643.html

### 误删数据恢复

* https://blog.csdn.net/dba_huangzj/article/details/8491327

### 其他博客

* https://www.cnblogs.com/lyhabc/category/505074.html


