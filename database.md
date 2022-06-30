## 运算

* “表达式”，可用在SELECT、WHERE、HAVING、CHECK等中
  * +-*/%、括号、常数、列名、标量子查询、聚合函数、CASE
  * 除号在MySQL中结果一定是浮点数，用DIV才是整除
  * PG支持用^表示乘方
* WHERE中不能用聚合函数因为那必须在分组后生效，ORDERBY中可以用聚合函数和SELECT中设定的别名
* 通用函数：ABS()、ROUND()、COALESCE(返回第一个不为NULL的参数)、RANDOM()

### CASE

* 可用于分类
* WHEN里的也可以是一般的带比较运算符的条件表达式，此时可不写CASE后的表达式，THEN里的可以是子查询等表达式
* 非标准的简化：MySQL有IF函数，Oracle有DECODE函数

```sql
SELECT col0
, CASE col1
    WHEN 'A' THEN 'it is a'
    ELSE 'other' -- 不写相当于ELSE NULL
END AS col2
FROM ...

-- 逻辑中的 x->y “蕴含”。注意x为假时结果为真，不只有x和y都为真时才为真，或者说只有x为真y为假时才为假
CASE x
WHEN a
    THEN CASE y
    WHEN b
        THEN 1
        ELSE 0
    END
ELSE 1
END
```

## 数据类型

* 整数
* 小数
* 字符串
* 日期
* 类型转换
  * CAST(原数据 AS 新类型) 标准写法，MySQL必须指明SIGNED
  * CONVERT()：MySQL第一个参数是数据，第二个参数是类型；MSSQL刚好相反；PG SQLite不支持
  * SQLite MySQL在处理字符串和数字之间运算时会隐式转换成数字，如'1.1'+'+1'->2.1，'a1'也是转换成1
* 获得表达式的类型：SQLite typeof()，PG pg_typeof()，MySQL可以创造临时表再查看表结构

### 字符串

* 第一个字符的索引是1
* 标准规定使用单引号，两个单引号转义出一个
* 大小写不敏感，PG除外
* 拼接
  * SQLite PG和Oracle用`||`
  * MSSQL用`+`
  * 都支持CONCAT()
  * SQLite MySQL支持GROUP_CONCAT()聚合函数，MSSQL PG支持STRING_AGG()，对应普通编程语言的字符串JOIN，其中前者第二个参数可省默认为逗号，有的还支持再排序
* 长度：MSSQL用LEN()，其它用LENGTH()。受编码影响，多字节字符慎用；MySQL支持CHAR_LENGTH()
* 替换：REPLACE(X,Y,Z) 把X里的Y替换成Z
* 子串
  * SUBSTRING(origin, start, len)。老版本SQLite只能用SUBSTR
  * start可以为负，此时SQLite MySQL能从后开始索引，而PG MSSQL相当于len=len-start再仍从头开始
  * 标准、PG、MySQL还支持SUBSTRING(str1 FROM x FOR y)的写法
* TRIM()、LTRIM()、RTRIM()，SQLite PG支持第二个参数指定字符列表；标准支持TRIM([LEADING/TRAILING] 'a' FROM 'abc')，SQLite除外，MySQL不看做字符列表而是字符串
* LOWER()、UPPER()
* COLLATE
  * SQLite：BINARY默认 NOCASE RTRIM去除尾随空格。可用在定义TABLE时，或SELECT中WHERE ORDERBY列名后
  * MSSQL：Chinese_PRC_CI_AS_SC_UTF8，其中CI不区分大小写，CS区分；AS区分重音但对中文好像没用；SC为启用U16非标准平面但好像2017默认启用了；_BIN2为二进制，不兼容其他的

### 日期和时间

* 预定义变量，无需引号：CURRENT_DATE、CURRENT_TIME、CURRENT_TIMESTAMP
  * CTS并不是秒数，而是类似于DATETIME
  * MSSQL只支持CTS，可CAST成另两种类型
  * MySQL PG的CTS等价于NOW()，SQLite的CTS等价于DATETIME()
