---
title: MySQL相关知识点
sticky: 36
categories: 数据库
index_img: /assert/mysql.jpg
img: 
---

#### 递归function,父子查询

创建函数时注意分隔符，mysql遇到分号就执行，在创建函数的时候容易报错（use near ' 'at line）,可使用  **delimiter //**  定义分隔符

delimiter //  (创建函数报错时添加这个)

**getParentList**

```sql
CREATE FUNCTION `getParentList`(rootId varchar(100))
RETURNS varchar(1000)
BEGIN
DECLARE fid varchar(100) default '';
DECLARE str varchar(1000) default rootId;
WHILE rootId is not null do
  SET fid =(SELECT parentid FROM treeNodes WHERE id = rootId);
  IF fid is not null THEN
	SET str = concat(str, ',', fid);
	SET rootId = fid;
  ELSE
	SET rootId = fid;
  END IF;
END WHILE;
return str;
END;-- (这样只有当//出现之后，MySQL解释器才会执行这段语句)

delimiter ; -- (改回默认的 MySQL delimiter：“;”）
```
使用：
```sql
select getParentList('001001');

select * from tbl where FIND_IN_SET(id,getParentList('001001'))
```

**getChildList**

```sql
-- 在root用户下创建 %代表任何ip都可以连接
CREATE DEFINER=`root`@`%` FUNCTION `getChildList`(rootId varchar(100)) RETURNS varchar(2000) CHARSET utf8
BEGIN
DECLARE str varchar(2000);
DECLARE cid varchar(1000);-- 注意设置的长度,过短的话当树很高的时候容易导致查询数据不全
SET str = '$';
SET cid = rootId;
WHILE cid is not null DO
  SET str = concat(str, ',', cid);
  SELECT group_concat(id) INTO cid FROM fke_supervision_template where FIND_IN_SET(parent_id, cid) > 0;
END WHILE;
 RETURN str;
END
```
使用：
```sql
select getChildList('001001001');

select * from tbl where FIND_IN_SET(id,getChildList('001001001'))
```


**FIND_IN_SET(str,strlist)**
str 要查询的字符串
strlist 字段名 参数以”,”分隔 如 (1,2,6,8)
查询字段(strlist)中包含(str)的结果，返回结果为null或结果
SELECT FIND_IN_SET('b', 'a,b,c,d');
因为b 在strlist集合中放在2的位置 从1开始

