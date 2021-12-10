---
title: SQL
---

## 杂项语法

* 注释，标准只有单行注释`--`，应该也可用于行尾；应该一般都支持`/* */`，MySQL和SQLite支持`#`
* 命名：一般不加s，列名小写，表名首字母大写，关键字全大写
* 写完一句后要打分号

## 运算

* “表达式”：+-*/、括号、常数、列名、标量子查询、聚合函数、CASE；一般可用在SELECT、WHERE、HAVING中
* WHERE中不能用聚合函数因为那必须在分组后生效，ORDERBY中可以用聚合函数和SELECT中设定的别名
* 通用函数：ABS、ROUND、COALESCE(返回第一个不为NULL的参数)

### CASE

* 是一个表达式，可用于分类，也可用于CHECK中
* when里的也可以是一般的带比较运算符的条件表达式，此时可不写CASE后的表达式，then里的可以是子查询等表达式
* 非标准的简化：MySQL有IF函数，Oracle有DECODE函数

```sql
SELECT co1
, CASE col1
    WHEN 'A' THEN 'it's a'
    ELSE 'other' -- 不写相当于ELSE NULL
END AS col2
FROM ...

-- 逻辑中的 x->y “蕴含”。注意x为假时结果为真，不只有x和y都为真时才为真
CASE x
WHEN a
    THEN CASE y
    WHEN b
        THEN 1
        ELSE 0
    END
ELSE 1
END

-- 行列互换，一维转二维：[(A,B(b1),V(v1)),(A,B(b2),V(v2))] -> [A,b1,b2]，其中A为主键，b1为B的值但变为列，V隐含为表里的数据
select A,
max(case B when b1 then v1 else null end) as b1, -- 当是b1时把值设为v1，否则设为0，之后取max就只会取到v1
max(case B when b2 then v2 else null end) as b2
from tb
group by A;
```

### 集合运算

* 只能在最后使用一次ORDERBY
* 结构必须相同（名字除外）
* 位于两个SELECT之间
* UNION [ALL]：直接附加第二个查询的结果；无ALL会自动剔除重复行，性能也更差
* INTERSECT：交集。MySQL不支持
* EXCEPT：差集。MySQL不支持，Oracle为MINUS

## DQL

* 顺序：FROM-WHERE-GROUPBY-HAVING-SELECT-DISTINCT-UNION-ORDERBY-LIMIT
* 聚合函数，会自动忽略为NULL的：AVG、COUNT、MIN、MAX、SUM
* COUNT(*)：行数，不忽略NULL
* 通用表语句，相当于在前面给子查询命个名或相当于一次性视图：WITH q1 AS (子查询) [,q2 AS ...] 正常的SELECT/DELETE ...；使用时还需FROM一遍，不能直接放在WHERE等中

### SELECT

* 经过计算后的内容一般要用AS命名
* DISTINCT：在SELECT后，不是每列前都加，对一整行生效，但聚合函数参数中可以加；会把NULL算作一个值；PG支持DISTINCT ON(col1)
* 选取前n条：SQLite和MySQL在最后`LIMIT n OFFSET m`或`LIMIT m,n`；MSSQL`SELECT [DISTINCT] TOP n [PERCENT]`，跳过很麻烦；Oracle`WHERE ROWNUM<=n`
* SELECT * INTO tb2 FROM ...：目标表不能已存在，会自动创建；最好只在MSSQL中作为不支持CREATE TABLE AS的替代
* FROM的可以是子查询且此处**必须**命名，此时可以有多列
* 关联子查询：外部查询的每一行都会执行一次子查询。例如想选择价格大于所属类别的平均价格的条目，其中计算某一类的平均价格本来需要分组和聚合函数，但最终目标又是选择条目，则需要在子查询中用WHERE过滤与外层相同的类型，就会用到外层FROM的表，分组却不必要了

### JOIN

