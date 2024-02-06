---
title: SQL
---

## 杂项语法

* 注释：标准只有单行注释`--`，能用于行尾；实际也支持`/* */`，Oracle除外；MySQL支持`#`
* 命名：一般不加s，列名小写，表名首字母大写，关键字全大写
* 写完一句后要打分号
* 标准规定标识符用双引号

## DQL

* 顺序：FROM-WHERE-GROUPBY-HAVING-SELECT-DISTINCT-UNION-ORDERBY-LIMIT

### SELECT

* 计算后的内容一般要用AS命名，此关键字可省但一般不省；FROM的也可用AS命名，一般省
* DISTINCT：在SELECT后，不是每列前都加，对一整行生效，但聚合函数参数中可以加；NULL只能出现一次。PG支持DISTINCT ON(col1)，其它可考虑GROUPBY
* 选取前n条：SQLite MySQL在最后`LIMIT n OFFSET m`或`LIMIT m,n`；MSSQL`SELECT [DISTINCT] TOP n [PERCENT]`，或`ORDER BY ID OFFSET m ROWS FETCH NEXT n ROWS ONLY`也是标准；Oracle`WHERE ROWNUM<=n`
* FROM的可以是子查询且此处必须命名，此时可以有多列
  * PG MSSQL表达式临时表：FROM (VALUES (1,'a'),(2,'b')) AS tmp (a,b)
* 关联子查询(Dependent/Correlated Query)：一个查询的WHERE里含有子查询，且子查询里使用了外部查询的列。外部查询的每一行都会执行一次子查询。例如想选择价格大于所属类别的平均价格的条目，其中计算某一类的平均价格本来需要分组和聚合函数，但最终目标又是选择条目，则需要在子查询中用WHERE过滤与外层相同的类型，就会用到外层FROM的表，分组却不必要了。如果子查询独立，应优先考虑JOIN子查询临时表，或WITH语句
* SELECT INTO：目标表不能已存在，会自动创建。最好只在MSSQL中作为不支持CREATE TABLE AS的替代；PG支持，SQLite不支持，MySQL仅支持into变量

### JOIN

* 如果存在同名的列，SELECT中要加表名，可用tb1.*表示第一个表的所有列
* CROSS JOIN：笛卡尔积，参数只需加表名，无需on；MySQL允许有on但直接忽略
* INNER JOIN tb2 ON tb1.col1 = tb2.col2：等价于老版本标准的FROM t1, t2 WHERE ...
* LEFT/RIGHT/FULL JOIN：左外连接就是完全保留左边的，右边的如果没有就为NULL；LEFT时一般把小表放到左边。SQLite3.39前只支持LEFT
* LEFT JOIN WHERE tb2.id IS NULL可查询仅存在于左边的，即减去共有部分
* SELF JOIN：如选择来自于相同城市的人、时间间隔
* 内连接不会把col1.NULL与col2.NULL看作相等，会忽略

### WHERE

* 等于用`=`，标准的不等于为`<>`
* NOT、AND、OR：与一般的编程语言一样
* IS NULL、IS NOT NULL：只有这个能判断。col = NULL和col != NULL或任何其它运算，结果都为NULL，用在WHERE中就恒假
* [NOT] BETWEEN 1 AND 9：测试的几个都为闭区间
* IN (1,2,3或单列子查询)、NOT IN：相当于多个判断等于的OR。`3 IN (1,2,NULL)`和NOT IN结果都是NULL，用子查询时尤其注意
* [NOT] LIKE：默认不区分大小写，PG除外用ilike不区分。用%表示任意字符，用_表示单个字符；只有MSSQL支持`[]`。正则匹配：MySQL用RLIKE，PG用SIMILAR TO，MSSQL不支持，SQLite只内置了接口默认没实现
* ANY、ALL：前加比较运算符，后跟子查询。用比较运算符（不等号）时可分情况改成聚合函数MIN和MAX，=ANY和!=ALL可换成IN和NOT IN，但=ALL和!=ANY没有等价的
* [NOT] EXISTS：前面不跟列，后跟子查询，判断是否返回至少一行数据，注意NULL也算存在
* 拼接时常加`WHERE 1=1`，方便之后加AND
* MySQL SQLite对于非空支持truthy，即WHERE A等价于WHERE A IS NOT NULL，WHERE 1永远为真
* 大于等于 比 大于 更优化