* 提取部分：EXTRACT(YEAR FROM ts)，MSSQL为DATEPART(YEAR, ts)，SQLite不支持
* 格式化
  * MySQL：DATE_FORMAT(ts, 'format')
  * SQLite：STRFTIME('format', datecol)
  * format同C语言，分别用%Y %m %d %H %M %S，%w为0-6的星期，%s为时间戳秒数
* SQLite
  * date('now', '+1 month', '-1 day')
  * time('12:00', 'utc'/'localtime') -> 04:00:00/20:00:00
  * UNIXEPOCH() Unix时间戳，秒数整数。秒数转日期用datetime(1092941466, 'auto')

### SQLite

* INTEGER、REAL、TEXT、BLOB
* BOOL内部用0和1存，能识别TRUE和FALSE
* 日期有三种储存方式，默认TEXT
* Type Affinity/Manifest typing/Flexible Typing
  * 创建表的时候指定的类型只是一种建议，甚至可以不指定类型
  * 插入数据时如果值能转换成那种类型就转换，否则不会报错而是就按值的类型存，这导致一列可以有不同类型
  * 能把其它DBMS的类型名识别为自己的类型，这导致可以往VARCHAR(50)里插入1000长度的字符串
  * 3.37(2021.11)：在定义完表的回小括号后加STRICT能禁用Flexible Typing且不允许不加类型，又引入了ANY类型；ANY在非STRICT表中会优先转换成整数，与不加类型行为不同
* 整数主键INTEGER PRIMARY KEY只能存整数，实际就是ROWID的别名；非INTEGER的PRIMARY KEY因为历史原因就等于UNIQUE且可以出现NULL，现在若要使用非整数主键，应在定义完表的回小括号后加WITHOUT ROWID，且最好不要超过200字节，这样不会出现NULL且效率更高
* 定义表的字符串或WHERE和ORDERBY时可指定COLLATE RTRIM/NOCASE
* STRICTh和WITHOUT ROWID若要同时指定，加逗号

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
* MySQL5后，一个char就代表一个字符，无论中英文
* 标识符使用反引号。若要用双引号，需SET GLOBAL/SESSION sql_mode='ANSI'或'ANSI_QUOTES'或在配置中sql-mode="ANSI"
* innodb_file_per_table=1
* ALTER DATABASE xxx CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
* mysqldump --default-character-set=utf8mb4

### MSSQL

* TEXT类型已弃用，应用varchar(max)

## 理论

* OLTP适合行储存，也即普通的RDBMS。OLAP适合列储存、大数据、数据仓库，用于高层分析，数据变化慢，使用人数少

### 索引

* 相当于本身就排序好了，因此会影响DML性能，能提高SELECT、WHERE、JOIN的性能
* 数据结构
  * MSSQL MySQL的InnoDB用B+树
  * PG用B树但支持其它几种策略，关键是它的表结构用堆储存，区分两者意义不大
  * SQLite有两种B树，且都不是传统的，但是文档中都称为B树
  * Oracle用B树
  * MongoDB现在的WiredTiger存储引擎用B+树
* 有大量重复值或者太宽不适合建立索引。区分度：COUNT(DISTINCT col1)/COUNT(*)在80%以上就可以
* 聚集索引：一个表只能有一个，顺序与表的物理顺序相同，叶子储存数据值，不额外占用空间，不应频繁修改；自增对插入和密集读很友好，用UUID容易导致页分裂
* 非聚集索引(NONCLUSTERED)
  * 顺序与物理储存顺序不同，叶子储存主键id
  * 可以一次性设置多个列，但在一组AND中使用时要按顺序，不能有间隔，可以只用前面的，不等条件必须在最后
  * 用到的所有列都在索引中叫覆盖索引，不会查表
  * SQLite PG支持把表达式作为索引内容，如ON(a+b)，当查询时出现同样的表达式如WHERE a+b=xxx时就会使用
  * 有的主键允许用非聚集索引，但研究这个没意义