* 如果存在同名的列，SELECT中要加表名，可用tb1.*表示第一个表的所有列
* CROSS JOIN：笛卡尔积，参数只需加表名，无需on；MySQL允许有on但直接忽略
* INNER JOIN tb2 ON tb1.col1 = tb2.col2：等价于老版本标准中的FROM t1, t2 WHERE ...
* LEFT/RIGHT/FULL JOIN：左外连接就是完全保留左边的，右边的如果没有就为NULL；SQLite只支持left
* LEFT JOIN WHERE tb2.id IS NULL可查询仅在左边的，即减去共有部分
* SELF JOIN：选择来自于相同城市的人、时间间隔
* 内连接不会把col1.NULL与col2.NULL看作相等，会忽略

### WHERE

* 等于用`=`，标准的不等于为`<>`
* NOT、AND、OR：与一般的编程语言一样；但SQLite和MySQL还支持`col1 = 1 OR 2`这种类似Py的写法
* IS NULL、IS NOT NULL；MySQL和SQLite对于非空和BOOL还支持类似Py的写法
* [NOT] BETWEEN 1 AND 9：测试的几个都为闭区间
* IN (1,2,3或单列子查询)、NOT IN：相当于多个判断等于的OR；`3 IN (1,2,NULL)`和NOT IN结果都是NULL，用子查询时尤其要注意
* [NOT] LIKE：默认不区分大小写，PG除外。用%表示任意字符，用_表示单个字符；只有MSSQL支持`[]`；SQLite支持GLOB且区分大小写
* ANY、ALL：前加比较运算符，后跟子查询。用比较运算符时可分情况改成聚合函数MIN和MAX，=ANY和!=ALL可换成IN和NOT IN，但=ALL和!=ANY没有等价的
* EXISTS、NOT EXISTS：前面不跟列，后跟子查询，判断是否返回至少一行数据，注意null也算存在
* 拼接的时候常加`WHERE 1=1`，方便之后加AND，但其实用1就好了

### GROUP BY

* SELECT的列必须也在GROUPBY中，或是聚合函数；用了聚合函数即使没有GROUPBY也不能SELECT普通的列；不能SELECT *
* 分组后一行就相当于一组，聚合函数会对每一组用一遍
* 会把NULL算作一组
* HAVING的只能是常数、分组的列、聚合函数
* SQLite能选择不在GROUPBY中的列，结果是聚合函数那一列的每一行值都相同，不再压缩为一行

### ORDER BY

* 可以有多个，desc只作用于那一个
* 会破坏GROUP BY的聚集性，除非也按分组的顺序来排序
* 如果没有分过组，可以使用SELECT中未使用的列
* 可以使用SELECT中的别名
* null在asc时在开头

### 窗口函数

* <窗口函数> OVER ([PARTITION BY 列] ORDER BY 列)
* PARTITION也是一种分组，但不会合并为一行，不同组的窗口函数会“初始化”；此处的ORDERBY后跟的是窗口函数计算顺序的基准，不是结果的排序
* 专用窗口函数，直接无参调用，对一窗口内进行排名，遇到值相同时的行为不同：RANK（1,1,3）、DENSE_RANK(1,1,2)、ROW_NUMBER(1,2,3)
* 聚合函数：仍要指定列，关键是每一行的数据范围相当于从第一行到当前行。可在ORDERBY后用ROWS N PRECEDING M FOLLOWING指定范围为当前行的前N行和当前行和当前行的后M行
* 只能用在SELECT中

## DML

* MySQL：不能在子查询中SELECT一个表，再按此条件进行更新和删除同一个表的记录。解决办法是再加一层SELECT

### INSERT

* 数据值支持DEFAULT关键字（SQLite除外），若表未指定则为NULL

```sql
INSERT INTO tb1 (
    col1, col2  -- 可选，但如果省略，则必须每个值都要有，即使允许NULL也不行
) VALUES (
    col1_val, col2_val
), (col1_val2, col2_val2) -- 多行数据

INSERT INTO tb1 -- tb1必须已存在，否则就用CREATE TABLE AS
SELECT * FROM ... -- ORDERBY无意义
```

### UPDATE