### GROUP BY

* 聚合函数，会自动忽略为NULL的：AVG、COUNT、MIN、MAX、SUM
  * COUNT(*)：行数，不忽略NULL
  * WHERE中不能用，分组后的步骤才能用
  * 用了聚合函数，即使没有GROUPBY也不能SELECT普通的列
* 分组后，SELECT和HAVING的列只能是常数、GROUPBY的列、聚合函数，不能SELECT *
  * 有的DBMS支持SELECT不在GROUPBY中的列，但没啥意义
* 分组后一行就相当于一组，聚合函数会对每一组用一遍
* 会把NULL算作一组
* 有的DBMS支持 GROUPBY 表达式，感觉正常应该用CASE

### ORDER BY

* 可以有多个，DESC只作用于那一个
* 会破坏GROUPBY的聚集性，除非也按分组的顺序来排序
* 如果没有分组，可以用SELECT中未使用的列
* 可以用聚合函数和SELECT中设定的别名
* 可以用数字表示SELECT中的列号，但SQL标准表示要废弃
* NULL在ASC时的位置：MySQL MSSQL在开头，PG SQLite在末尾
* 相同字母大小写不同ASC：如 a A z Z，MySQL MSSQL PG为a A z Z，SQLite为 A Z a z

### 窗口函数

* <窗口函数> OVER ([PARTITION BY 列] ORDER BY 列)
* PARTITION也是一种分组，但不是合并为一行，而是算作不同的组（窗口），不同组的窗口函数会“初始化”
* 此处的ORDERBY后跟的是窗口函数计算顺序的基准，不是结果的排序
* 专用窗口函数，直接无参调用，对一窗口内进行排名，遇到值相同时的行为不同：RANK - 1,1,3、DENSE_RANK - 1,1,2、ROW_NUMBER - 1,2,3
* 聚合函数：仍要指定列，关键是每一行的数据范围相当于从第一行到当前行。可在ORDERBY后可加ROWS N PRECEDING M FOLLOWING指定范围为当前行的前N行和当前行和当前行的后M行
* 只能用在SELECT中
* TODO: https://www.sqlite.org/windowfunctions.html 另外SELECT中有WINDOW wnd AS wnd-def语句，在GROPUBY后
* https://zhuanlan.zhihu.com/p/60226935

### Compound集合运算

* UNION [ALL]：直接附加第二个查询的结果；无ALL会自动剔除重复行，性能也更差
* INTERSECT：交集。MySQL不支持
* EXCEPT：差集。MySQL不支持，Oracle为MINUS
* 只能在最后使用一次ORDERBY
* 结构必须相同（名字除外）
* 位于两个SELECT之间，中间不用加分号

### WITH 通用表语句(Common Table Expression)

* 相当于在前面给子查询命个名或相当于一次性视图，使用时仍要FROM和JOIN它
* WITH q1 AS (子查询) [,q2 AS ...] 正常的SELECT/DELETE ... FROM q1
* WITH RECURSIVE fib(a,b) AS (VALUES(1,1) UNION ALL SELECT b,a+b FROM fib where b < 100) SELECT a FROM fib; 把查询结果再次代入到查询子句中继续查询

## DML

* MySQL不能在子查询中SELECT一个表，再按此条件进行更新和删除同一个表的记录。解决办法是再加一层SELECT
* SQLite
  * INSERT/UPDATE OR ABORT/FAIL/IGNORE/REPLACE/ROLLBACK：定义违反约束时的行为，默认ABORT，是曾经ON CONFLICT的简化，另有REPLACE是INSERT OR REPLACE的简化
  * RETURNING id：一般加在insert语句最后，返回创建的id
  * 能开启选项在UPDATE和DELETE中支持LIMIT和ORDERBY

### INSERT

* 默认值：插入NULL不会使用，支持DEFAULT关键字来使用，SQLite除外

```sql
INSERT INTO tb1 (
    col1, col2  -- 可选，但如果省略，则必须每个值都要有
) VALUES (
    col1_val, col2_val
), (col1_val2, col2_val2) -- 多条数据

INSERT INTO tb1 -- tb1必须已存在，否则应用CREATE TABLE AS
SELECT * FROM ... -- ORDERBY无意义

-- UPSERT，SQLite，指定（由于约束）插入失败时的行为
INSERT ...
ON CONFLICT [(col1)] [where ...] DO UPDATE -- 也可以DO NOTHING
SET ...
-- MySQL：INSERT IGNORE、REPLACE INTO、ON DUPLICATE KEY UPDATE
```

