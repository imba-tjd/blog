---
title: SQL实例
---

不存在的显示0
-------------

> 查询数据库有几个用户？每个用户都拥有几张表（考虑可能有些用户是无表的，无表显示0）？

-   从查询用户的表中左外连接所有表的表，以用户名分组，但计算表名的数量

<!-- -->

    select USERNAME,COUNT(TABLE_NAME) from DBA_USERS
       left join DBA_TABLES on DBA_TABLES.OWNER = DBA_USERS.USERNAME
       group by USERNAME

合并两个同类型的表
------------------

同Key，Value相加：

    select C1, sum(C2)
    from (select * from T1 union all select * from T2) as t
    group by C1