```sql
UPDATE tb1
SET col1 = val1, col2 = val2 -- PG支持(c1,c2)=(v1,v2)
WHERE ...
```

### DELETE

* 会走事务，会执行触发器
* InnoDB中DELETE只是标记为已删除
* 清除所有数据而保留表结构用TRUNCATE TABLE，更快，且能重置自增数；SQLite除外

```sql
DELETE
FROM tb1
WHERE ...

DELETE tb1
FROM tb1, tb2 -- 老版本的JOIN
WHERE tb1.id=tb2.id AND ...
```

### MERGE(MSSQL)

* 将INSERT、UPDATE、DELETE合并为一句

```sql
MERGE INTO tgttb
USING srctb
ON tgttb.id = srctb.id
WHEN MATCHED THEN
UPDATE SET ...
WHEN NOT MATCHED [AND ...] THEN
INSERT VALUES(...)
WHEN NOT MATCHED BY srctb THEN -- 目标表中存在，源表中不存在
DELETE
```

### UPSERT(SQLite)

* https://www.sqlite.org/lang_upsert.html

## DDL

### TABLE

* 临时表：MSSQL表名以#开头，当前会话断开后删除，以##开头，所有引用该表的会话断开后删除；MySQL和SQLite CREATE TEMPORARY TABLE AS SELECT，SQLite还可简写为TEMP
* 自增：MSSQL为identity[(from, step)]，MySQL为AUTO_INCREMENT和表级的AUTO_INCREMENT_OFFSET/INCREMENT；SQLite为AUTOINCREMENT，但其实整数主键插入NULL就会自动自增，相当于取最大值加1，当删除了最后的行再插入就会出现用过的值，前者保证不重用，但一般不需要，甚至其实没必要定义整数主键，会自动用。自增主键用完了可以改BigInt，但其实int43亿行数据早就慢了
* 唯一约束：MSSQL不允许有多个NULL，但其它数据库允许，只对非NULL的不允许有多个

```sql
CREATE TABLE [IF NOT EXISTS] tb1 ( -- MSSQL除外
    col1 type PRIMARY KEY,
    col2 type NOT NULL DEFAULT n, -- 默认值只有插入时才会使用，ALTER TABLE添加有默认值的列会发现全是NULL
    col3 type FOREIGN KEY REFERENCES tb2(col1), -- tb2.col1必须为主键；MSSQL和Oracle和SQL标准能定义在这里
    -- 表约束
    index ndx1(col2 [desc]),
    FOREIGN KEY (col3) REFERENCES tb2(col1) [ON UPDATE/DELETE CASCADE/RESTRICT/SET NULL], -- 被引用行更新/删除时引用行也（/禁止）更新删除
    [CONSTRAINT cons1] CHECK(col1>0 and col2>0), -- 在多个列上定义约束，如果是单列也可以放在列后；用IN运算符可起到ENUM的效果
)
CREATE TABLE tb2 -- MSSQL除外；相比于SELECT INTO更明确，会保留非空约束
AS SELECT * FROM tb1;
DROP TABLE [IF EXISTS] tb1; -- 都可用

-- 修改表名（MSSQL除外）
ALTER TABLE tb1 RENAME TO tb2
-- 添加列，注意不需要加括号；只有MSSQL不支持写成ADD COLUMN；MySQL支持AFTER col1指定在某一列后添加，否则和其余的都在最后添加
ALTER TABLE tb1 ADD col3 type, 表约束
-- 重命名列（MSSQL除外）
ALTER TABLE tb1 RENAME COLUMN col1 TO col2
-- SQLite的ALTER TABLE只支持以上三项
-- 删除列
ALTER TABLE tb1 DROP COLUMN col1

-- 修改列类型和约束，如果列中已有数据会尝试转换，但各数据库容忍程度不同
ALTER TABLE tb1 ALTER COLUMN col1 type [NOT NULL] -- MSSQL, Oracle，只有非空约束可以在这里改；PG真的要加TYPE
ALTER TABLE tb1 MODIFY col1 ... -- MySQL，可改任何约束；还有一种CHANGE语句，也能用来修改列名，但若不修改则要写两遍原列名

-- 添加约束，如果已有数据不符合会失败；修改约束都要先DROP再ADD
ALTER TABLE tb1 ADD [CONSTRAINT cons1] UNIQUE/PRIMARY KEY/FOREIGN KEY/CHECK(col1) -- 除PG外通用，cons1只是命名
ALTER TABLE tb1 ADD DEFAULT n FOR col1 -- MSSQL，CREATE DEFAULT弃用了
ALTER TABLE tb1 ALTER COLUMN col1 SET NOT NULL/DEFAULT n -- MySQL, PG

-- 创建非聚集索引
CREATE INDEX ndx1 ON tb1(col1, col2) -- 复合索引，使用时也要按顺序等限制，如单独查询col2不生效
-- 重建索引
ALTER INDEX ALL ON tb1 REBUILD/REORGANIZE -- MSSQL，前者更彻底但会上锁，后者资源消耗小
REINDEX tb1 -- SQLite
-- 删除索引
DROP INDEX ndx1 ON tb1
-- 唯一筛选索引：一个队只能有一个队长，只当is_team_leader为真时才对team_id有唯一约束
-- 不能对team_id或者is_team_leader进行唯一约束因为有多个普通队员有同样的team_id
CREATE UNIQUE INDEX team_leader ON person(team_id) WHERE is_team_leader;
```