* 唯一索引(UNIQUE)：指数据是唯一的，不是只能创建一个；等值查询性能高
* 筛选索引/部分索引(MySQL除外)：创建非聚集索引时添加WHERE，维护开销更小；如果查询时WHERE中有AND，创建索引一般要用OR
* 包含列的索引(MSSQL)：创建非聚集索引时添加INCLUDE(col1)，适合col1只存在于SELECT而不在WHERE中时
* 列存储索引(MSSQL)：适用于频繁使用聚合函数，而行储存适用于表查找；多于100万行时才考虑，适合大量插入、少量更新和删除
* 堆(MSSQL)：没有聚集索引的表，SELECT的顺序不确定，适合暂存大量无序插入
* 视图索引：也称虚表
* 全文索引：搜索引擎的关键技术，每个表只能有一个
* 哈希索引：好像只有内存数据库才支持，而且没法进行范围比较

### WHERE中索引失效

* LIKE '%xxx'
* 列类型与操作数类型不同，包括字符集不同
* 使用函数。这种情况需要建立基于函数的索引
* IS [NOT] NULL。索引不会储存空值。但SQLite说可以用
* 对列进行加减乘除运算
* 复合索引没有按最左匹配原则
* 复合索引a>1 and b=2，a可以走索引二分，但a没有唯一确定，b无法走索引；要尽量前面的判断相等，最后才能用不想等
* 选择的列太多。避免SELECT*
* OR了不在索引里的列
* IN太多，少量的其实和=是一样的。看能不能换成BETWEEN

### 隔离级别、并发、数据不一致性

* ACID
  * 原子性(Atomicity)要么都执行要么都不执行
  * 一致性(Consistency)其它三个的目标
  * 隔离性(Isolation)事务之间互相看不见，不会以另一事务修改了一半的状态为基础
  * 持久性(Durability)出问题后也不丢数据
* 脏写
  * 某事务a=b=1，另一事务a=b=2，但最后结果a!=b
  * 解决办法：加写锁
  * 解决后隔离级别为*Read Uncommitted*读未提交，不会脏写，会出现脏读
* 污读/脏读(Dirty Reads)
  * 先写后读，某事务更新了数据但还未提交，另一事务读取，能读取到这些内容，但前者进行了回滚或又写了新的，则后者读到了“从未存在”（未提交）的数据
  * 解决办法：要读/写的时候在需要的行上加读/写锁，其中读锁读过了需要的行以后就释放，写锁直到事务完成才释放，使得写了未提交时无法读
  * 解决后隔离级别为*Read Committed*读已提交，可避免脏读，但不可重读。Oracle MSSQL PG默认
* 不可重读(Nonrepeatable Reads)
  * 先读后写，某事务读取某行，另一事务更新或删除了该行并提交，前一事务再次读取同一行，则两次结果不一样
  * 更新丢失/写丢失：两事务读入后都进行写入，则后写覆盖前写。没有脏写因为前写已提交才有后写，没有脏读因为前写了之后根本就没有读
  * 读偏：某事务读取A列，另一事务更新B列并提交，前一事务再次读取B列，则两列数据来自不同事务。我觉得如果AB在同一行，与普通的不可重读没有区别
  * 解决办法：读锁也直到事务完成才释放，使得另一事务必须等读完后才能写
  * 解决后隔离级别为*Repeatable Read*可重复读，会出现幻读，不会出现读偏写偏更新丢失
    * 分布式事务无法实现RR以上的隔离级别
    * MySQL默认RR，但又用了MVCC多版本并发控制，也用了间隙锁，能减少幻读
* 幻读(Phantoms)
  * 某事务读取某些行，另一事务插入了前一事务**条件范围内**的行，前一事务再次读取，则结果不一样。其实跟不可重读一样，只是之前只给选中的行加锁，这次给WHERE条件加锁
  * 解决办法：与上一隔离级别相比，对整个表加锁或者对条件范围加锁(间隙锁)，因为新增的行在第一次读的时候不存在，无法对行加锁
  * 解决后隔离级别为*Serializable*
* *Snapshot*快照隔离
  * 另一种解决可重读的方向。不基于锁，另一事务可以写，前一事务可以读，只是会读到旧内容，不会幻读
  * 若按表版本控制，则没有和序列化不一致的问题，只要写时不是最新的就失败；若按行版本，会出现“幻读的写入版”，如果插入了条件内的行，老行的版本没变，但已经不是最新的了。如果不进行版本控制，那确实仍然不会出现最基本的不可重读问题；如果按按列值进行版本控制，也能防止更新丢失
  * 写偏(Write Skew)：某事务读取A列，另一事务读取B列，前一事务写入B列，后一事务写入A列。我不明白为什么有文章说会出现写偏，除非没有按行而是按列值版本控制
