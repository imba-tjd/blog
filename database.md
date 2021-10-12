
OLTP适合行储存；OLAP适合列储存（大数据），用于高层分析，数据变化慢，使用人数少。SQL Server内存数据库是BWTree。

## 数据类型

* 整数
* 小数
* 字符串
* 日期
* 类型转换：CAST(原数据 AS 新类型)；CONVERT(原数据, 新类型)

### 字符串

* 第一个字符的索引是1
* 标准规定使用单引号，两个单引号转义出一个
* 大小写不敏感，PG除外
* 拼接：SQLite和PG和Oracle用`||`，MSSQL用`+`，MySQL和MSSQL和PG和Oracle用`CONCAT()`；SQLite和MySQL支持GROUP_CONCAT()，等价于普通语言的字符串JOIN
* 长度：MSSQL用LEN()，其它用LENGTH()。受编码影响，多字节字符慎用；MySQL支持CHAR_LENGTH()
* 替换：REPLACE()
* 子串：SQLite用SUBSTR，其它用SUBSTRING(origin, start, len)。start可以为负，此时SQLite和MySQL能从后开始索引，而PG和MSSQL相当于len=len-start再仍从头开始。SQL标准和PG和MySQL还支持SUBSTRING(str1 FROM x FOR y)的写法
* TRIM()、LTRIM()、RTRIM()；除SQLite外还支持TRIM('a' FROM 'abc')，但仍只会去除开头和结尾的，且是字符列表
* LOWER()、UPPER()

### 日期和时间

* 预定义变量，无需引号：CURRENT_DATE、CURRENT_TIME、CURRENT_TIMESTAMP。MSSQL只支持时间戳，考虑CAST成另两种；MySQL和PG的CTS等价于NOW()
* EXTRACT(YEAR FROM ts)，MSSQL为DATEPART(YEAR, ts)
* 格式化：MySQL：DATE_FORMAT(ts, 'format')，SQLite：STRFTIME('format', datecol)
* SQLite：DATE('now', '+1 month', '-1 day')、time('12:00', 'utc'/'localtime')->04:00:00/20:00:00、DATETIME()
* format同C语言，分别用%Y %m %d %H %M %S，%w为0-6的星期，%s为时间戳秒数

### SQLite

* INTEGER、REAL、TEXT、BLOB
* BOOL内部用0和1存，能识别TRUE和FALSE
* 日期有三种储存方式，默认TEXT
* Type Affinity/Manifest typing/Flexible Typing：创建表的时候指定的类型只是一种建议，甚至可以不指定类型。插入数据时如果值能转换成那种类型就转换，否则不会报错而是就按值的类型存，这导致一列可以有不同类型。能把其它DBMS的类型名识别为自己的类型，这导致可以往VARCHAR(50)里插入1000长度的字符串
* 整数主键是唯一只能存整数的，实际就是ROWID；非整数主键因为历史原因可以为NULL，最好在定义完表后加上WITHOUT ROWID，且最好一行不要超过200字节
* 定义表的字符串或WHERE和ORDERBY时可指定COLLATE RTRIM/NOCASE

### MySQL

* TINYINT, SMALLINT, INT, BIGINT分别使用1、2、4、8个字节；INT UNSIGNED不接受负数；INT(n) ZEROFILL小于列宽时补0且也不接受负数，不影响储存；BOOL就是TINYINT
* FLOAT, DOUBLE/REAL；也能用UNSIGNED
* DECIMAL(P,D)：P为有效位数（包含整数位数和D），D为小数位数
* YEAR, DATE, TIME, TIMESTAMP, DATETIME：分别占1、3、3、4、8个字节，TIMESTAMP最大2038年
* CHAR(0-255), VARCHAR(0-65535), BINARY, VARBINARY：后两者也存字符串但是没有字符集
* [TINY/无/MEDIUM/LONG]BLOB/TEXT：分别为255字节、64KB、16MB、4GB
* BIT(n)：值用`b'111'`表示
* ENUM(x,y,z)
* SET：与ENUM类似，相当于位域。字符串自动对应数字，使用时可用数字相加，或一个以逗号分隔多个值的字符串
* mysql5以后，一个char就代表一个字符，无论中英文
* 标识符使用反引号
* CAST的时候不能只用INT，必须指明SIGNED

## 索引