### VIEW

* 取表或其他视图的一部分变成一个新的“表”，实际保存的是SELECT语句，不能使用ORDERBY，每次使用都会重新查询？
* 只能进行有限程度的更新，会转换成对基本表的更新。有GROUPBY或有DISTINCT时无法更新，只FROM了一张表可以更新
* SQLite：支持temp view，不能修改

```sql
CREATE VIEW viewname
AS SELECT ...
```

## TCL

### 事务

* MSSQL、SQLite、PG：BEGIN TRANSACTION
* MySQL：START TRANSACTION
* Oracle：无需开始事务的命令
* MSSQL：SAVE TRANSACTION
* COMMIT、ROLLBACK
* 其实所有的数据库在连上时就自动开启事务了，只是Oracle默认必须手动提交才生效，MSSQL遇到GO生效，有的默认一句就提交一次
* SQLite: SAVEPOINT、RELEASE

### 存储过程

* 与DBMS密切相关，移植性差
* 不适合需求经常变的，因为表会变，就要改所有对应的存储过程
* 难横向扩展，只能纵向扩展
* 没有日志
* 适合处理场景固定的复杂事务，不要求并发性
* 能消除不必要的网络IO，适合做集合运算
* 建议只用来存数据，不要写业务逻辑

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

* create/alter/drop/enable/disable trigger TriggerName on
* DML：Table|View for/instead of {insert, update, delete} as ...
* DDL：all server|database for {create, alter, drop, ...} as ...
* https://www.runoob.com/sqlite/sqlite-trigger.html

### Rule(MSSQL)

* CREATE RULE r1 as ...
* exec sp_bindrule r1, obj
* drop rule r1
* 与约束类似，但是方便重用
* 完整性约束：一条规则可用五元组(Data, Operation, Assertion, Condition, Procedure)表示，例如“学号(SNo)不能为空”这条约束，数据对象D是SNo属性，O是当用户插入时执行这条规则，A是不能为空，C是A可作用于所有D，P是拒绝用户的请求

## DCL

* 在删除角色之前必须删除该角色的成员；如果角色有用安全对象，必须首先转移这些安全对象的所有权


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

## 其它

* 在线测试：https://dbfiddle.uk 有点问题，有时语法不支持也不会报错；https://www.db-fiddle.com js的cdn打开太慢；http://sqlfiddle.com/ 不新；https://sqliteonline.com/ 感觉比较好

## 参考

* 《SQL基础教程》

### TODO

* https://www.sqlite.org/windowfunctions.html
* https://www.sqlite.org/gencol.html
* 唯一约束是否会自动创建唯一索引？