* 死锁的解决办法
  * 若允许读锁升级为写锁，RR下更新丢失的情形就会死锁，两者都持有读锁，都在等其它读锁消失加写锁
  * 一次加锁法，但扩大了封锁范围
  * 顺序加锁法，对可能产生死锁的对象规定加锁顺序
  * 都不太适合数据库，降低了并发度
  * MSSQL能自动检测，如果发现了死锁，就终止一个
  * 死锁预防、死锁检测、死锁恢复
* 乐观锁：加一个更新时间/Hash/Version列，读取的时候读入，写的时候检查是否一致；快照隔离一半用的就是它
* 日志：redolog记录提交了但没写入磁盘的数据，undolog记录未提交的数据。出现故障时，检查点之前完成的事务无须操作，仍在进行的事务回滚，反向扫描日志；检查点之后、发生故障前执行完成的事务前滚/重做，正向扫描日志
* SQLite单纯BEGIN是没有加任何锁的，其他连接提交了能看到，开始SELECT了才会加锁。PG不同，BEGIN了，其他提交了看不到
* PG：默认RC下就已经能重读了，好像也不存在幻读，读写也能并发，都跟快照差不多了，写入时如果存在任意未提交的就会报错；RR下能并发写，在提交时非最新版报错，感觉就等于快照了
* MSSQL：默认RC下非常标准，脏读就是会阻塞的，也会出现不可重读和幻读。RR下的行为非常怪，另一事务可以写，未提交时读取的事务会阻塞即不会脏读，提交后读取的事务能读到新内容，这不就是完全没有实现吗？难道是因为localdb的问题？

### 备份还原

* 数据库完整备份：无法还原备份以后的事务，但简单
* 差异备份
* 事务日志备份：可热备份，但必须和一次完整备份一起才能恢复，而且有一定的时间顺序。可以将数据库还原到失败点，失败点前未提交的无法还原
* 文件及文件组备份：单一文件备份，最好备份后立刻做一个事务日志备份，否则会造成不一致性

### 优化

* 顺序
  1. 优化SQL和索引
  2. 缓存，redis
  3. 主从复制或主主复制，读写分离
  4. mysql自带分区表
  5. 垂直拆分：按不同业务或活跃度把一些列分到另一个表
  6. 水平切分：按照一定策略（hash、range）把一个表的数据分到多个表
* 分表的意义：一部分可能经常变化，另一部分很少变化
* 分表的问题：分布式事务、跨节点Join和聚合（分别取出再在应用层处理）、ID无法自增（可用UUID）
* 读多写少可冗余

## SQL Server

* 查询服务器属性：select SERVERPROPERTY('collation')
* 查询数据库属性：select DATABASEPROPERTYEX('Your DB Name','collation')
* 更改会话语言：SET LANGUAGE 'Simplified Chinese'可更改datetime和货币的表示方式，以及报错信息
* 升级大版本后，已存在的数据库的版本不会自动升级，需要修改它们的COMPATIBILITY_LEVEL
* select @@version;
* GO命令：不是T-SQL语句，而是用于分隔sql文件的标记，有些语句要在第一行执行就要再在前面用GO。后面不能跟分号，可以跟数字表示执行上一段多少遍
* exec sp_rename 'a', 'b'：重命名表、索引、列等对象；db用renamedb
* sp_spaceused 查看占用空间
* UPDATE STATISTICS
* WITH (DROP_EXISTING = ON)：创建索引时可用
* sp_helpdb
* CREATE DATABASE db1 ON/LOG ON filespec；ALTER DATABASE db1 ADD/MODIFY/REMOVE FILE/LOG FILE filespec
* sp_helptext：显示定义对象的SQL语句
* ALTER SERVER CONFIGURATION SET MEMORY_OPTIMIZED TEMPDB_METADATA = ON;对TempDB启用内存中OLTP
* ALTER DATABASE ... SET DELAYED_DURABILITY = FORCED：延迟事务持续性，IO太重又可容忍丢失部分数据可以启用；若设为ALLOWED就必须每次事务手动控制