* 聚集和非聚集默认都是B树，相当于本身就排序好了，因此会影响DML性能，能提高SELECT、WHERE、JOIN的性能；只有内存数据库才支持哈希索引
* 有大量重复值或者太宽不适合建立索引。区分度：COUNT(DISTINCT col1)/COUNT(*)在80%以上就可以
* 聚集索引：一个表只能有一个，顺序与表的物理顺序相同，不应频繁修改索引列；自增对插入和密集读很友好
* 非聚集索引(NONCLUSTERED)：顺序与物理储存顺序不同。MSSQL先把其它列设为聚集索引，再设主键，主键就是非聚集索引了
* 唯一索引：指数据是唯一的（UNIQUE），不是只能创建一个；等值查询性能高
* 筛选索引(MSSQL, SQLite)：创建非聚集索引时添加WHERE，维护开销更小，或用于大部分为NULL时索引非空的；如果查询中有AND，创建索引时要用OR
* 包含列的索引(MSSQL)：创建非聚集索引时添加INCLUDE(col1)，适合col1只存在于SELECT而不在WHERE中时，这些列不会排序故开销更小
* 列存储索引(MSSQL)：适用于频繁使用聚合函数，而行储存适用于表查找；多于100万行时才考虑，适合大量插入、少量更新和删除
* 堆(MSSQL)：没有聚集索引，SELECT的顺序不确定，适合暂存大量无序插入
* 视图索引：也称虚表
* 全文索引：搜索引擎的关键技术，每个表只能有一个
* 非聚集索引可以一次性设置多个列，但在一组AND中使用时要按顺序，不能有间隔，可以只用前面的，不等条件必须在最后；用到的所有列都在索引中叫覆盖索引，不会查表，SQLite还支持把表达式作为索引内容

## 隔离级别

* Read Uncommitted读未提交：会出现脏读，不会脏写
* Read Committed读已提交：可避免脏读，不会读到另一事务未提交的内容，但不可重读；Oracle和SQL Server默认
* Repeatable Read可重复读：可避免不可重读和更新丢失，会出现幻读；Oracle不支持
* Snapshot：一致性非锁定读(MVCC)，可避免不可重读、幻读，但会出现写偏；MSSQL支持，要将ALLOW_SNAPSHOT_ISOLATION设为ON
* Serializable：不会幻读和写偏
* 分布式事务无法实现RR以上的隔离级别
* redolog记录提交了但没写入磁盘的数据，undolog记录未提交的数据。出现故障时，检查点之前完成的事务无须操作，仍在进行的事务回滚，反向扫描日志；检查点之后、发生故障前执行完成的事务前滚/重做，正向扫描日志
* MySQL的RR和PG完全不同，MySQL默认RR但却也可以防止幻读，但是会出现更新丢失

### 并发操作与数据不一致性

* 原子性(Atomicity)不会改一半、一致性(Consistency)其它三个的目标，另一种说法是并发执行的事务最后的结果应与某种顺序串行执行相同、隔离性(Isolation)事务之间互相看不见，乐观锁和悲观锁、持久性(Durability)出问题后不丢数据
* 脏写：某事务a=b=1，另一事务a=b=2，但最后结果a!=b。解决办法：加写锁
* 污读/脏读(Dirty Reads)：某事务更新了数据但还未提交，另一事务读取，能读取到这些内容，但前者进行了回滚，则后者读到了“从未存在”的数据。解决办法：要读/写的时候在需要的行上加读/写锁，其中读锁读过了需要的行以后就释放，写锁直到事务完成才释放
* 不可重读(Nonrepeatable Reads)：某事务读取某行，另一事务更新或删除了该行并提交，前一事务再次读取同一行，则两次结果不一样；读偏：某事务读取某列，另一事务更新了别的列被前一事务读取。解决办法：与上一隔离级别相比，读锁也直到事务完成才释放，事务可以同时持有读写锁；另一种办法是用快照
* 幻读(Phantoms)：某事务读取某些行，另一事务增加了前一事务**条件范围内**的行，前一事务再次读取，则结果不一样。解决办法：与上一隔离级别相比，对整个表加锁或者对条件范围加锁(间隙锁)，因为新增的行在第一次读的时候不存在，无法加锁
* 更新丢失：两个事务读入后都进行写入，则后写覆盖前写。没有脏写因为前写已提交才有后写，没有脏读因为前写了之后根本就没有读。解决办法：同不可重读；或者快照取第一个写的结果，第二个报错
* 写偏：两个事务分别读取和交叉写入两列
* 解决死锁：一次加锁法，但扩大了封锁范围、顺序加锁法，对可能产生死锁的对象规定加锁顺序。但这两者都不太适合数据库，降低了并发度
* 乐观锁：加一个更新时间/hash/Version列，读取的时候读入，写的时候检查是否一致，或者写的时候把读的过滤条件也补上，就会重新过滤一遍