### UPDATE

```sql
UPDATE tb1
SET col1 = col1 + 1, -- 此处的col1仅为那个列在当前行的值，不是指一整列
col2 = val2 -- 仅SQLite PG支持(c1,c2)=(v1,v2)
[FROM (SELECT ...) AS val2] -- PG SQLite3.30(2020.8)，还需在WHERE中tb1.id=tb2.id；MSSQL也支持但需要在FROM中再写一次tb1；其实相当于一次JOIN
WHERE ...

-- MySQL的UPDATE FROM
UPDATE tb1 JOIN
(SELECT ...) USING (id)
SET ...
```

### DELETE

* 会走事务，会执行触发器
* InnoDB中DELETE只是标记为已删除
* 清除所有数据而保留表结构：TRUNCATE TABLE，更快，能重置自增数。SQLite没有此语句但清除所有数据时会自动优化；除MSSQL外也能省略TABLE
* MySQL：[LOW_PRIORITY] [QUICK] [IGNORE] LIMIT只删除n条

```sql
DELETE
FROM tb1
WHERE ORDERBY LIMIT

DELETE tb1 -- 仅MySQL？存在多表时只删除指定表中的数据。好像MSSQL也可以
FROM tb1, tb2 -- 老版本的JOIN
WHERE tb1.id=tb2.id
```

### MERGE(MSSQL PG15)

* 将INSERT、UPDATE、DELETE合并为一句

```sql
MERGE INTO dsttb
USING srctb
ON dsttb.id = srctb.id
WHEN MATCHED THEN
UPDATE SET ...
WHEN NOT MATCHED [AND ...] THEN
INSERT VALUES(...)
WHEN NOT MATCHED BY srctb THEN -- 目标表中存在，源表中不存在
DELETE
```

### 其它功能

```sql
-- MySQL，是SELECT INTO OUTFILE的反向，另有mysqlimport命令行工具。数据文件以tab分隔，NULL值用\N表示，换行符用LF
LOAD DATA LOCAL INFILE 'data.txt' INTO TABLE tb1;
```

## DDL

* MSSQL不支持所有的IF NOT EXISTS

### TABLE

* 临时表
  * MSSQL：表名以#开头，当前会话断开后删除，以##开头，所有引用该表的会话断开后删除
  * MySQL SQLite PG：CREATE TEMPORARY TABLE；SQLite PG还可简写为TEMP
  * TODO: 不同连接对临时表的可见性，已知SQLite不同连接是不可见的
* 自增
  * MSSQL：IDENTITY、IDENTITY(from, step)
  * MySQL：AUTO_INCREMENT和表级的AUTO_INCREMENT_OFFSET/INCREMENT
  * SQLite：AUTOINCREMENT。整数主键插入NULL也会自增。两者有一点区别，后者是取最大值+1，如果删除了最后的行再插入就会出现用过的值。其实也可以不定义主键
  * 自增主键用完了可以改BigInt，但其实int43亿行数据早就慢了，SQlite也改不了
  * PG：id serial PRIMARY KEY
* 外键
  * 假设B(b)引用A(a)，A称为ParentTable，a称为ParentKey，B称为Child，A是外键refer to的，B是外键apply to的
  * a必须为主键或UNIQUE，Oracle下必须是唯一约束不能是唯一索引
  * b的值必须存在于a中否则插入失败，但b也可以为NULL
  * a无法修改或删除对应存在于b中的值，但若不存在，则可以删除；推荐给b加索引，因为内部实际上做了SELECT
  * MySQL SQlite支持ON UPDATE/DELETE CASCADE/SET NULL/SET DEFAULT使得a更新/删除时对应更新/删除b或设为NULL或默认值（仍需满足） TODO: 测试PG和MSSQL
  * SQLite
    * 默认未开启外键检查，PRAGMA foreign_keys=on;开启
    * 定义时加上DEFERRABLE INITIALLY DEFERRED或用PRAGMA defer_foreign_keys=on能使得检查推迟到提交时
    * 定义表时允许引用A中不存在的列甚至不存在的表，就好像未开启外键约束一样，只在插入删除时检查
    * 支持FOREIGN KEY (a,b) REFERENCES (c,d)
  * MySQL
    * 若不写FOREIGN KEY只写REFERENCE，它什么都不做，静默忽略