### LocalDB

* C:\Program Files\Microsoft SQL Server\150\Tools\Binn\SqlLocalDB.exe create/delete/start/stop 实例名，不加时或为点默认为MSSQLLocalDB
* info无参使用列出所有的实例，info加实例名显示命名管道等信息，有可能客户端要去掉np前缀
* 如果启动失败，考虑删除重建，有可能是用了老版本的数据库
* 连接字符串：`Server=(localdb)\\.;Integrated Security=true`。如果点不支持就换成MSSQLLocalDB或其它实例名；IntegratedSecurity好像不加也行
* 需指定数据库级别的排序规则：`CREATE/ALTER DATABASE db1 COLLATE Chinese_PRC_CI_AS`；不会影响已有的列，还需要ALTER TABLE一下
* 下载地址：https://www.hanselman.com/blog/download-sql-server-express
* 数据库文件和日志在`%LocalAppData%\Microsoft\Microsoft SQL Server Local DB\Instances\MSSQLLocalDB`

### Azure SQL Edge

* 仅Linux，同样使用MSSQL引擎，可用Docker拉取，需求内存1GB，磁盘10GB

## MySQL

* set foreign_key_checks=0：不检查外键约束
* select version()：版本，database()：数据库名，user()：当前用户名，status：状态
* show create table：显示能创建那个表的语句；create table targettb like srctb：按指定表的结构创建另一个表
* 工具：https://github.com/github/gh-ost MySQLTuner-perl
* DESC tb1：显示表结构
* show database
* 定义列时可以加没有FOREIGN KEY的REFERENCE，但它什么都不做，直接忽略

### my.cnf

```conf
启用慢查询日志，如果执行时间大于3秒则记录
slow_query_log=1
slow_query_log_file=/var/log/mysql/log-slow-queries.log
long_query_time=3
```

### CLI

* mysql [-h 主机名] -u 用户名 -p：主机参数默认本机可省，用户名一般用root，指定初始数据库用-D，参数也可以不加空格紧挨着写。重定向stdin可读取执行sql脚本
* 修改密码：mysqladmin -u root -p password 新密码；或set password='新密码';

## SQLite

* 没有GRANT和INVOKE、不支持输出参数
* 一个文件就是一个database，没有create database语句。一个连接可以打开多个文件
* ATTACH DATABASE 'url' AS data
* Linux下默认的权限是都能读但只有创建者能写
* URI文件名
  * file:data.db?cache=shared&mode=ro/rw/rwc
  * 文件名用:memory:为内存数据库，默认仍会在tmp中产生处理临时表的文件；文件名用空的则为临时文件数据库
  * 如果也不会被其它进程改变可用immutable=1
  * cache=shared：单进程多连接都启用此参数能类似于一个连接减少资源占用，内部再自动序列化，感觉可以无脑开
  * 打开已存在的数据库时小心别打错字，否则就自动创建了一个新的，或者mode不用rwc就能避免
  * psow=1：假定断电时文件系统不会写入超过范围的数据
* 读写锁和事务
  * 事务一开始，单纯的BEGIN是没有加锁的。开始读取了会加SHARED锁，允许有多个，能同一时间多个连接并发读取
  * 当遇到了DML语句，会给整个数据库文件加RESERVED锁，只允许有一个，然后写日志。此时仍可以正常读，也允许新连接读，读不到未提交的，因为新内容在日志里，不在文件里
  * 当COMMIT时，会加PENDING锁，能阻止新连接读，但无法阻止老连接多游标不间断读。如果一段时间内仍无法加上，就会失败，此时事务并未取消，可以再次提交
  * 当所有的读锁都结束后，就加EXCLUSIVE锁，此时只有本连接能读写
  * BEGIN IMMEDIATE提前加RESERVED锁；BEGIN EXCLUSIVE在WAL下与IMMEDIATE相同
* 隔离级别
  * 非WAL下为serializable，读写到的肯定都是最新的
  * 只有启用共享缓存且启用PRAGMA read_uncommitted才会读到另一连接的事务中未commit的数据，此时不会获得读锁，被认为是同一连接，不会被block或者block别人
  * 一个连接可以有多个游标，它们之间没有隔离也没有锁。一个游标读取到一半又有另一个游标更新数据是未定义的