## 优化

1. 优化SQL和索引
2. 缓存，redis
3. 主从复制或主主复制，读写分离
4. mysql自带分区表
5. 垂直拆分
6. 水平切分

* 分表的意义：一部分可能经常变化，另一部分很少变化
* 读多写少可冗余

## SQL Server

* 查询服务器属性：select SERVERPROPERTY('collation')
* 查询数据库属性：select DATABASEPROPERTYEX('Your DB Name','collation')
* SET LANGUAGE 'Simplified Chinese'可更改datetime和货币的表示方式
* ALTER DATABASE <dbname> SET COMPATIBILITY_LEVEL = 150;
* select @@version;
* GO命令：不是T-SQL语句，而是用于分隔sql文件的标记，有些语句要在第一行执行就要再在前面用GO。后面不能跟分号，可以跟数字表示执行上一段多少遍
* exec sp_rename 'a', 'b'：重命名表、索引、列等对象；db用renamedb
* sp_spaceused 查看占用空间
* UPDATE STATISTICS
* WITH (DROP_EXISTING = ON)：创建索引时可用
* sp_helpdb
* CREATE DATABASE db1 ON/LOG ON filespec；ALTER DATABASE db1 ADD/MODIFY/REMOVE FILE/LOG FILE filespec
* sp_helptext：显示定义对象的SQL语句
* ALTER SERVER CONFIGURATION SET MEMORY_OPTIMIZED TEMPDB_METADATA = ON;对TempDB启用内存中OLTP

## SQL Server LocalDB

* C:\Program Files\Microsoft SQL Server\130\Tools\Binn\SqlLocalDB.exe
* 连接地址为`(localdb)\MSSQLLocalDB`
* 有的命令不加实例名时默认为MSSQLLocalDB，但只在该命令中有效
* info列出所有的实例，info加实例名显示一些信息
* create、delete、start、stop
* 需指定数据库级别的排序规则：`create/alter database <dbname> COLLATE Chinese_PRC_CI_AS`（中国默认），非localdb还可以加`_UTF8`；只修改此级别不会影响已有的列，还需要ALTER TABLE一下

## MySQL

* set foreign_key_checks=0：不检查外键约束
* select version()：版本，database()：数据库名，user()：当前用户名，status：状态
* show create table：显示能创建那个表的语句；create table targettb like srctb：按指定表的结构创建另一个表
* 工具：https://github.com/github/gh-ost MySQLTuner-perl
* DESC tb1：显示表结构
* show database
* 定义列时可以加没有FOREIGN KEY的REFERENCE，但它什么都不做，直接忽略

### CLI

* mysql [-h 主机名] -u 用户名 -p：主机参数默认本机可省，用户名一般用root，指定初始数据库用-D，参数也可以不加空格紧挨着写。重定向stdin可读取执行sql脚本
* 修改密码：mysqladmin -u root -p password 新密码；或set password='新密码';

## SQLite

* 没有GRANT和INVOKE、不支持输出参数
* ATTACH DATABASE 'file:data.db?cache=shared&mode=ro/rw/rwc' AS data；数据库名用:memory:可强制创建纯内存数据库，如果为空则会创建临时文件数据库；如果也不会被其它进程改变可用immutable=1
* 元数据：SELECT name, sql FROM sqlite_schema WHERE type='table/index';
* 默认是serializable隔离级别。只有启用共享缓存并且启用PRAGMA read_uncommitted才会读到事务中未commit的数据，此时看作同一个连接。WAL下表现出snapshot隔离级别，开始读取后看不到另一个连接更新的数据，但如果本连接又要更新，则会报错。可用`BEGIN IMMEDIATE`提前加写锁。开始从游标读取后到一半又在同一个连接中更新数据是未定义的
* WAL(3.7,2010)：读写之间可以并发，但多个写不能并发；每个数据库会对应多个文件；数据库级别而不是连接级别，且关闭连接重新打开后还是此模式；不支持NFS
* Py和APT编译的THREADSAFE=1，支持单连接多线程不同时写，需要程序来保证同步；一般为了简单就一个线程一个连接，不过可开共享缓存

### CLI

* 一定要用sqlite3进入，否则默认进的是2；命令行的第三个参数也可以是这些命令，一般用.dump
* ctrl+u清除当前行的输入；.exit退出程序；.help显示帮助；.stats显示连接的基本信息，不是数据库信息；这些命令都不需要加分号，尤其是文件名
* .mode column：更适合阅读的结果展现方式，还能显示列名；控制台最好的是box
* .databases/tables/indexes：列出所有数据库/表/索引；.schema：显示创建指定表的SQL语句；这些都可以跟LIKE语句的模式匹配
* .open data.db：关闭对当前文件并打开另一个；.save/.backup data.db：保存main数据库
* .dump：输出创建表的SQL语句到stdout；.read file.sql：执行SQL文件；.import data.csv tb1：导入csv的数据；.excel data.scv然后运行select语句，保存到csv中
* .shell 运行shell命令；cd：更换目录