* 查询表的元数据
  * MySQL：SHOW TABLES有哪些表; DESCRIBE tb1表的列和类型; SELECT TABLE()当前表？; SHOW CREATE TABLE创建表的SQL语句; CREATE TABLE tgttb LIKE srctb按指定表的结构创建另一个表
* 只有MSSQL支持trailing comma TODO: 测试Select
* UNIQUE的列，其它数据库都允许存在多个NULL，MSSQL不允许
* 默认值：ALTER TABLE新添加有默认值的列，已存在的行的那一列，MySQL PG SQLite为那个默认值，MSSQL为NULL，再加NOT NULL则为默认值
* SQLite支持在某些约束后定义ON CONFLICT ROLLBACK/ABORT/FAIL/IGNORE/REPLACE，默认是ABORT即终止语句但保留当前事务，如果没有“活动事务”则相当于ROLLBACK，FAIL相当于在本句之前提交掉，IGNORE相当于本句不存在会继续执行后面的
* CHECK约束：MySQL8后某个版本才支持

```sql
CREATE TABLE [IF NOT EXISTS] tb1 ( -- MSSQL除外
    col1 type PRIMARY KEY,
    col2 type NOT NULL DEFAULT n UNIQUE, -- default加括号(表达式或函数)
    col3 type REFERENCES tb2(col1), -- MySQL能写在此处却无效
    -- 表约束
    INDEX/UNIQUE ndx1(col2 [desc], col3), -- 索引的顺序应和常用的ORDERBY顺序一致，主键也支持指定顺序
    FOREIGN KEY (col3) REFERENCES tb2(col1),
    [CONSTRAINT cons1] CHECK(col1>0 and col2>0), -- 在多个列上定义约束，如果是单列也可以放在列后；用IN运算符可起到ENUM的效果
)
DROP TABLE [IF EXISTS] tb1; -- 如果被外键引用了则无法直接删除 TODO: 怎么强删
CREATE TABLE tb2 -- MSSQL除外；SQLite不保留主键和约束
AS SELECT * FROM tb1;

-- 修改表名（MSSQL除外）
ALTER TABLE tb1 RENAME TO tb2
-- 添加列，注意不需要加括号；只有MSSQL不支持写成ADD COLUMN；MySQL支持AFTER col1指定在某一列后添加
ALTER TABLE tb1 ADD col3 type, 表约束
-- 重命名列（MSSQL除外）
ALTER TABLE tb1 RENAME COLUMN col1 TO col2
-- 删除列
ALTER TABLE tb1 DROP COLUMN col1
-- SQLite的ALTER TABLE只支持以上几项，且删除列具有一些限制，如存在索引或被约束和视图引用了就不能删

-- 修改列类型和约束，如果列中已有数据会尝试转换，但各数据库容忍程度不同
ALTER TABLE tb1 ALTER COLUMN col1 type [NOT NULL] -- MSSQL, Oracle，只有非空约束可以在这里改；PG真的要加TYPE
ALTER TABLE tb1 MODIFY col1 ... -- MySQL，可改任何约束；还有一种CHANGE语句，也能用来修改列名，但若不修改则要写两遍原列名

-- 添加约束，如果已有数据不符合会失败；修改约束都要先DROP再ADD
ALTER TABLE tb1 ADD [CONSTRAINT cons1] UNIQUE/PRIMARY KEY/FOREIGN KEY/CHECK(col1) -- 通用，PG除外。cons1只是命名
ALTER TABLE tb1 ADD DEFAULT n FOR col1 -- MSSQL，CREATE DEFAULT弃用了
ALTER TABLE tb1 ALTER COLUMN col1 SET NOT NULL/DEFAULT n -- MySQL, PG

-- 创建非聚集索引
CREATE [UNIQUE] INDEX [IF NOT EXISTS] ndx1 -- MySQL MSSQL除外
ON tb1(col1, col2) -- 复合索引，在WHERE等中使用时也要按顺序，如单独查询col2不生效
WHERE ... -- 部分索引，MySQL Oracle。与UNIQUE结合可达到普通唯一约束做不到的指定条件
-- 重建索引
ALTER INDEX ALL ON tb1 REBUILD/REORGANIZE -- MSSQL，前者更彻底但会上锁，后者资源消耗小
REINDEX [tb1] -- SQLite，只在collation顺序变化或部分索引含有函数且函数内容变化后使用
-- 删除索引
DROP INDEX [IF EXISTS] ndx1 -- MySQL Oracle除外
ON tb1 -- MSSQL MySQL一定要有，PG SQLite Oracle一定不能有

-- 生成列(SQLite 3.31,2020.1)，只读，值来自于表达式和标量函数。能有除默认值以外的约束，不能是主键，只能使用本行的列，可以使用另一个生成列但不能循环或自我引用，不能直接使用ROWID
d INT AS (a*abs(b)) -- 每读此行就计算一次
e TEXT AS (substr(c,b,b+1)) STORED -- 插入此行时计算唯一一次，无法用ALTER TABLE添加
```