* 并发和线程安全
  * THREADSAFE=1下，官方说是“线程安全”的；=2时官方说只要没有两个线程同时使用同一个连接就是安全的
  * 各种非官方文章说即使=1下也不能重用连接，只是能多线程使用此模块，因为存在全局状态。我认为不是这样，=1下单个连接可以同时使用多个游标，只是没有隔离
  * =0时不应用在多线程程序中，官方CLI就是=0，所以应该仍可以多个进程使用同一个数据库文件
  * Linux下一定不能打开连接、fork()、再在子进程中用原来的连接
* WAL
  * 隔离性表现为Snapshot，开始读取事务后另一连接能并发写且能提交，本连接始终读到的是旧数据；如果之后本连接又要写，则会报错，因为数据不是最新的，解决办法是一开始BEGIN IMMEDIATE。释放完本连接所有读锁后再读到的是新数据，或者新连接读到的也是新数据
  * 每个数据库会生成对应多个文件，正常退出后会删除
  * 对应数据库级别或者所有连接，不是单个连接级别，且是持久的，关闭连接重新打开后还是此模式
  * 不支持NFS
* rqlite：Go实现的分布式关系数据库，使用SQLite作为储存，三大系统都支持，还提供了多种语言的binding。相比之下dqlite的生态就差得多

### CLI

* 一定要用sqlite3进入，否则默认是2；命令行的第三个参数也可以是下列这些命令，一般用.dump；这些命令都不需要加分号，尤其是文件名
* Ctrl+U清除当前行的输入，CMD下是ESC；.exit和Ctrl+C退出程序；.help显示帮助；.version/.v显示版本和编译器版本；.stats显示当前连接消耗的内存
* .mode/m box/column：更适合人类阅读的展现方式，还能显示列名，但数据太多时反而不利于阅读
* .databases或da/tables或ta/indexes或in：列出所有数据库/表/索引；.schema/sch [tb1]：显示创建指定表的SQL语句；.dbinfo能看到有多少个表索引视图，其他信息用处不大；这些都可以跟LIKE语句的模式匹配
* .open data.db：关闭当前文件并打开另一个；.backup/.save data.db：另存main数据库
* .dump/d [tb1]：输出创建表的SQL语句到stdout，.recover：对于受损的数据库尽可能dump数据；.read file.sql：执行SQL文件；.import data.csv tb1：导入csv的数据；输出到csv：.headers on; .mode csv; .once/.output data.csv; select ...
* .shell/sh 运行shell命令；.cd：略
* .timeout：等待加锁的时间
* 用DQL取得schema元数据：SELECT name, sql FROM sqlite_schema WHERE type='table/index'; 版本：SELECT sqlite_version();

### PRAGMA

* 每项前可以跟`schema名.`指定附加的数据库，省略则可能为main也可能为所有数据库；后面要加分号；打错字了不会报错；几乎所有设置都限于连接非持久，下次连接还要设置
* optimize 推荐关闭连接时使用，或长时间连接每隔几小时用一次，用于内部优化查询性能
* journal_mode = DELETE/TRUNCATE/PERSIST/WAL/MEMORY。默认DELETE完成事务后就删除日志，TRUNCATE不删除日志文件只是清空，PERSIST在日志头部写零；这三种性能依次少量提升，这日志不在临时文件夹而是在数据库同级目录；MEMORY不安全但也算能用，OFF无法ROLLBACK无意义
* synchronous = 默认是FULL/2，切换成WAL会自动改为NORMAL/1
* secure_delete = FAST/2 默认为off，开启后删除时会写0
* temp_store = MOMORY/2 让临时表放到内存里，默认用tmp下的临时文件
* database_list 显示附加了的数据库文件；table_list 显示存在哪些表(3.37,2021.11)；table_info(tb1) 显示表的列信息，每项一行
* integrity_check quick_check 进行错误和约束检查，前者更完整，后者更快
* auto_vacuum FULL 默认关闭，当删除数据时不会真的删除，磁盘空间占用不缩小。FULL全自动，INCREMENTAL要定期用pragma incremental_vacuum才行。对于已存在表的数据库，修改为FULL后要运行一遍VACUUM命令才能生效
* compile_options 显示编译选项
* PRAGMA mmap_size=xxx字节 能提高IO效率，但发生IO错误时无法捕获，Win下无法VACUUM
* 用DQL取得元数据：SELECT * FROM pragma_xxx