```sql
select * from treenodes where FIND_IN_SET(id, '1,2,3,4,5');
```
使用find_in_set函数一次返回多条记录，id 是一个表的字段，然后每条记录分别是id等于1，2，3，4，5的时候，有点类似in（集合）
```sql
select * from treenodes where id in (1,2,3,4,5);
```
**当使用find_in_set配合递归函数完成递归查询时，有时会查询很慢，此时sql可以这样优化**
```sql
select * from (select ... from t1 join t2 on ...)temp,(select getChildList('10001') as cids) where find_in_set(id,cids);
```
[MySQL 中 FIND_IN_SET 函数执行非常慢的某种写法](https://blog.csdn.net/wokelv/article/details/78915502)

**MySQL8.0使用CTE完成递归查询**
```sql
WITH RECURSIVE cte AS (
SELECT  ID,  PID, NAME, LEVEL, Type  FROM tmp_zjs  WHERE  ID = '102' UNION ALL 
SELECT sou.ID, sou.PID, sou.NAME, sou.LEVEL, sou.Type  FROM 
cte c INNER JOIN tmp_zjs sou ON c.ID = sou.PID  )  

SELECT * FROM	cte
```
[MySQL使用递归CTE遍历分层数据](https://www.begtut.com/mysql/mysql-recursive-cte.html)
[MySQL CTE(公共表表达式)](https://www.yiibai.com/mysql/cte.html)
[一种避免递归查询所有子部门的树数据表设计与实现](https://mp.weixin.qq.com/s/ymAjaaKKwgbpkHOZ9tgbJw)


#### 触发器

```sql
CREATE TRIGGER trigger_name
trigger_time
trigger_event ON tbl_name
FOR EACH ROW
trigger_stmt
-- 其中：
-- trigger_name：标识触发器名称，用户自行指定；
--trigger_time：标识触发时机，取值为 BEFORE 或 AFTER；
-- trigger_event：标识触发事件，取值为 INSERT、UPDATE 或 DELETE；
-- tbl_name：标识建立触发器的表名，即在哪张表上建立触发器；
-- trigger_stmt：触发器程序体，可以是一句SQL语句，或者用 BEGIN 和 END 包含的多条语句。
```

#### 存储过程(游标)

```sql
CREATE DEFINER=`root`@`%` PROCEDURE `NewProc`(IN a VARCHAR(50),OUT b VARCHAR(50))
BEGIN
declare my_id varchar(32); -- 自定义变量1
declare my_name varchar(50); -- 自定义变量2
DECLARE done INT DEFAULT FALSE; -- 自定义控制游标循环变量,默认false
DECLARE My_Cursor CURSOR FOR ( SELECT id, area_name FROM fke_area where id < 20 order by id desc); -- 定义游标并输入结果集
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE; -- 绑定控制变量到游标,游标循环结束自动转true

OPEN My_Cursor; -- 打开游标

myLoop: LOOP -- 开始循环体,myLoop为自定义循环名,结束循环时用到
	FETCH My_Cursor into my_id, my_name; -- 将游标当前读取行的数据顺序赋予自定义变量
	IF done THEN -- 判断是否继续循环
		LEAVE myLoop; -- 结束循环 (break) iterate myLoop(continue)
	END IF;
  -- 自己要做的事情,在 sql 中直接使用自定义变量即可
	IF (my_id < 7 ) THEN
		set b = my_name;
	ELSEIF(a='a') THEN -- 不用"=="做判断,用'='
		set b = 'q';
	ELSE
		set b = 'qq';
	END IF;
  -- COMMIT; -- 提交事务
END LOOP myLoop; -- 结束自定义循环体
CLOSE My_Cursor; -- 关闭游标
END
```
调用:set @a = 'a';set @b = 'b';call NewProc(@a,@b);
select @b;


#### 函数

**时间函数**

```sql
select now(),curdate(),curtime(); -- 2008-12-29 16:25:46，2008-12-29 ，16:25:46

date_add(date,INTERVAL expr type) -- 函数向日期添加指定的时间间隔。

select date_add(curdate(),INTERVAL 15 DAY) -- 15天后

date_sub(date,INTERVAL expr type) -- 函数从日期减去指定的时间间隔。

select date_sub(curdate(),INTERVAL 15 DAY) -- 15天前

select date_format(now(),'%y-%m-%d')

period_diff(P1,P2) -- 返回周期P1和P2之间的月数。 P1和P2格式为YYMM或YYYYMM。注意周期参数 P1 和 P2 都不是日期值

select * from 表名 where period_diff( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1 --上月

from_unixtime(时间戳) -- 时间戳转日期

unix_timestamp(日期) -- 日期转时间戳

select to_days('1997-10-07'); -- 729669 可以用来查看时间间隔多少天，就是从0年开始 到1997年10月7号之间的天数

select str_to_date("2010-11-23 14:39:51",'%Y-%m-%d %H:%i:%s');
```

**数值函数**
```sql
round(x,y) -- 四舍五入小数点到y位，如果y为负数，则保留x数值小数点从左起y位

round(x) -- 四舍五入保存整数

truncate(x,y) -- 不进行四舍五入

floor(x) -- 不四舍五入保存整数 x为float类型

left(str, length) -- 从左开始截取字符串 说明：left（被截取字段，截取长度） 例：select left（content,200）

right(str, length) -- 从右开始截取字符串 说明：right（被截取字段，截取长度） 例：select right（content,200）
```

**字符函数**
```sql
substring --（被截取字段，从第几位开始截取） // 截取字符串 索引从1开始

substring --（被截取字段，从第几位开始截取，截取长度）

-- substr()与substring()一样

group_concat()--函数通常与group by语句在一起使用,用于返回group by分组后的组的多条数据，用","合开 例:select name,group_concat(number) from test group by name
```


#### exists
表A（小表），表B（大表）
```sql
select * from A where c in (select c from B) -- ，用到了A表上c列的索引；
select * from A where exists(select c from B where c=A.c) -- 效率高，用到了B表上c列的索引。
-- 相反的
select * from B where c in (select c from A) -- 效率高，用到了B表上c列的索引
select * from B where exists(select c from A where c=B.c) -- 效率低，用到了A表上c列的索引。
```
[MySQL 多表联合查询有何讲究？](https://mp.weixin.qq.com/s/O121Y0sywAtt4jk9vy4b0w)


#### 复制表,但是不保留表中数据:

```sql
create table 表名 as select 列名 from 表名 where 1=2;
create table 表名(列名) as select 列名 from 表名 where 1=2;
--复制数据
insert into 表名 ( ) select a.* from ()a
```

#### sql执行顺序
```
from
on
join
where
group by
(avg,sum)
having
select
distinct
union
order by
limit
```
[面试官：你来讲讲一条查询语句的具体执行过程](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247490897&idx=1&sn=e5eaa58d426255c2c1401578d5606138&source=41#wechat_redirect)
[一文读懂MySQL查询语句的执行过程](https://mp.weixin.qq.com/s/dLJet2geb-dQ9hLE0oL6Qw)
[MySQL 的执行过程及执行顺序](https://mp.weixin.qq.com/s/DS4-ObUFeGsU2l6FvF0Pcw)
[图解 SQL 执行顺序，通俗易懂！](https://mp.weixin.qq.com/s/k1Zjr6ucFZzI-w6wC2Hz3Q)

#### 重复数据
```sql
-- 显示重复
select * from tablename group by id having count(*)>1
-- 不显示重复
select * from tablename group by id having count(*)=1
```

#### 分页

limit(start,size) start:从第几条记录开始 size:读取几条记录


#### 逃离符escape

如果要查%或者\_，怎么办呢？使用escape，转义字符后面的%或\_就不作为通配符了，注意前面没有转义字符的%和\_仍然起通配符作用

```sql
select username from user where username like '%xiao/_%' escape '/' ;
select username from user where username like '%xiao/%%' escape '/' ;
```

#### 分组排序

```sql
select a.id,a.class,a.source
from student a left join student b on a.class=b.class and a.source<=b.source
group by a.class,a.source
order by a.class,a.source
```
[阿里二面：group by 怎么优化？](https://mp.weixin.qq.com/s/igGKepCsf-SUKChco4P6Lw)


#### 有就更新 没有就插入

```sql
INSERT INTO table (name, gender)  VALUES ('Jerry', 'boy')  ON DUPLICATE KEY UPDATE name='Jerry'，gender='girl';
-- 或
REPLACE INTO table(name, gender) VALUES ('Jerry', 'girl');
-- 或
REPLACE INTO table(name, gender) SELECT 'Jerry', 'girl';
-- 或
REPLACE INTO test SET name='Jerry', gender='girl';
```


#### 三大日志

binlog,redo log,undo log

**binlog**
主要使用场景有两个，分别是主从复制和数据恢复
binlog记录了数据库表结构和表数据变更，比如update/delete/insert/truncate/create。它不会记录select（因为这没有对表没有进行变更）

**redo log**
包括两部分：一个是内存中的日志缓冲(redo log buffer)，另一个是磁盘上的日志文件(redo log file)。Mysql的基本存储结构是页(记录都存在页里边)，所以MySQL是先把这条记录所在的页找到，然后把该页加载到内存中，将对应记录进行修改。现在就可能存在一个问题：如果在内存中把数据改了，还没来得及落磁盘，而此时的数据库挂了怎么办？显然这次更改就丢了。所以引入redo log解决。MySQL每执行一条DML语句，先将记录写入redo log buffer（缓冲），后续某个时间点再一次性将多个操作记录写到redo log file（磁盘）。这种先写日志，再写磁盘的技术就是MySQL里经常说到的WAL(Write-Ahead Logging) 技术。redo log也是写到磁盘，不过是顺序IO所以速度很快。

**bin log和 redo log区别**
1. 存储的内容
binlog记载的是update/delete/insert这样的SQL语句，而redo log记载的是物理修改的内容（xxxx页修改了xxx）。
所以在搜索资料的时候会有这样的说法：redo log 记录的是数据的物理变化，binlog 记录的是数据的逻辑变化
2. 功能
redo log的作用是为持久化而生的。写完内存，如果数据库挂了，那我们可以通过redo log来恢复内存还没来得及刷到磁盘的数据，将redo log加载到内存里边，那内存就能恢复到挂掉之前的数据了。
binlog的作用是复制和恢复而生的。主从服务器需要保持数据的一致性，通过binlog来同步数据。如果整个数据库的数据都被删除了，binlog存储着所有的数据变更情况，那么可以通过binlog来对数据进行恢复。看到这里，你会想：”如果整个数据库的数据都被删除了，那我可以用redo log的记录来恢复吗？“不能,因为功能的不同，redo log 存储的是物理数据的变更，如果我们内存的数据已经刷到了磁盘了，那redo log的数据就无效了。所以redo log不会存储着历史所有数据的变更，文件的内容会被覆盖的。
3. redo log是MySQL的InnoDB引擎所产生的。binlog无论MySQL用什么引擎，都会有的。

**undo log**
主要有两个作用：回滚和多版本控制(MVCC)
在数据修改的时候，不仅记录了redo log，还记录undo log，如果因为某些原因导致事务失败或回滚了，可以用undo log进行回滚，undo log主要存储的也是逻辑日志，比如我们要insert一条数据了，那undo log会记录的一条对应的delete日志。我们要update一条记录时，它会记录一条对应相反的update记录。


- [MySQL undo log、binlog、redo log](https://mp.weixin.qq.com/s/0z6GmUp0Lb1hDUo0EyYiUg)
- [必须了解的mysql三大日志-binlog、redo log和undo log](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494134&idx=1&sn=241d91929e6a8b7b5f92db1528286645&source=41#wechat_redirect)
- [MySQL 日志(redo log 和 undo log) 都是什么鬼？](https://mp.weixin.qq.com/s/JLV8GEcF1z4D8oiDhFo6-g)
- [MySQL不会丢失数据的秘密，就藏在它的 7种日志里](https://mp.weixin.qq.com/s/d_CRZGbC6qpxdP4BYetFcA)
- [Java代码中，如何监控Mysql的binlog？](https://mp.weixin.qq.com/s/IEG6O3xNv62moh5DYxPL2Q)
- [手把手教你玩 MySQL 删库不跑路，直接把 MySQL 的 binlog 玩溜！](https://mp.weixin.qq.com/s/w4vPFHJkog2nbl68_0LOnw)
- [666！MySQL 的 binlog 的三种格式这么好玩！](https://mp.weixin.qq.com/s/FlhZsx9MBGY0ypXQMqe6wA)
- [讲一讲 MySQL 数据备份杀手锏 binlog](https://mp.weixin.qq.com/s/0dBwL3nwcldbS7hv7O5rOQ)
- [3000帧动画图解MySQL为什么需要binlog、redo log和undo log](https://mp.weixin.qq.com/s/lvw89Ix73oTqc2juF4X5sg)
- [MySQL binlog的三个业务应用场景](https://mp.weixin.qq.com/s/kqfMxkpYZ3ICkHfbv8ET1g)
- [MySQL中binlog 与 redo log 的区别](https://zhuanlan.zhihu.com/p/299512401)
- [听我讲完 redo log、binlog 原理，面试官老脸一红](https://mp.weixin.qq.com/s/PvQxdDMdC98mpwSb7pyXvw)
- [binlog恢复误删除的数据](https://mp.weixin.qq.com/s/JYuh2c3Tad8C_DP7LZO5bg)
- [聊聊redo log是什么？](https://mp.weixin.qq.com/s/ayI190ZxhAoUracU3K_HTA)
- [MySQL 为什么需要 redo log？](https://mp.weixin.qq.com/s/cXQ3RFwi-4JgH_x-3HJQnA)



#### explain执行计划

1. id: 是用来顺序标识整个查询中 select 语句的，在嵌套查询中id越大的语句越先执
2. select_type:查询的类型
- simple: 简单的SELECT（不使用UNION或子查询）
- primary: 最外面的SELECT
- union: UNION中的第二个或更高的SELECT语句
- dependent union: UNION中的第二个或更高的SELECT语句，取决于外部查询
- union result: UNION的结果
- subquery: 在子查询中首先选择SELECT
- dependent subquery: 子查询中的第一个SELECT，取决于外部查询
- derived: 派生表——该临时表是从子查询派生出来的，位于from中的子查询
- uncacheable subquery: 无法缓存结果的子查询，必须为外部查询的每一行重新计算
- uncacheable union: 在UNION中的第二个或更晚的选择属于不可缓存的子查询
3. table:当前行所查询的表
4. type:访问类型，表示数据库引擎查找表的方式。常见的type类型有： all,index,range,ref,eq_ref,const。
- all :全表扫描，表示sql语句会把表中所有表数据全部读取读取扫描一遍。效率最低，我们应尽量避免
- index :全索引扫描,表示sql语句将会把整颗二级索引树全部读取扫描一遍，因为二级索引树的数据量比全表数据量小，所以效率比all高一些。一般查询语句中查询字段为索引字段，且无where子句时，type会为index,类似全表扫描，只是扫描表的时候按照索引次序进行而不是行。主要优点就是避免了排序, 但是开销仍然非常大。如在Extra列看到Using index，说明正在使用覆盖索引，只扫描索引的数据，它比按索引次序全表扫描的开销要小很多
- range :部分索引扫描，当查询为区间查询，且查询字段为索引字段时，这时会根据where条件对索引进行部分扫描,范围扫描，一个有限制的索引扫描。key 列显示使用了哪个索引。当使用=、 <>、>、>=、<、<=、IS NULL、<=>、BETWEEN 或者 IN 操作符，用常量比较关键字列时,可以使用 range
- unique_subquery: 在使用 in 查询的情况下会取代 eq_ref
- index_merge: 此连接类型表示使用了索引合并优化。在这种情况下，输出行中的 key 列包含使用的索引列表，key_len包含所用索引的最长 key 部分列表
- ref :出现于where操作符为‘=’，且where字段为非唯一索引的单表查询或联表查询。 通过普通索引查询匹配的很多行时的类型
- ref_or_null: 跟 ref 类似的效果，不过多一个列不能 null 的条件
- eq_ref :出现于where操作符为‘=’，且where字段为唯一索引的联表查询。最多只返回一条符合条件的记录，通过使用在两个表有关联字段的时候
- const :出现于where操作符为‘=’，且where字段为唯一索引的单表查询,此时最多只会匹配到一行,当确定最多只会有一行匹配的时候，MySQL优化器会在查询前读取它而且只读取一次，因此非常快。使用主键查询往往就是 const 级别的，非常高效
- system: const 的一种特例，表中只有一行数据
fulltext: 全文索引
**单从type字段考虑效率，const > eq_ref > ref > range > index > all**
5. possible_keys:查询可能用到的索引,MySQL 可能采用的索引，但是并不一定使用
6. key:mysql 决定采用的索引来优化查询,真正使用的索引名称
sql语句实际执行时使用的索引列，有时候mysql可能会选择优化效果不是最好的索引，这时，我们可以在select语句中使用force index(INDEXNAME)来强制mysql使用指定索引或使用ignore index(INDEXNAME)强制mysql忽略指定索引
7. key_len: 索引 key 的长度
8. ref: 显示了之前的表在key列记录的索引中查找值所用的列或常量
9. rows:查询扫描的行数，预估值，不一定准确
10. Extra:额外的查询辅助信息
- extra列会包含一些十分重要的信息，我们可以根据这些信息进行sql优化
- using index: sql语句没有where查询条件，使用覆盖索引，不需要回表查询即可拿到结果 
- using where: 没有使用索引/使用了索引但需要回表查询且没有使用到下推索引 
- using index && useing where: sql语句有where查询条件，且使用覆盖索引，不需要回表查询即可拿到结果。 
- Using index condition:使用索引查询，sql语句的where子句查询条件字段均为同一索引字段，且开启索引下推功能，需要回表查询即可拿到结果
- Using index condition && using where:使用索引查询，sql语句的where子句查询条件字段存在非同一索引字段，且开启索引下推功能，需要回表查询即可拿到结果。
- using filesort: 当语句中存在order by时，且orderby字段不是索引，这个时候mysql无法利用索引进行排序，只能用排序算法重新进行排序，会额外消耗资源。 
- Using temporary：建立了临时表来保存中间结果，查询完成之后又要把临时表删除。会很影响性能，需尽快优化
11. filtered ：查询的表行占表的百分比
12. partitions：匹配的分区

- [explain都不懂，还好意思说会SQL调优?](https://mp.weixin.qq.com/s/aTkGwVU5D9u1RAbMHQconw)
- [MySQL 之 Explain 输出分析](https://mp.weixin.qq.com/s/NwRVW6Z8AMZ_QClTXH71ow)
- [什么是MySQL的执行计划（Explain关键字）？](https://mp.weixin.qq.com/s/E8wJQvldwEAzxK5mEuFhog)
- [MySQL究竟是怎么执行的(explain)](https://mp.weixin.qq.com/s/kYcrHtE82-sOqNOp_qM4Ig)


#### InnoDB

- [InnoDB自增原理都搞不清楚，还怎么CRUD？](https://mp.weixin.qq.com/s/FHoOdlWyefQLJ34MkQgFvA)
- [InnoDB原理篇：聊聊数据页变成索引这件事](https://mp.weixin.qq.com/s/V0BX5tRlrrZhfe2-ui2aRA)
- [两万字详解！InnoDB锁专题！](https://mp.weixin.qq.com/s/Io64o3tbEcf150dS5_ltbQ)
- [MySQL存储引擎InnoDB详解](https://mp.weixin.qq.com/s/6BoGlaYpdDjzZy19YhInEw)
- [innodb是如何存数据的](https://mp.weixin.qq.com/s/sr5tQF7lNjTcvwg7Fr6HjQ)
- [MySQL是如何查询数据的](https://mp.weixin.qq.com/s/ymWeGlaBYWYmfogVDFHo5w)
- [MySQL 是如何实现 ACID 的?](https://mp.weixin.qq.com/s/KbOiJ8SKJ_wFZcIyDVGD9g)
- [一文讲清，MySQL数据库一行数据在磁盘上是怎么存储的？](https://mp.weixin.qq.com/s/qJINIZ3QcXODHSMm1RTPKQ)

#### 主从复制

- [MySQL的主从如何配置](https://mp.weixin.qq.com/s/OHhX9XCQJnrBYdA27ipHHg)
- [MySQL 主从复制](https://mp.weixin.qq.com/s/xES94DmApf_GGYvT1Ku5QQ)
- [面试官：Mysql 中主库跑太快，从库追不上怎么整？](https://mp.weixin.qq.com/s/2JtNBnKShIe-0pgt3nYGkg)
- [MySQL 定时备份的几种方式，这下稳了！](https://mp.weixin.qq.com/s/gUoDR4WLeHQYzyGtmR5K3w)
- [MySQL 怎么保证备份数据的一致性？](https://mp.weixin.qq.com/s/zOOBkcl8Ns7Kk-k-WaXNhw)
- [删库不跑路！我含泪写下了 MySQL 数据恢复大法…](https://mp.weixin.qq.com/s/jgM0aZY7UJSA2NMnkmKO3Q)
- [京东一面：MySQL 主备延迟有哪些坑？主备切换策略](https://mp.weixin.qq.com/s/SXnwM_n8UVMkKF04cDVNtw)


#### other

- [MySQL自增主键为何不是连续的呢？](https://mp.weixin.qq.com/s/zv25dNB8FXn3KrnN3uJ3uQ)
- [线上MySQL的自增id用尽怎么办？](https://mp.weixin.qq.com/s/c6Dx1xEh2BNIuGpxQahAuw)
- [MySQL的自增id](https://mp.weixin.qq.com/s/BHE7YTOlo6UZ2WQzRieDxA)
- [MySQL 用 limit 为什么会影响性能？](https://mp.weixin.qq.com/s/5o-sAOFpmjRSqIoLUNesOQ)
- [MySQL 查询 limit 1000,10 和 limit 10 速度一样快吗？ 深度分页如何破解](https://mp.weixin.qq.com/s/1pmVqHBS0CyFJW9ykySF_Q)
- [MySQL 体系架构简介](https://mp.weixin.qq.com/s/HdE5QiK5Qp3_sGd_cXABIg)
- [炸裂！MySQL 82 张图带你飞！※](https://mp.weixin.qq.com/s/HWQbWSQE4O5ndI-NG1Hwwg)
- [MySQL 面试必会！](https://mp.weixin.qq.com/s/9Ex9tzoQCAEUZNU6iXLHgQ)
- [MySQL 都不清楚？直接挂了!](https://mp.weixin.qq.com/s/FUyRHpPgw0LFWEOPTPmT0g)
- [最全91道MySQL面试题 | 附答案解析](https://mp.weixin.qq.com/s/_Gd-lBfJnhwczNQA4IJyGg)
- [138 张图带你 MySQL 入门](https://mp.weixin.qq.com/s/XcNZeHdaMgx35dFoB4_n4A)
- [专治 MySQL 乱码， 再也不想看到 � �！](https://mp.weixin.qq.com/s/ar35RqaoDgasO-OY0TnDpQ)
- [MySQL 的 varchar 水太深了，你真的会用吗？](https://mp.weixin.qq.com/s/ImB0O-Z8IXldvt6YQk80jg)
- [聊聊MySQL的10大经典错误](https://mp.weixin.qq.com/s/vdz0aS8qgHHHhvZWPuaE8A)[MySQ 8.0 推出直方图，性能大大提升！](https://mp.weixin.qq.com/s/3gc8tMfnih7WQoPwTtN8IA)
- [MySQL模糊查询再也用不着 like+% 了！](https://mp.weixin.qq.com/s/ZAzRtRnYGhvie8MDZvy1Gg)
- [MySQL最大建议行数2000w, 靠谱吗？](https://mp.weixin.qq.com/s/PeZ8Py3NNb4CU2BUtxpygg)
- [图解MVCC！](https://mp.weixin.qq.com/s/1XmvVNYqt5KycmD8pJanug)
- [再有人问你什么是MVCC，就把这篇文章发给他！](https://mp.weixin.qq.com/s/WZa5UKYgU-pYKbvovLt1YQ)
- [MySQL 最朴素的监控方式](https://mp.weixin.qq.com/s/meS5Au1o9qdrSv7505mIcA)
- [1亿条数据批量插入 MySQL，哪种方式最快](https://mp.weixin.qq.com/s/c71ATJLT6_KXtb_iiUlMjg)