### PRAGMA

* 每项前都可以跟schema指定附加的数据库，省略则可能为main也可能为所有数据库；后面要加分号
* optimize 推荐关闭连接时使用，用于内部优化查询性能；如果数据量太大可设定analysis_limit再定期运行避免资源消耗太多
* journal_mode = WAL 对于非内存数据库使用此项可以提高性能；WAL是持久的
* synchronous = NORMAL/1 对于WAL模式时是此项更好，默认是FULL/2
* secure_delete=FAST 默认为true，删除时会写0
* temp_store = MOMORY/2 让临时表放到内存里，默认用的是文件
* foreign_keys = true 开启外键检查；ignore_check_constraints = true 关闭check约束检查，默认启用
* database_list 显示附加了的数据库；table_info(tb1) 用行显示表的列信息
* integrity_check quick_check 进行错误检查，前者更完整，后者更快
* auto_vacuum FULL：默认关闭，当删除数据时不会真的删除，磁盘空间占用不缩小。FULL全自动，INCREMENTAL要定期用pragma incremental_vacuum才行。对于已存在表的数据库，修改为FULL后要运行一遍VACUUM命令才能生效
* compile_options 显示编译选项

## 备份还原

* 数据库完整备份：无法还原备份以后的事务，但简单
* 差异备份
* 事务日志备份：可热备份，但必须和一次完整备份一起才能恢复，而且有一定的时间顺序。可以将数据库还原到失败点，失败点前未提交的无法还原
* 文件及文件组备份：单一文件备份，最好备份后立刻做一个事务日志备份，否则会造成不一致性

## GUI

* Navicat：收费
* Microsoft Azure Data Studio
* HeidiSQL：Delphi，开源，有中文
* https://gethue.com
* https://dbeaver.io：JAVA，开源，Star数多
* https://www.devart.com/free-products.html：其实不用注册，企业版试用过后自动变为Express版
* phpMyAdmin
* DataGrip：收费

---

## TODO

创建登录用户
查询存在哪些数据库、表，改名

SQLite
查询某个表的元信息：PRAGMA table_info(tbname)
内置函数：https://www.sqlite.org/lang_corefunc.html typeof显示值的实际类型
https://www.sqlite.org/pragma.html
on conflict定义在约束后可指定不满足时的操作，默认ABORT，终止语句，保留同一事务中之前插入的，不是回滚

https://github.com/sqlmapproject/sqlmap SQL注入工具
https://github.com/microsoft/sql-server-samples/tree/master/samples

with no check：http://blog.csdn.net/vezn_king/article/details/53225126

https://www.sqlite.org/docs.html 看到features了

MySQL和MSSQL：use db1;

日期：https://sqlzoo.net/wiki/DATE_and_TIME_Reference https://www.runoob.com/sql/sql-dates.html https://www.w3school.com.cn/sql/sql_dates.asp
函数：https://sqlzoo.net/wiki/Functions_Reference
技巧：https://sqlzoo.net/wiki/Hacks_Reference
元数据：https://sqlzoo.net/wiki/Meta_Data_Reference

IFNULL(a,b)：如果a为NULL或者一行数据也没有就返回b，否则仍返回a。MySQL和SQLite支持
ISNULL NVL https://www.runoob.com/sql/sql-isnull.html

查看隔离级别：DBCC USEROPTIONS；修改隔离级别：SET TRANSACTION ISOLATION LEVEL ...，每个数据库都要分别设置

* MySQL游标：https://zhuanlan.zhihu.com/p/41224077
* MySQL的select加HOLDLOCK可临时使用Serializable；还有select for update

各数据库设置隔离级别的方法

主从复制

你们创建的那么多索引，到底有没有生效，或者说你们的SQL语句有没有使用索引查询你们有统计过吗？
有什么手段可以知道有没有走索引查询呢
MySQL可以通过explain查看sql语句的执行计划，通过执行计划来分析索引使用情况

https://roadmap.sh/postgresql-dba

ACCESS:
Get-OdbcDriver
https://www.microsoft.com/zh-cn/download/details.aspx?id=54920
  
PG：
  PostgREST 把PG变成RESTAPI https://github.com/PostgREST/postgrest