### 编译

```bash
#gcc sqlite3.c -o sqlite3.dll -shared -lpthread  # 编译dll
gcc sqlite3.c shell.c -o sqlite3.exe \
-Os -mtune=native -Wl,--as-needed -Wl,--strip-all \
-DNDEBUG \
-DSQLITE_ENABLE_ATOMIC_WRITE \
-DSQLITE_LIKE_DOESNT_MATCH_BLOBS \
-DSQLITE_OMIT_DECLTYPE \
-DSQLITE_OMIT_DEPRECATED \
-DSQLITE_OMIT_LOAD_EXTENSION \  # 不指定此项则要加-ldl -lm
-DSQLITE_UNTESTABLE \
-DSQLITE_USE_ALLOCA \
-DSQLITE_WIN32_MALLOC \
-DSQLITE_DEFAULT_FOREIGN_KEYS \
-DSQLITE_DEFAULT_AUTOVACUUM \
-DSQLITE_DEFAULT_MEMSTATUS=0 \  # 会禁用.dbinfo和.stats
-DSQLITE_DQS=0 \  # 禁用双引号表示字符串
-DSQLITE_MAX_EXPR_DEPTH=0 \
-DSQLITE_POWERSAFE_OVERWRITE \
-DSQLITE_TEMP_STORE=2 \
-DSQLITE_THREADSAFE=0  # 编译dll时不加
```

### 内置函数

* likely() unlikely() 给优化器提示大概率为真/假，默认WHERE中没有索引的大概率为真
* quote() 进行某些转义，如字符串两边加引号
* format() 类似于printf

## PostgreSQL

* 默认端口5433
* 命令行：psql "host=xxx port=xxx dbname=xxx user=xxx"
* 具有jsonb类型
* Linux下安装：https://www.postgresql.org/download/linux/debian/ 仅客户端为postgresql-client
* Win下客户端可用pipx install pgcli

## Access

* Get-OdbcDriver
* ACE驱动下载：https://www.microsoft.com/zh-cn/download/details.aspx?id=54920
* OLEDB通过COM调用，比ODBC高层，比ADO低层。感觉没有必要使用

## 工具和其它DBMS

* https://github.com/sqlmapproject/sqlmap 注入
* https://github.com/PostgREST/postgrest 把PG变成RESTAPI
* https://clickhouse.com/ OLAP数据库
* https://github.com/questdb/questdb/blob/master/i18n/README.zh-cn.md 国产，基于PG，JAVA，自带WebUI
* etcd、Consul
* edgedb：自创DML的关系型数据库，基于pg
* https://duckdb.org/ OLAP
* https://github.com/directus/directus node，给数据库创建RESTAPI
* https://github.com/milvus-io/milvus/ 国产向量搜索引擎

### GUI

* HeidiSQL：Delphi，有中文，支持MySQL(选6.1版本的dll)、MSSQL、PG、SQLite(内置dll)，有32位，上架了商店但为32位，有少量的维护功能，对于大量数据很卡，对MSSQL的脚本不太兼容
* https://dbeaver.io：JAVA，Star数多
* https://www.beekeeperstudio.io/ Electron，有便携版，常见的四种都支持
* phpMyAdmin：Php+Web，仅MySQL，有中文，一般在数据库服务器本身上搭建
* MySQL WorkBench：官方客户端，大小也不大
* pgAdmin：官方客户端，163MB
* https://sqlitebrowser.org/ C++，目标是让普通用户也能用，有中文
* https://www.devart.com/free-products.html：闭源不跨平台，有MSSQL MySQL PG Oracle，企业版试用过后自动变为免费版，下载需要注册，曾经有单独的Express版还更小
* https://github.com/webyog/sqlyog-community 仅MySQL，贡献者极少
* https://sqlitestudio.pl/ C+Qt，仅SQLite，有中文但很多条目还是未翻译
* https://sqlectron.github.io/ 感觉相比beekeeper唯一优势是有32位，大小都差不多
* https://gethue.com Py+Web，主要支持一些大数据的数据库，虽然提交数很多贡献者也很多，但Star很少，估计就是公司自己在用
* Microsoft Azure Data Studio：感觉不如直接VSC，反正功能都少
* DataGrip：JB的，收费
* Navicat：收费
* SequelPro/Ace：仅Mac