### VIEW

* 取表或其他视图的一部分变成一个新的“表”，实际保存的是SELECT语句
* 标准对于更新的限制：不能有GROUPBY、HAVING、DISTINCT，只能FROM一张表
* MSSQL：不支持ORDERBY，不支持SELECT中用表达式
* SQLite：支持TEMP；不支持直接修改但能通过定义INSTEAD OF触发器修改
* MySQL：不支持TEMP VIEW
* TODO: 每次使用都会重新查询？索引？

```sql
CREATE VIEW [IF NOT EXISTS] viewname
[(col1, col2)] -- 如果不定义列名，SELECT的就必须全都用AS，否则万一修改了难以保证一致性。TODO: 已知SQLite支持，测试其他是否支持
AS SELECT ...

DROP VIEW [IF EXISTS]
```

### DATABASE

* CREATE DATABASE [IF NOT EXISTS] db1
  * MySQL在Linux下对数据库名是大小写敏感的
* SHOW DATABASES
* USE db1
* SELECT DATABASE()显示当前使用的数据库

## TCL

### 事务

* 开启
  * SQLite、PG：BEGIN和END(对应COMMIT)
  * MSSQL：BEGIN TRAN
  * MSSQL、SQLite、PG：BEGIN TRANSACTION
  * MySQL：START TRANSACTION
  * Oracle无需开启，相当于一开始就开启了，必须手动提交才生效
  * COMMIT、ROLLBACK
  * 其余数据库在不显式开启事务时默认一句就提交一次
  * MSSQL在关闭自动提交后也变得像Oracle一样了，另外两个GO之间的代码会先编译一遍，如果有错误则不会执行
* SQLite
  * SAVEPOINT spname 相当于事务中的事务，必须命名
  * RELEASE spname 类似于COMMIT，使得不能回滚到指定及之后的点，也可看做把内层的保存点合并到外层，真的事务没有提交，后序也可能回滚；但如果最开始就是用SAVEPOINT开启的事务，则最后一层RELEASE会提交
  * ROLLBACK TO spname 回滚到指定点
  * BEGIN IMMEDIATE/EXCLUSIVE：进入事务时就加写锁
* MSSQL
  * 设置隔离级别：SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;，处在连接级别
  * 启用快照隔离：ALTER DATABASE db1 SET ALLOW_SNAPSHOT_ISOLATION/READ_COMMITTED_SNAPSHOT ON
  * 悲观锁：SELECT * FROM xxx WITH (UPDLOCK)
  * 默认情况下会一直等待，不会超时，除非设置LOCK_TIMEOUT

### 存储过程

* 与DBMS密切相关，移植性差，SQLite不支持
* 不适合需求经常变的，因为表会变，就要改所有对应的存储过程
* 难横向扩展，只能纵向扩展
* 适合处理场景固定的复杂事务，不要求并发性
* 能消除不必要的网络IO，适合做集合运算
* 调试困难

```
CREATE PROCEDURE proc1(@var1[=default_value1] [output], ...)
AS SELECT/INSERT ...

* exec *ProcedureName LiteralValue/*@*Variable1* = ..., @*OutputVariable1*, ...
* select *ColumnName* = @*OutputVariable1, ...*
```

### 触发器 TODO:

* 触发器种类：DML触发器（insert、update、delete）、DDL触发器（create、alter、drop）、登陆触发器
* 触发器是自动执行的，create trigger必须是批处理（两个go之间）中的第一个语句，且其他语句都将被解释为create trigger语句定义的一部分（因为没有括号）
* 不能在临时表和系统表上创建触发器，但可引用临时表、不可引用系统表；可引用当前数据库以外的对象，但只能创建在当前数据库里
* 下列语句不能创建触发器：create/alter/drop/load/restore database、disk init/resize、load/restore log、reconfigure
* 每个触发器有两个特殊的表：inserted和deleted，数据插入和删除的时候会复制一份到表中，update触发器同时使用者两个
* 比如保证学生往选课表里添加记录时，学号必须存在于学生基本信息表里

```
CREATE TRIGGER tg AFTER DELETE ON tb1 BEGIN
<DML> UPDATE child SET trackartist = 0 WHERE trackartist = old.artistid;
END;
```

* create/alter/drop/enable/disable trigger TriggerName on
* DML：Table|View for/instead of {insert, update, delete} as ...
* DDL：all server|database for {create, alter, drop, ...} as ...
* https://www.runoob.com/sqlite/sqlite-trigger.html https://www.sqlite.org/lang_createtrigger.html
* DROP TRIGGER [IF EXISTS] tn；关联的表删除时自动删除；SQLite一定没有on tb
* 查看update和delete在创建触发器中的Restrictions

### Rule(MSSQL)

* CREATE RULE r1 as ...
* exec sp_bindrule r1, obj
* drop rule r1
* 与约束类似，但是方便重用
* 完整性约束：一条规则可用五元组(Data, Operation, Assertion, Condition, Procedure)表示，例如“学号(SNo)不能为空”这条约束，数据对象D是SNo属性，O是当用户插入时执行这条规则，A是不能为空，C是A可作用于所有D，P是拒绝用户的请求

## DCL

* 在删除角色之前必须删除该角色的成员

```sql
CREATE ROLE r1 [*AUTHORIZATION*]

create/alter/drop user *UserName*

grant [系统权限/角色] to [用户名/角色]
revoke [系统权限/角色] from [用户名/角色]
grant all/对象权限 on [表名] to [用户/public] [with grant option]（允许它将此权限授予其他用户）
revoke ... from [CASCADE]（级联，也收回它授予其它用户的权限）

audit select, insert, delete, update on *TableName* whenever succecssful
no audit all on *TableName*

grant SELECT/INSERT/UPDATE(col1)/ALTER/ALL PRIVILEGES ON TABLE t1,t2 To user1, user2
```

### MySQL

```sql
CREATE USER 'username'@'hostname' IDENTIFIED BY 'passwd'; -- hostname用%表示任意客户端ip，不加时默认就是；当用户名没有空格和-等字符时可不加引号
GRANT ALL ON db1.* to 'username'@'%';

select user(); SELECT CURRENT_USER();
```

## JSON

### SQLite

* 从3.38(2022.2)开始内建，提供了JSON类型
* 参数为*json*的函数，能解析字符串为内部的json对象，一般作为第一个参数；内部对象在最后输出的时候为TEXT；不应用它检验是否合法，应在WHERE中用json_valid()
* 参数为*value*的函数，若传字符串，即使对应有效的json，也不会解析。如json_object('k','[1,2]')为{"k":"[1,2]"}而非"k":[1,2]，json_array(1,'2')为[1,'2']。出现在“构造函数”和作为“值”语义的参数
* path：以$开头的字符串，后跟.X或[N]。表达数组长度用#，可+-运算，支持负索引
* ->和->>运算符：*json*->*path*，前者解析为内部对象，后者取出为SQL类型；另外path对于取单层还支持简写为不加$的'X'和N。如`'[1,2]'->'$'`与json('[1,2]')效果一样，`'[1,2]'->>'1'`是INTEGER的2而非TEXT，->>'$'相当于转换为字符串
* json_set(json,path,value[,path,value]) json_replace() json_insert()：set就是一般的赋值，不存在时会创建；replace仅覆盖，不存在时不会创建；insert插入到某个位置，原位置的对象后移，不会覆盖；数组插入#位置相当于append
* json_patch(json,json) json_remove() json_type()：略
* json_quote()：将可能不合理的字符串转换为json能接受的字符串，因为各种*json*参数包括->会去解析字符串；如果合法或者已是内部对象，就什么也不做
* 遍历：json_each() json_tree()，用在FROM中，返回同函数名的表，key对于数组是索引 value type atom类似于值但对于数组和对象是空 fullkey路径
* 聚合函数：json_group_array() json_group_object()