### 在线测试

* https://dbfiddle.uk 可能出错的语句以及查询语句最好分开写，否则可能显示不出，尤其是SQLite
* https://www.db-fiddle.com js的cdn被封IP了
* http://sqlfiddle.com/ 不新
* https://sqliteonline.com/ 感觉比较好
* https://demo.questdb.io/ 仅兼容PG
* https://livesql.oracle.com 需要登录
* https://sandboxsql.com 仅SQLite，但自带三个示例数据库

### 示例数据库

* https://github.com/microsoft/sql-server-samples/tree/master/samples/databases 有四个，都是MSSQL的创建脚本
* https://github.com/lerocha/chinook-database
* https://github.com/jpwhite3/northwind-SQLite3
* https://github.com/harryho/db-samples Northwind的其它数据库版本

## 教程

* https://database.guide/sql-tutorial-for-beginners/
* https://github.com/microsoft/sql-server-samples/tree/master/samples/tutorials
* https://www.guru99.com/sqlite-tutorial.html
* https://dba.stackexchange.com
* https://roadmap.sh/postgresql-dba
* https://www.db-recruiter.com 7天免费

---

## TODO

* 创建登录用户
* 查询存在哪些数据库、表，数据库改名
* 各数据库设置隔离级别的方法
* 删除存在外键引用的值或列时怎么办
* 通过SSH tunnel连接
* https://docs.microsoft.com/zh-cn/sql/relational-databases/tutorial-getting-started-with-the-database-engine https://docs.microsoft.com/zh-cn/sql/t-sql/tutorial-writing-transact-sql-statements
* https://www.1keydata.com/cn/sql/

### 未整理

```
MySQL：
死磕MySQL系列，不错 https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=2015004473084936195
创建表时支持COMMENT语句
RLIKE 正则匹配
select locate('You','LoveYou3000Times')将返回5，select find_in_set('Times','Love,You,3000,Times')将返回4
select加HOLDLOCK可临时使用Serializable；还有select for update
dayofyear() date_add() date_sub(ts, interval 3 hour) timestampdiff(hour,'2021-01-03 23:00:03','2021-01-03') curdate() dayname()中文星期
select weekofyear('2022-01-02');返回52，dayofweek('2022-01-02')返回1
如何比较日期数据 https://mp.weixin.qq.com/s?__biz=MzAxMTMwNTMxMQ==&mid=2649247727&idx=1&sn=414455c2f0303a55a31e0c189f1e2c12

MSSQL：
创建数据库 https://docs.microsoft.com/zh-cn/sql/relational-databases/databases/create-a-database
insert ignore，replace into，insert on duplicate key update
可以DISABLE索引

---

切换数据库：MySQL和MSSQL：use db1;

with no check：http://blog.csdn.net/vezn_king/article/details/53225126

日期：https://sqlzoo.net/wiki/DATE_and_TIME_Reference https://www.runoob.com/sql/sql-dates.html https://www.w3school.com.cn/sql/sql_dates.asp
函数：https://sqlzoo.net/wiki/Functions_Reference
技巧：https://sqlzoo.net/wiki/Hacks_Reference
元数据：https://sqlzoo.net/wiki/Meta_Data_Reference

IFNULL(a,b)：如果a为NULL或者一行数据也没有就返回b，否则仍返回a。MySQL和SQLite支持
ISNULL NVL https://www.runoob.com/sql/sql-isnull.html

查看隔离级别：DBCC USEROPTIONS；修改隔离级别：SET TRANSACTION ISOLATION LEVEL ...，每个数据库都要分别设置

https://zhuanlan.zhihu.com/p/429637485 mysql和redis数据一致性问题

Citus：分布式PG
```
