---
title: MySQL相关知识点
categories: 数据库
index_img: /assert/mysql.jpg
img: https://picx.zhimg.com/v2-3683fb2db4672c8fc119e736dcb1834a_1440w.jpg

---

## MySQL应用知识点

### 递归function,父子查询

创建函数时注意分隔符，mysql遇到分号就执行，在创建函数的时候容易报错（use near ' 'at line）,可使用**delimiter //** 定义分隔符

delimiter // (创建函数报错时添加这个)

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

delimiter ; -- (改回默认的MySQL delimiter：“;”）
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

**FIND_IN_SET(str,strlist)**,str-要查询的字符串,strlist-字段名参数以”,”分隔如(1,2,6,8)。查询字段(strlist)中包含(str)的结果，返回结果为null或结果。`SELECT FIND_IN_SET('b','a,b,c,d');`因为b在strlist集合中放在2的位置,下标从1开始

```sql
select * from treenodes where FIND_IN_SET(id,'1,2,3,4,5');
```
使用find_in_set函数一次返回多条记录，id是一个表的字段，然后每条记录分别是id等于1，2，3，4，5的时候，有点类似in（集合）
```sql
select * from treenodes where id in(1,2,3,4,5);
```
**当使用find_in_set配合递归函数完成递归查询时，有时会查询很慢，此时sql可以这样优化**
```sql
select * from (select ... from t1 join t2 on ...)temp,(select getChildList('10001') as cids) where find_in_set(id,cids);
```
> [MySQL中FIND_IN_SET函数执行非常慢的某种写法](https://blog.csdn.net/wokelv/article/details/78915502)

**MySQL8.0使用CTE完成递归查询**

```sql
WITH RECURSIVE cte AS (
SELECT  ID,  PID, NAME, LEVEL, Type  FROM tmp_zjs  WHERE  ID = '102' UNION ALL 
SELECT sou.ID, sou.PID, sou.NAME, sou.LEVEL, sou.Type  FROM 
cte c INNER JOIN tmp_zjs sou ON c.ID = sou.PID  )

SELECT * FROM cte
```
> [MySQL使用递归CTE遍历分层数据](https://www.begtut.com/mysql/mysql-recursive-cte.html)
> [MySQL CTE(公共表表达式)](https://www.yiibai.com/mysql/cte.html)
> [一种避免递归查询所有子部门的树数据表设计与实现](https://mp.weixin.qq.com/s/ymAjaaKKwgbpkHOZ9tgbJw)


### 触发器

```sql
CREATE TRIGGER trigger_name
trigger_time
trigger_event ON tbl_name
FOR EACH ROW
trigger_stmt
-- 其中：
-- trigger_name：标识触发器名称，用户自行指定；
--trigger_time：标识触发时机，取值为BEFORE或AFTER；
-- trigger_event：标识触发事件，取值为INSERT、UPDATE或DELETE；
-- tbl_name：标识建立触发器的表名，即在哪张表上建立触发器；
-- trigger_stmt：触发器程序体，可以是一句SQL语句，或者用BEGIN和END包含的多条语句。
```

### 存储过程(游标)

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
调用
```sql
set @a = 'a';set @b = 'b';call NewProc(@a,@b);select @b;
```

demo

```sql
CREATE DEFINER=`root`@`%` PROCEDURE `zxjc_city_month_data`(IN `table_name` varchar(50),IN `start_time` VARCHAR(50),IN `end_time` varchar(50))
BEGIN
	DECLARE ydfh_table VARCHAR(255);
	DECLARE fdcl_table VARCHAR(255);
	DECLARE dates date;
	DECLARE datee date;
	delete from zxjc_city_month_data where time = table_name;
	set dates = DATE_FORMAT(start_time,'%Y-%m-%d');
	set datee = DATE_FORMAT(end_time,'%Y-%m-%d');
	set ydfh_table = concat('zxjc_dp_sspwrgrid_his_',table_name);
	set fdcl_table = concat('zxjc_dp_ssgenerator_his_',table_name);
	set @str = concat('insert into zxjc_city_month_data(ydfh,fdcl,grl,ydl,rdb,city,time) SELECT round(ydfh.ydfh,2) as ydfh,round(fdcl.fdcl,2) as fdcl,round(grl.grl,2) as grl,round(( ydfh.ydfh * 1000 )/( 24 * 30 ),2) as ydl,round( grl.grl / ( grl.fdl * 3600 )* 100, 2 ) as rdb,dm.dq as city,\'',table_name , '\' as time	FROM bi_xzqh_dm dm LEFT JOIN ( SELECT sum( VALUE ) ydfh, id FROM ' , ydfh_table , ' WHERE meas_type = \'93872001\' AND create_time BETWEEN \'',start_time,'\' AND \'',end_time,'\' GROUP BY id ) ydfh ON ydfh.id = concat( \'0101\', dm.xzqh_dm ) LEFT JOIN ( SELECT city, sum( VALUE ) fdcl FROM ' , fdcl_table , ' WHERE create_time BETWEEN \'' ,start_time, '\' AND \'' ,end_time,'\' GROUP BY city ) fdcl ON fdcl.city = dm.dq LEFT JOIN ( SELECT city,round(ifnull(sum(
CASE WHEN (oms_rfdl=0 || oms_rfdl is null) and tmr_rfdl!=0 and tmr_rfdl is not null THEN tmr_rfdl
     WHEN (tmr_rfdl = 0||tmr_rfdl is NULL) and dfdc_rfdl!=0 and dfdc_rfdl is not null THEN dfdc_rfdl
     WHEN (dfdc_rfdl = 0||dfdc_rfdl is NULL) and qsh_rfdl!=0 and qsh_rfdl is not null THEN qsh_rfdl
     WHEN (qsh_rfdl = 0||qsh_rfdl is NULL) and gl_rfdl!=0 and gl_rfdl is not null THEN gl_rfdl
ELSE oms_rfdl END ),0),2) as fdl, ROUND(IFNULL(sum(grl),0),2) as grl FROM zxjc_dp_fdjdata_day WHERE date_time BETWEEN \'',dates,'\' AND \'',datee,'\' GROUP BY city ) grl ON grl.city = dm.dq');

-- 			select @str;
	PREPARE cm FROM @str;
	EXECUTE cm;
END
```
调用
```sql
call zxjc_city_month_data('202308', '2023-08-01 00:00:00', '2023-08-31 23:59:59');
```
如果调用输出out的话
```sql
CREATE FUNCTION myfunc(OUT out_param1 INT(11), OUT out_param2 VARCHAR(20)))
BEGIN
SET out_param1= 10;
SET out_param2='hello';
END;
CALL myfunc(@a,@b);
SELECT CONCAT('out_param1=',@a,' out_param2=',@b);
-- 或者
CREATE PROCEDURE `get_total_orders`(OUT total INT)
BEGIN
	SELECT COUNT into total from orders;
END;
-- set @total = 1;
CALL get_total_orders(@total);
SELECT @total as total;
```

#### 存储过程和函数的区别

一、含义不同

1、存储过程：存储过程是SQL 语句和可选控制流语句的预编译集合，以一个名称存储并作为一个单元处理。
2、函数：是由一个或多个 SQL 语句组成的子程序，可用于封装代码以便重新使用。 函数限制比较多，如不能用临时表，只能用表变量等

二、使用条件不同

1、存储过程：可以在单个存储过程中执行一系列 SQL 语句。而且可以从自己的存储过程内引用其它存储过程，这可以简化一系列复杂语句。
2、函数：自定义函数诸多限制，有许多语句不能使用，许多功能不能实现。函数可以直接引用返回值，用表变量返回记录集。但是，用户定义函数不能用于执行一组修改全局数据库状态的操作。

三、执行方式不同

1、存储过程：存储过程可以返回参数，如记录集，函数只能返回值或者表对象。存储过程的参数有in，out，inout三种，存储过程声明时不需要返回类型。
2、函数：函数参数只有in，而函数需要描述返回类型，且函数中必须包含一个有效的return语句。


### MySQL事件(如定时执行存储过程)

1. 确保事件调度器已启用。您可以通过运行以下命令来检查和启用事件调度器（如果未启用）：

```sql
-- 查看启动器状态 ON为打开 OFF为关闭
SHOW VARIABLES LIKE 'event_scheduler';
-- 打开
SET GLOBAL EVENT_SCHEDULER = ON;
-- 或者my.ini配置文件下加入
[mysqld]
event_scheduler = ON
-- 然后重启MySQL
```

2. 创建一个存储过程

3. 创建事件

```sql
CREATE EVENT event_name ON SCHEDULE
-- AT '2023-07-19 10:00:00' -- 某一时间执行一次
-- AT CURRENT_TIMESTAMP
EVERY 1 DAY -- 设置执行间隔，例如每天
STARTS '2023-07-19 10:00:00' -- 设置事件的开始时间
ENDS '2023-07-21 10:00:00' -- 设置事件的结束时间（可选）
ON COMPLETION PRESERVE -- 使事件在完成后保持有效（可选）
ENABLE -- 启用事件（必需）
COMMENT '这是一个定时执行存储过程的事件'
DO
    CALL my_stored_procedure(); -- 调用存储过程
-- DO CALL procedure_name('input1', @output1);

-- 选项
CREATE
	[DEFINER = { user | CURRENT_USER }]
    EVENT
    [IF NOT EXISTS]
    event_name
    ON SCHEDULE schedule
    [ON COMPLETION [NOT] PRESERVE]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT 'comment']
    DO event_body; 

```

|              sql语法              |                           **说明**                           |
| :-------------------------------: | :----------------------------------------------------------: |
|              definer              |                **可选项**，给指定用户使用权限                |
|           if not exists           |           **可选项**，用于判断要创建的事件是否存在           |
|         event event_name          | **必选项**，指定事件名称，event_name的最大长度为64个字符，如果为指定event_name，则默认为当前的mysql用户名（不区分大小写） |
|       on schedule schedule        | **必选项**，这里的schedule用于定义执行的时间和时间间隔，在下面我们详细讲解 |
|   on completion [not] preserve    | **可选项**，配置事件执行完一次后的处理方式；当为on completion preserve 的时候,当event到期了,event会被disable,但是该event还是会存在当为on completion not preserve的时候,当event到期的时候,该event会被自动删除掉. |
| enable、disable、disable on slave | **可选项**，用于指定事件的一种属性。enable表示该事件是开启的，也就是调度器检查事件是否必选调用；disable表示该事件是关闭的，也就是事件的声明存储到目录中，但是调度器不会检查它是否应该调用；disable on slave表示事件在从机中是关闭的。如果不指定这三个选择中的任意一个，则在一个事件创建之后，它立即变为活动的。 |
|         comment ‘comment’         |                **可选项**，用于定义事件的注释                |
|           do event_body           | **必选项**，用于指定事件启动时所要执行的代码。可以是任何有效的sql语句、存储过程或者一个计划执行的事件。如果包含多条语句，可以使用begin…end复合结构 |

4. 查看事件

```sql
show events;
-- 条件筛选
show events like '%event_name%';
```

5. 修改事件

```sql
--开启事件
alter event event_name enable;
--关闭事件
alter event event_name disable;
-- 修改执行计划
ALTER EVENT event_name ON SCHEDULE schedule;
-- 修改事件主体
ALTER EVENT event_name DO event_body;
-- 重命名事件
ALTER EVENT event_name RENAME TO new_event_name;
-- 移动事件到其他数据库
-- 可修改事件名称
ALTER EVENT demo.event_name RENAME TO test.event_name;
-- 删除事件
drop event event_name;
```

## MySQL函数

### 字符串函数

> 对于针对字符串位置的操作，第一个位置被标记为1。

**ASCII(str)**:返回字符串str的最左面字符的ASCII代码值。如果str是空字符串，返回0。如果str是NULL，返回NULL。
```sql
select ASCII('2');-- 50
select ASCII(2);-- 50
select ASCII('dx');-- 100
```

**ORD(str)**:如果字符串str最左面字符是一个多字节字符，通过以格式((first byte ASCII code) * 256+(second byte ASCII code))[ * 256+third byte ASCII code...]返回字符的ASCII代码值来返回多字节字符代码。如果最左面的字符不是一个多字节字符。返回与ASCII()函数返回的相同值。
```sql
select ORD('2');-- 50
```

**CONV(N,from_base,to_base)**:在不同的数字基之间变换数字。返回数字N的字符串数字，从from_base基变换为to_base基，如果任何参数是NULL，返回NULL。参数N解释为一个整数，但是可以指定为一个整数或一个字符串。最小基是2且最大的基是36。如果to_base是一个负数，N被认为是一个有符号数，否则，N被当作无符号数。CONV以64位点精度工作。
```sql
select CONV("a",16,2);-- '1010'
select CONV("6E",18,8);-- '172'
select CONV(-17,10,-18);-- '-H'
select CONV(10+"10"+'10'+0xa,10,10);-- '40'
```

**BIN(N)**:返回二进制值N的一个字符串表示，在此N是一个长整数(BIGINT)数字，这等价于`CONV(N,10,2)`。如果N是NULL，返回NULL。
```sql
select BIN(12);-- '1100'
```

**OCT(N)**:返回八进制值N的一个字符串的表示，在此N是一个长整型数字，这等价于`CONV(N,10,8)`。如果N是NULL，返回NULL。
```sql
select OCT(12);-- '14'
```

**HEX(N)**:返回十六进制值N一个字符串的表示，在此N是一个长整型(BIGINT)数字，这等价于`CONV(N,10,16)`。如果N是NULL，返回NULL。
```sql
select HEX(255);-- 'FF'
```

**CHAR(N,...)**:CHAR()将参数解释为整数并且返回由这些整数的ASCII代码字符组成的一个字符串。NULL值被跳过。
```sql
select CHAR(77,121,83,81,'76');-- 'MySQL'
select CHAR(77,77.3,'77.3');-- 'MMM'
```

**CONCAT(str1,str2,...)**:返回来自于参数连结的字符串。如果任何参数是NULL，返回NULL。可以有超过2个的参数。一个数字参数被变换为等价的字符串形式。
```sql
select CONCAT('My', 'S', 'QL');-- 'MySQL'
select CONCAT('My', NULL, 'QL');-- NULL
select CONCAT(14.3);-- '14.3'
```

**LENGTH(str)，OCTET_LENGTH(str)，CHAR_LENGTH(str)，CHARACTER_LENGTH(str)**,返回字符串str的长度。
```sql
select LENGTH('text');-- 4
select OCTET_LENGTH('text');-- 4
```
注意，对于多字节字符，其`CHAR_LENGTH()`仅计算一次。

**LOCATE(substr,str)，POSITION(substr IN str)**,返回子串substr在字符串str第一个出现的位置，如果substr不是在str里面，返回0
```sql
select LOCATE('bar', 'foobarbar');-- 4
select LOCATE('xbar', 'foobar');-- 0
```
该函数是多字节可靠的。

**LOCATE(substr,str,pos)**:返回子串substr在字符串str第一个出现的位置，从位置pos开始。如果substr不是在str里面，返回0。
```sql
select LOCATE('bar', 'foobarbar',5);-- 7
```
这函数是多字节可靠的。

**INSTR(str,substr)**:返回子串substr在字符串str中的第一个出现的位置。这与有2个参数形式的LOCATE()相同，除了参数被颠倒。
```sql
select INSTR('foobarbar', 'bar');-- 4
select INSTR('xbar', 'foobar');-- 0
```
这函数是多字节可靠的。

**LPAD(str,len,padstr)**:返回字符串str，左面用字符串padstr填补直到str是len个字符长。
```sql
select LPAD('hi',4,'??');-- '??hi'
```

**RPAD(str,len,padstr)**:返回字符串str，右面用字符串padstr填补直到str是len个字符长。
```sql
select RPAD('hi',5,'?');-- 'hi???'
```

**LEFT(str,len)**:返回字符串str的最左面len个字符。
```sql
select LEFT('foobarbar', 5);-- 'fooba'
```
该函数是多字节可靠的。

**RIGHT(str,len)**:返回字符串str的最右面len个字符。
```sql
select RIGHT('foobarbar', 4);-- 'rbar'
```
该函数是多字节可靠的。

**SUBSTRING(str,pos,len)，SUBSTRING(str FROM pos FOR len)，MID(str,pos,len)**
从字符串str返回一个len个字符的子串，从位置pos开始。使用FROM的变种形式是ANSI SQL92语法。
```sql
select SUBSTRING('Quadratically',5,6);-- 'ratica'
```
该函数是多字节可靠的。

**SUBSTRING(str,pos)，SUBSTRING(str FROM pos)**:从字符串str的起始位置pos返回一个子串。
```sql
select SUBSTRING('Quadratically',5);-- 'ratically'
select SUBSTRING('foobarbar' FROM 4);-- 'barbar'
```
该函数是多字节可靠的。

**SUBSTRING_INDEX(str,delim,count)**:返回从字符串str的第count个出现的分隔符delim之后的子串。如果count是正数，返回最后的分隔符到左边(从左边数)的所有字符。如果count是负数，返回最后的分隔符到右边的所有字符(从右边数)。
```sql
select SUBSTRING_INDEX('www.mysql.com', '.', 2);-- 'www.mysql'
select SUBSTRING_INDEX('www.mysql.com', '.', -2);-- 'mysql.com'
```
该函数对多字节是可靠的。

**LTRIM(str)**:返回删除了其前置空格字符的字符串str。
```sql
select LTRIM(' barbar');-- 'barbar'
```

**RTRIM(str)**:返回删除了其拖后空格字符的字符串str。
```sql
select RTRIM('barbar   ');-- 'barbar'
```
该函数对多字节是可靠的。

**TRIM([[BOTH | LEADING | TRAILING] [remstr] FROM] str)**:返回字符串str，其所有remstr前缀或后缀被删除了。如果没有修饰符BOTH、LEADING或TRAILING给出，BOTH被假定。如果remstr没被指定，空格被删除。
```sql
select TRIM(' bar   ');-- 'bar'
select TRIM(LEADING 'x' FROM 'xxxbarxxx');-- 'barxxx'
select TRIM(BOTH 'x' FROM 'xxxbarxxx');-- 'bar'
select TRIM(TRAILING 'xyz' FROM 'barxxyz');-- 'barx'
```
该函数对多字节是可靠的。

**SOUNDEX(str)**:返回str的一个同音字符串。听起来“大致相同”的2个字符串应该有相同的同音字符串。一个“标准”的同音字符串长是4个字符，但是SOUNDEX()函数返回一个任意长的字符串。你可以在结果上使用SUBSTRING()得到一个“标准”的同音串。所有非数字字母字符在给定的字符串中被忽略。所有在A-Z之外的字符国际字母被当作元音。
```sql
select SOUNDEX('Hello');-- 'H400'
select SOUNDEX('Quadratically');-- 'Q36324'
```

**SPACE(N)**:返回由N个空格字符组成的一个字符串。
```sql
select SPACE(6);-- '     '
```

**REPLACE(str,from_str,to_str)**:返回字符串str，其字符串from_str的所有出现由字符串to_str代替。
```sql
select REPLACE('www.mysql.com', 'w', 'Ww');-- 'WwWwWw.mysql.com'
```
该函数对多字节是可靠的。

**REPEAT(str,count)**:返回由重复countTimes次的字符串str组成的一个字符串。如果count<=0，返回一个空字符串。如果str或count是NULL，返回NULL。
```sql
select REPEAT('MySQL', 3);-- 'MySQLMySQLMySQL'
```

**REVERSE(str)**:返回颠倒字符顺序的字符串str。
```sql
select REVERSE('abc');-- 'cba'
```
该函数对多字节可靠的。

**INSERT(str,pos,len,newstr)**:返回字符串str，在位置pos起始的子串且len个字符长得子串由字符串newstr代替。
```sql
select INSERT('Quadratic', 3, 4, 'What');-- 'QuWhattic'
```
该函数对多字节是可靠的。

**ELT(N,str1,str2,str3,...)**:如果N=1，返回str1，如果N=2，返回str2，等等。如果N小于1或大于参数个数，返回NULL。ELT()是FIELD()反运算。
```sql
select ELT(1, 'ej', 'Heja', 'hej', 'foo');-- 'ej'
select ELT(4, 'ej', 'Heja', 'hej', 'foo');-- 'foo'
```

**FIELD(str,str1,str2,str3,...)**:返回str在str1,str2,str3,...清单的索引。如果str没找到，返回0。FIELD()是ELT()反运算。
```sql
select FIELD('ej', 'Hej', 'ej', 'Heja', 'hej', 'foo');-- 2
select FIELD('fo', 'Hej', 'ej', 'Heja', 'hej', 'foo');-- 0
```

**FIND_IN_SET(str,strlist)**:如果字符串str在由N子串组成的表strlist之中，返回一个1到N的值。一个字符串表是被“,”分隔的子串组成的一个字符串。如果第一个参数是一个常数字符串并且第二个参数是一种类型为SET的列，FIND_IN_SET()函数被优化而使用位运算！如果str不是在strlist里面或如果strlist是空字符串，返回0。如果任何一个参数是NULL，返回NULL。如果第一个参数包含一个“,”，该函数将工作不正常。
```sql
SELECT FIND_IN_SET('b','a,b,c,d');-- 2
```

**MAKE_SET(bits,str1,str2,...)**:返回一个集合(包含由“,”字符分隔的子串组成的一个字符串)，由相应的位在bits集合中的的字符串组成。str1对应于位0，str2对应位1，等等。在str1,str2,...中的NULL串不添加到结果中。
```sql
SELECT MAKE_SET(1,'a','b','c');-- 'a'
SELECT MAKE_SET(1 | 4,'hello','nice','world');-- 'hello,world'
SELECT MAKE_SET(0,'a','b','c');-- ''
```

**EXPORT_SET(bits,on,off,[ separator,[number_of_bits] ])**:返回一个字符串，在这里对于在“bits”中设定每一位，你得到一个“on”字符串，并且对于每个复位(reset)的位，你得到一个“off”字符串。每个字符串用“separator”分隔(缺省“,”)，并且只有“bits”的“number_of_bits”(缺省64)位被使用。
```sql
select EXPORT_SET(5,'Y','N',',',4);-- Y,N,Y,N
```

**LCASE(str),LOWER(str)**:返回字符串str，根据当前字符集映射(缺省是ISO-8859-1 Latin1)把所有的字符改变成小写。该函数对多字节是可靠的。
```sql
select LCASE('QUADRATICALLY');-- 'quadratically'
```

**UCASE(str),UPPER(str)**:返回字符串str，根据当前字符集映射(缺省是ISO-8859-1 Latin1)把所有的字符改变成大写。该函数对多字节是可靠的。
```sql
select UCASE('Hej');-- 'HEJ'
```
该函数对多字节是可靠的。

**LOAD_FILE(file_name)**:读入文件并且作为一个字符串返回文件内容。文件必须在服务器上，你必须指定到文件的完整路径名，而且你必须有file权限。文件必须所有内容都是可读的并且小于max_allowed_packet。如果文件不存在或由于上面原因之一不能被读出，函数返回NULL。
```sql
UPDATE table_name SET blob_column=LOAD_FILE("/tmp/picture") WHERE id=1;
```
MySQL必要时自动变换数字为字符串，并且反过来也如此：
```sql
SELECT 1+"1";-- 2
SELECT CONCAT(2,' test');-- '2 test'
```
如果你想要明确地变换一个数字到一个字符串，把它作为参数传递到CONCAT()。如果字符串函数提供一个二进制字符串作为参数，结果字符串也是一个二进制字符串。被变换到一个字符串的数字被当作是一个二进制字符串。这仅影响比较

**COMPRESS(string_to_compress)**:压缩一个字符串。这个函数要求MySQL已经用一个诸如zlib的压缩库压缩过。否则，返回值始终是NULL。**UNCOMPRESS()** 可将压缩过的字符串进行解压缩。

**CONCAT_WS(separator ,str1 ,str2 ,...)**:CONCAT_WS()代表CONCAT With Separator，是CONCAT()的特殊形式。第一个参数是其它参数的分隔符，分隔符的位置放在要连接的两个字符串之间。分隔符可以是一个字符串，也可以是其它参数。如果分隔符为NULL，则结果为NULL。函数会忽略任何分隔符参数后的NULL值。

**FORMAT(X ,D)**:将number X设置为格式'#,###,###.##',以四舍五入的方式保留到小数点后D位,而返回结果为一个字符串。

**QUOTE(str)**:引证一个字符串，由此产生一个在SQL语句中可用作完全转义数据值的结果。 返回的字符串由单引号标注，每例都带有单引号(‘'’)、反斜线符号(‘\’)、ASCII NUL以及前面有反斜线符号的Control-Z。如果自变量的值为NULL,则返回不带单引号的单词“NULL”。

**UNCOMPRESSED_LENGTH(compressed_string)**:返回压缩字符串压缩前的长度。

**UNHEX(str)**:执行从HEX(str)的反向操作。就是说，它将参数中的每一对十六进制数字理解为一个数字，并将其转化为该数字代表的字符。结果字符以二进制字符串的形式返回


### 时间函数

这里是一个使用日期函数的例子。下面的查询选择了所有记录，其date_col的值是在最后30天以内：
```sql
SELECT something FROM table WHERE TO_DAYS(NOW()) - TO_DAYS(date_col) <= 30;
```

**DAYOFWEEK(date)**:返回日期date的星期索引(1=星期天，2=星期一,……7=星期六)。这些索引值对应于ODBC标准。
```sql
select DAYOFWEEK('1998-02-03');-- 3
```

**WEEKDAY(date)**:返回date的星期索引(0=星期一，1=星期二,……6=星期天)。
```sql
select WEEKDAY('1997-10-04 22:23:00');-- 5
select WEEKDAY('1997-11-05');-- 2
```

**DAYOFMONTH(date)**:返回date的月份中日期，在1到31范围内。
```sql
select DAYOFMONTH('1998-02-03');-- 3
```

**DAYOFYEAR(date)**:返回date在一年中的日数,在1到366范围内。
```sql
select DAYOFYEAR('1998-02-03');-- 34
```

**MONTH(date)**:返回date的月份，范围1到12。
```sql
select MONTH('1998-02-03');-- 2
```

**DAYNAME(date)**:返回date的星期名字。
```sql
select DAYNAME("1998-02-05");-- 'Thursday'
```

**MONTHNAME(date)**:返回date的月份名字。
```sql
select MONTHNAME("1998-02-05");-- 'February'
```

**QUARTER(date)**:返回date一年中的季度，范围1到4。
```sql
select QUARTER('98-04-01');-- 2
```

**WEEK(date),WEEK(date,first)**:对于星期天是一周的第一天的地方，有一个单个参数，返回date的周数，范围在0到52。2个参数形式WEEK()允许你指定星期是否开始于星期天或星期一。如果第二个参数是0，星期从星期天开始，如果第二个参数是1，从星期一开始。
```sql
select WEEK('1998-02-20');-- 7
select WEEK('1998-02-20',0);-- 7
select WEEK('1998-02-20',1);-- 8
```

**YEAR(date)**:返回date的年份，范围在1000到9999。
```sql
select YEAR('98-02-03');-- 1998
```

**HOUR(time)**:返回time的小时，范围是0到23。
```sql
select HOUR('10:05:03');-- 10
```

**MINUTE(time)**:返回time的分钟，范围是0到59。
```sql
select MINUTE('98-02-03 10:05:03');-- 5
```

**SECOND(time)**:回来time的秒数，范围是0到59。
```sql
select SECOND('10:05:03');-- 3
```

**PERIOD_ADD(P,N)**:增加N个月到阶段P(以格式YYMM或YYYYMM)。以格式YYYYMM返回值。注意阶段参数P不是日期值。
```sql
select PERIOD_ADD(9801,2);-- 199803
```

**PERIOD_DIFF(P1,P2)**:返回在时期P1和P2之间月数，P1和P2应该以格式YYMM或YYYYMM。注意，时期参数P1和P2不是日期值。
```sql
　　mysql> select PERIOD_DIFF(9802,199703);-- 11
```

**DATE_ADD(date,INTERVAL expr type),DATE_SUB(date,INTERVAL expr type)，ADDDATE(date,INTERVAL expr type),SUBDATE(date,INTERVAL expr type)**,这些功能执行日期运算。对于MySQL3.22，他们是新的。ADDDATE()和SUBDATE()是DATE_ADD()和DATE_SUB()的同义词。在MySQL3.23中，你可以使用+和-而不是DATE_ADD()和DATE_SUB()。（见例子）date是一个指定开始日期的DATETIME或DATE值，expr是指定加到开始日期或从开始日期减去的间隔值一个表达式，expr是一个字符串；它可以以一个“-”开始表示负间隔。type是一个关键词，指明表达式应该如何被解释。EXTRACT(type FROM date)函数从日期中返回“type”间隔。下表显示了type和expr参数怎样被关联

| type值        | 含义               | 期望的expr格式               |
| ------------- | ------------------ | ---------------------------- |
| SECOND        | 秒                 | SECONDS                      |
| MINUTE        | 分钟               | MINUTES                      |
| HOUR          | 时间               | HOURS                        |
| DAY           | 天                 | DAYS                         |
| MONTH         | 月                 | MONTHS                       |
| YEAR          | 年                 | YEARS                        |
| MINUTE_SECOND | 分钟和秒           | "MINUTES:SECONDS"            |
| HOUR_MINUTE   | 小时和分钟         | "HOURS:MINUTES"              |
| DAY_HOUR      | 天和小时           | "DAYS HOURS"                 |
| YEAR_MONTH    | 年和月             | "YEARS-MONTHS"               |
| HOUR_SECOND   | 小时, 分钟         | "HOURS:MINUTES:SECONDS"      |
| DAY_MINUTE    | 天, 小时, 分钟     | "DAYS HOURS:MINUTES"         |
| DAY_SECOND    | 天, 小时, 分钟, 秒 | "DAYS HOURS:MINUTES:SECONDS" |

MySQL在expr格式中允许任何标点分隔符。表示显示的是建议的分隔符。如果date参数是一个DATE值并且你的计算仅仅包含YEAR、MONTH和DAY部分(即，没有时间部分)，结果是一个DATE值。否则结果是一个DATETIME值。
```sql
SELECT "1997-12-31 23:59:59" + INTERVAL 1 SECOND;-- 1998-01-01 00:00:00
SELECT INTERVAL 1 DAY + "1997-12-31";-- 1998-01-01
SELECT "1998-01-01" - INTERVAL 1 SECOND;-- 1997-12-31 23:59:59
SELECT DATE_ADD("1997-12-31 23:59:59",INTERVAL 1 SECOND);-- 1998-01-01 00:00:00
SELECT DATE_ADD("1997-12-31 23:59:59",INTERVAL 1 DAY);-- 1998-01-01 23:59:59
SELECT DATE_ADD("1997-12-31 23:59:59",INTERVAL "1:1" MINUTE_SECOND);-- 1998-01-01 00:01:00
SELECT DATE_SUB("1998-01-01 00:00:00",INTERVAL "1 1:1:1" DAY_SECOND);-- 1997-12-30 22:58:59
SELECT DATE_ADD("1998-01-01 00:00:00", INTERVAL "-1 10" DAY_HOUR);-- 1997-12-30 14:00:00
SELECT DATE_SUB("1998-01-02", INTERVAL 31 DAY);-- 1997-12-02
SELECT EXTRACT(YEAR FROM "1999-07-02");-- 1999
SELECT EXTRACT(YEAR_MONTH FROM "1999-07-02 01:02:03");-- 199907
SELECT EXTRACT(DAY_MINUTE FROM "1999-07-02 01:02:03");-- 20102
```

如果你指定太短的间隔值(不包括type关键词期望的间隔部分)，MySQL假设你省掉了间隔值的最左面部分。例如，如果你指定一个type是DAY_SECOND，值expr被希望有天、小时、分钟和秒部分。如果你象"1:10"这样指定值，MySQL假设日子和小时部分是丢失的并且值代表分钟和秒。换句话说，"1:10" DAY_SECOND以它等价于"1:10" MINUTE_SECOND的方式解释，这对那MySQL解释TIME值表示经过的时间而非作为一天的时间的方式有二义性。如果你使用确实不正确的日期，结果是NULL。如果你增加MONTH、YEAR_MONTH或YEAR并且结果日期大于新月份的最大值天数，日子在新月用最大的天调整。
```sql
select DATE_ADD('1998-01-30', Interval 1 month);-- 1998-02-28
```
注意，从前面的例子中词INTERVAL和type关键词不是区分大小写的。
　　
**TO_DAYS(date)**:给出一个日期date，返回一个天数(从0年的天数)。
```sql
select TO_DAYS(950501);-- 728779
select TO_DAYS('1997-10-07');-- 729669
```
TO_DAYS()不打算用于使用格列高里历(1582)出现前的值。

**FROM_DAYS(N)**:给出一个天数N，返回一个DATE值。
```sql
select FROM_DAYS(729669);-- '1997-10-07'
```

**DATE_FORMAT(date,format)**:根据format字符串格式化date值。下列修饰符可以被用在format字符串中:

- %M 月名字(January……December)
- %W 星期名字(Sunday……Saturday)
- %D 有英语前缀的月份的日期(1st,2nd,3rd,等等。)
- %Y 年,数字,4位
- %y 年,数字,2位
- %a 缩写的星期名字(Sun……Sat)
- %d 月份中的天数,数字(00……31)
- %e 月份中的天数,数字(0……31)
- %m 月,数字(01……12)
- %c 月,数字(1……12)
- %b 缩写的月份名字(Jan……Dec)
- %j 一年中的天数(001……366)
- %H 小时(00……23)
- %k 小时(0……23)
- %h 小时(01……12)
- %I 小时(01……12)
- %l 小时(1……12)
- %i 分钟,数字(00……59)
- %r 时间,12小时(hh:mm:ss [AP]M)
- %T 时间,24小时(hh:mm:ss)
- %S 秒(00……59)
- %s 秒(00……59)
- %p AM或PM
- %w 一个星期中的天数(0=Sunday ……6=Saturday)
- %U 星期(0……52),这里星期天是星期的第一天
- %u 星期(0……52),这里星期一是星期的第一天
- %% 一个文字“%”。

所有的其他字符不做解释被复制到结果中。
```sql
select DATE_FORMAT('1997-10-04 22:23:00', '%W %M %Y');-- 'Saturday October 1997'
select DATE_FORMAT('1997-10-04 22:23:00', '%H:%i:%s');-- '22:23:00'
select DATE_FORMAT('1997-10-04 22:23:00','%D %y %a %d %m %b %j');-- '4th 97 Sat 04 10 Oct 277'
select DATE_FORMAT('1997-10-04 22:23:00','%H %k %I %r %T %S %w');-- '22 22 10 10:23:00 PM 22:23:00 00 6'
```

MySQL3.23中，在格式修饰符字符前需要%。在MySQL更早的版本中，%是可选的。

**TIME_FORMAT(time,format)**:这像上面的DATE_FORMAT()函数一样使用，但是format字符串只能包含处理小时、分钟和秒的那些格式修饰符。其他修饰符产生一个NULL值或0。
　　
**CURDATE(),CURRENT_DATE**:以'YYYY-MM-DD'或YYYYMMDD格式返回今天日期值，取决于函数是在一个字符串还是数字上下文被使用。
```sql
select CURDATE();-- '1997-12-15'
select CURDATE() + 0;-- 19971215
```

**CURTIME(),CURRENT_TIME**:以'HH:MM:SS'或HHMMSS格式返回当前时间值，取决于函数是在一个字符串还是在数字的上下文被使用。
```sql
select CURTIME();-- '23:50:26'
select CURTIME() + 0;-- 235026
```

**NOW(),SYSDATE(),CURRENT_TIMESTAMP**,以'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS格式返回当前的日期和时间，取决于函数是在一个字符串还是在数字的上下文被使用。
```sql
select NOW();-- '1997-12-15 23:50:26'
select NOW() + 0;-- 19971215235026
```

**UNIX_TIMESTAMP(),UNIX_TIMESTAMP(date)**:如果没有参数调用，返回一个Unix时间戳记(从'1970-01-01 00:00:00'GMT开始的秒数)。如果UNIX_TIMESTAMP()用一个date参数被调用，它返回从'1970-01-01 00:00:00'GMT开始的秒数值。date可以是一个DATE字符串、一个DATETIME字符串、一个TIMESTAMP或以YYMMDD或YYYYMMDD格式的本地时间的一个数字。
```sql
select UNIX_TIMESTAMP();-- 882226357
select UNIX_TIMESTAMP('1997-10-04 22:23:00');-- 875996580
```

当UNIX_TIMESTAMP被用于一个TIMESTAMP列，函数将直接接受值，没有隐含的“string-to-unix-timestamp”变换。

**FROM_UNIXTIME(unix_timestamp)**:以'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS格式返回unix_timestamp参数所表示的值，取决于函数是在一个字符串还是或数字上下文中被使用。
```sql
select FROM_UNIXTIME(875996580);-- '1997-10-04 22:23:00'
select FROM_UNIXTIME(875996580) + 0;-- 19971004222300
```

**FROM_UNIXTIME(unix_timestamp,format)**:返回表示Unix时间标记的一个字符串，根据format字符串格式化。format可以包含与DATE_FORMAT()函数列出的条目同样的修饰符。
```sql
select FROM_UNIXTIME(UNIX_TIMESTAMP(),'%Y %D %M %h:%i:%s %x');-- '1997 23rd December 03:43:30 x'
```

**SEC_TO_TIME(seconds)**:返回seconds参数，变换成小时、分钟和秒，值以'HH:MM:SS'或HHMMSS格式化，取决于函数是在一个字符串还是在数字上下文中被使用。
```sql
select SEC_TO_TIME(2378);-- '00:39:38'
select SEC_TO_TIME(2378) + 0;-- 3938
```

**TIME_TO_SEC(time)**:返回time参数，转换成秒。
```sql
select TIME_TO_SEC('22:23:00');-- 80580
select TIME_TO_SEC('00:39:38');-- 2378
```
Mysql取系统函数：
```sql
Select curtime();
Select curdate();
Select sysdate();
select now();
```

**ADDDATE(date ,INTERVAL expr type ),ADDDATE(expr ,days)**:当被第二个参数的INTERVAL格式激活后，ADDDATE()就是DATE_ADD()的同义词。相关函数SUBDATE()则是DATE_SUB()的同义词。对于INTERVAL参数上的信息，请参见关于DATE_ADD()的论述。

**ADDTIME(expr ,expr2)**:ADDTIME()将expr2添加至expr然后返回结果。expr是一个时间或时间日期表达式，而expr2是一个时间表达式。

**CONVERT_TZ(dt ,from_tz ,to_tz)**:将时间日期值dt从from_tz给出的时区转到to_tz给出的时区，然后返回结果值。关于可能指定的时区的详细论述，若自变量无效，则这个函数会返回NULL

**DATE(expr)**:提取日期或时间日期表达式expr中的日期部分。

**DATEDIFF(expr,expr2)**:返回起始时间expr和结束时间expr2之间的天数。Expr和expr2为日期或date-and-time表达式。计算中只用到这些值的日期部分。

**DATE_ADD(date ,INTERVAL expr type ),DATE_SUB(date ,INTERVAL expr type)**:这些函数执行日期运算。date是一个DATETIME或DATE值，用来指定起始时间。expr是一个表达式，用来指定从起始日期添加或减去的时间间隔值。Expr是一个字符串;对于负值的时间间隔，它可以以一个‘-’开头。type为关键词，它指示了表达式被解释的方式。

**DAY(date)**:DAY()和DAYOFMONTH()的意义相同

**EXTRACT(type FROM date)**:函数所使用的时间间隔类型说明符同DATE_ADD()或DATE_SUB()的相同,但它从日期中提取其部分，而不是执行日期运算。

**GET_FORMAT(DATE|TIME|DATETIME, 'EUR'|'USA'|'JIS'|'ISO'|'INTERNAL')**:返回一个格式字符串。这个函数在同DATE_FORMAT()及STR_TO_DATE()函数结合时很有用

**LAST_DAY(date)**:获取一个日期或日期时间值，返回该月最后一天对应的值。若参数无效，则返回NULL。

**LOCALTIME,LOCALTIME()**:LOCALTIME及LOCALTIME()和NOW()具有相同意义。

**LOCALTIMESTAMP, LOCALTIMESTAMP()**:LOCALTIMESTAMP和LOCALTIMESTAMP()和NOW()具有相同意义。

**MAKEDATE(year ,dayofyear)**:给出年份值和一年中的天数值，返回一个日期。dayofyear必须大于0，否则结果为NULL。

**MAKETIME(hour ,minute ,second)**:返回由hour、minute和second参数计算得出的时间值

**CROSECOND(expr)**:从时间或日期时间表达式expr返回微秒值，其数字范围从0到999999。

**STR_TO_DATE(str,format)**:这是DATE_FORMAT()函数的倒转。它获取一个字符串str和一个格式字符串format。若格式字符串包含日期和时间部分，则STR_TO_DATE()返回一个DATETIME值，若该字符串只包含日期部分或时间部分，则返回一个DATE或TIME值。

**SUBDATE(date ,INTERVAL expr type),SUBDATE(expr,days)**:当被第二个参数的INTERVAL型式调用时,SUBDATE()和DATE_SUB()的意义相同。对于有关INTERVAL参数的信息，见有关DATE_ADD()的讨论。

**SUBTIME(expr,expr2)**:SUBTIME()从expr中提取expr2,然后返回结果。expr是一个时间或日期时间表达式，而xpr2是一个时间表达式。

**SYSDATE()**:返回当前日期和时间值，格式为'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS，具体格式根据函数是否用在字符串或数字语境而定。

**TIME(expr)**:提取一个时间或日期时间表达式的时间部分，并将其以字符串形式返回。

**TIMEDIFF(expr ,expr2)**:返回起始时间expr和结束时间expr2之间的时间。expr和expr2为时间或date-and-time表达式 , 两个的类型必须一样。

**TIMESTAMP(expr) , TIMESTAMP(expr ,expr2)**:对于一个单参数,该函数将日期或日期时间表达式expr作为日期时间值返回，对于两个参数,它将时间表达式expr2添加到日期或日期时间表达式expr中，将theresult作为日期时间值返回。

**TIMESTAMPADD(interval ,int_expr ,datetime_expr)**:将整型表达式int_expr添加到日期或日期时间表达式datetime_expr中。int_expr的单位被时间间隔参数给定，该参数必须是以下值的其中一个：FRAC_SECOND、SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、QUARTER或YEAR。可使用所显示的关键词指定Interval值，或使用SQL_TSI_前缀。例如,DAY或SQL_TSI_DAY都是正确的

**TIMESTAMPDIFF(interval ,datetime_expr1 ,datetime_expr2)**:返回日期或日期时间表达式datetime_expr1和datetime_expr2之间的整数差。其结果的单位由interval参数给出。interval的法定值同TIMESTAMPADD()函数说明中所列出的相同。

**UTC_DATE()**:返回当前UTC日期值，其格式为'YYYY-MM-DD'或YYYYMMDD，具体格式取决于函数是否用在字符串或数字语境中。

**UTC_TIME()**:返回当前UTC值，其格式为'HH:MM:SS'或HHMMSS，具体格式根据该函数是否用在字符串或数字语境而定。

**UTC_TIMESTAMP()**:返回当前UTC日期及时间值，格式为'YYYY-MM-DD HH:MM:SS'或YYYYMMDDHHMMSS，具体格式根据该函数是否用在字符串或数字语境而定

**WEEKOFYEAR(date)**:将该日期的阳历周以数字形式返回，范围是从1到53。它是一个兼容度函数，相当于WEEK(date ,3)。

**YEARWEEK(date), YEARWEEK(date,start)**:返回一个日期对应的年或周。start参数的工作同start参数对WEEK()的工作相同。结果中的年份可以和该年的第一周和最后一周对应的日期参数有所不同。

### 控制流程函数

**CASE WHEN THEN函数**

语法：
```sql
CASE value WHEN [compare-value] THEN result [WHEN [compare-value] THEN result ……] [ELSE result ] END CASE WHEN [condition] THEN result [WHEN[condition] THEN result ……] [ELSE result] END;
```
函数用法说明：在第一个方案的返回结果中，value=compare-value。而第二个方案的返回结果是第一种情况的真实结果。如果没有匹配的结果值，则返回结果为ELSE后的结果，如果没有ELSE部分，则返回值为NULL

**IF函数用法**

语法：
```sql
IF(expr1,expr2,expr3)
```
函数用法说明：如果expr1是TRUE(expr1 <> 0 and expr1 <> NULL) ，则IF()的返回值为expr2;否则返回值则为expr3。IF()的返回值为数字值或字符串值，具体情况视其所在语境而定

**IFNULL函数**

语法：
```sql
IFNULL(expr1,expr2)
```
函数用法说明：假如expr1不为NULL，则IFNULL()的返回值为expr1;否则其返回值为expr2。IFNULL()的返回值是数字或是字符串，具体情况取决于其所使用的语境

### 数学函数

**ABS(X)**:返回X的绝对值

**ACOS(X)**:返回X反余弦,即,余弦是X的值。若X不在-1到1的范围之内，则返回NULL。

**ASIN(X)**:返回X的反正弦，即，正弦为X的值。若X不在-1到1的范围之内，则返回NULL。

**ATAN(X)**:返回X的反正切，即，正切为X的值。

**ATAN(Y,X),ATAN2(Y,X)**:返回两个变量X及Y的反正切。它类似于Y或X的反正切计算, 除非两个参数的符号均用于确定结果所在象限。

**CEILING(X),CEIL(X)**:返回不小于X的最小整数值。

**COS(X)**:返回X的余弦，其中X在弧度上已知。

**COT(X)**:返回X的余切

**CRC32(expr)**:计算循环冗余码校验值并返回一个32比特无符号值。若参数为NULL，则结果为NULL。该参数应为一个字符串，而且在不是字符串的情况下会被作为字符串处理（若有可能）

**DEGREES(X)**:返回参数X,该参数由弧度被转化为度。

**EXP(X)**:返回e的X乘方后的值(自然对数的底)。

**FLOOR(X)**:返回不大于X的最大整数值。

**FORMAT(X,D)**:将数字X的格式写成'#,###,###.##'格式,即保留小数点后D位，而第D位的保留方式为四舍五入，然后将结果以字符串的形式返回

**LN(X)**:返回X的自然对数,即X相对于基数e的对数

**LOG(X),LOG(B,X)**:若用一个参数调用，这个函数就会返回X的自然对数。

**LOG2(X)**:返回X的基数为2的对数。

**LOG10(X)**:返回X的基数为10的对数。

**MOD(N,M),N%M,N MOD M**:模操作。返回N被M除后的余数。

**PI()**:返回ϖ(pi)的值。默认的显示小数位数是7位,然而MySQL内部会使用完全双精度值。

**POW(X,Y),POWER(X,Y)**:返回X的Y乘方的结果值。

**RADIANS(X)**:返回由度转化为弧度的参数X, (注意ϖ弧度等于180度)。

**RAND(),RAND(N)**:返回一个随机浮点值v，范围在0到1之间(即,其范围为0 ≤ v ≤ 1.0)。若已指定一个整数参数N，则它被用作种子值，用来产生重复序列。

**ROUND(X),ROUND(X,D)**:返回参数X,其值接近于最近似的整数。在有两个参数的情况下，返回X，其值保留到小数点后D位，而第D位的保留方式为四舍五入。若要接保留X值小数点左边的D位，可将D设为负值。

**SIGN(X)**:返回参数作为-1、0或1的符号，该符号取决于X的值为负、零或正。

**SIN(X)**:返回X正弦，其中X在弧度中被给定。

**SQRT(X)**:返回非负数X的二次方根。

**TAN(X)**:返回X的正切，其中X在弧度中被给定。

**TRUNCATE(X,D)**:返回被舍去至小数点后D位的数字X。若D的值为0,则结果不带有小数点或不带有小数部分。可以将D设为负数,若要截去(归零)X小数点左起第D位开始后面所有低位的值


### 全文搜索功能函数

MATCH (col1,col2,...)
AGAINST (expr [IN BOOLEAN MODE | WITH QUERY EXPANSION])

### 加密函数

**AES_ENCRYPT(str,key_str),AES_DECRYPT(crypt_str,key_str)**:这些函数允许使用官方AES进行加密和数据加密(高级加密标准)算法,即以前人们所熟知的“Rijndael”。保密关键字的长度为128比特，不过你可以通过改变源而将其延长到256比特。我们选择了128比特的原因是它的速度要快得多，且对于大多数用途而言这个保密程度已经够用。

**DECODE(crypt_str,pass_str)**:使用pass_str作为密码，解密加密字符串crypt_str，crypt_str应该是由ENCODE()返回的字符串。

**ENCODE(str,pass_str)**:使用pass_str作为密码，解密str。使用DECODE()解密结果。

**DES_DECRYPT(crypt_str [,key_str ])**:使用DES_ENCRYPT()加密一个字符串。若出现错误，这个函数会返回NULL。

**DES_ENCRYPT(str [,(key_num |key_str )])**:用Triple-DES算法给出的关键字加密字符串。若出现错误，这个函数会返回NULL

**ENCRYPT(str [,salt ])**:使用Unix crypt()系统调用加密str。salt参数应为一个至少包含2个字符的字符串。若没有给出salt参数，则使用任意值。

**MD5(str)**:为字符串算出一个MD5 128比特检查和。该值以32位十六进制数字的二进制字符串的形式返回,若参数为NULL则会返回NULL。例如，返回值可被用作散列关键字

**OLD_PASSWORD(str)**:当PASSWORD()的执行变为改善安全性时，OLD_PASSWORD()会被添加到MySQL。OLD_PASSWORD()返回从前的PASSWORD()执行值(4.1之前)，同时允许你为任何4.1之前的需要连接到你的5.1版本MySQL服务器前客户端设置密码，从而不至于将它们切断

**PASSWORD(str)**:从原文密码str计算并返回密码字符串，当参数为NULL时返回NULL。这个函数用于用户授权表的Password列中的加密MySQL密码存储

### 信息函数

**BENCHMARK(count,expr)**:函数重复count次执行表达式expr。它可以被用于计算MySQL处理表达式的速度。结果值通常为0 。另一种用处来自mysql客户端内部,能够报告问询执行的次数

**CHARSET(str)**:返回字符串自变量的字符集。

**COERCIBILITY(str)**:返回字符串自变量的整序可压缩性值。

**COLLATION(str)**:返回惠字符串参数的排序方式。

**CONNECTION_ID()**:返回对于连接的连接ID(线程ID)。每个连接都有各自的唯一ID。

**CURRENT_USER,CURRENT_USER()**:返回当前话路被验证的用户名和主机名组合。这个值符合确定你的存取权限的MySQL账户。在被指定SQL SECURITY DEFINER特征的存储程序内，CURRENT_USER()返回程序的创建者

**DATABASE()**:返回使用utf8字符集的默认(当前)数据库名。在存储程序里，默认数据库是同该程序向关联的数据库，但并不一定与调用语境的默认数据库相同。

**FOUND_ROWS()**:SELECT语句可能包括一个LIMIT子句，用来限制服务器返回客户端的行数。在有些情况下，需要不用再次运行该语句而得知在没有LIMIT时到底该语句返回了多少行。为了知道这个行数,包括在SELECT语句中选择SQL_CALC_FOUND_ROWS，随后调用FOUND_ROWS()

**LAST_INSERT_ID(),LAST_INSERT_ID(expr)**:自动返回最后一个INSERT或UPDATE问询为AUTO_INCREMENT列设置的第一个发生的值。

**ROW_COUNT()**:返回被前面语句升级的、插入的或删除的行数。这个行数和mysql客户端显示的行数及mysql_affected_rows() C API函数返回的值相同。

**SCHEMA()**:这个函数和DATABASE()具有相同的意义

**SESSION_USER()**:SESSION_USER()和USER()具有相同的意义。

**SYSTEM_USER()**:SYSTEM_USER()和USER()具有相同的意义

**USER()**:返回当前MySQL用户名和机主名

**VERSION()**:返回指示MySQL服务器版本的字符串。这个字符串使用utf8字符集。

### 其他函数

**DEFAULT(col_name)**:返回一个表列的默认值。若该列没有默认值则会产生错误。

**GET_LOCK(str,timeout)**:设法使用字符串str给定的名字得到一个锁，超时为timeout秒。若成功得到锁，则返回1，若操作超时则返回0 (例如,由于另一个客户端已提前封锁了这个名字),若发生错误则返回NULL(诸如缺乏记忆或线程mysqladmin kill被断开)。假如你有一个用GET_LOCK()得到的锁，当你执行RELEASE_LOCK()或你的连接断开(正常或非正常)时，这个锁就会解除

**INET_ATON(expr)**:给出一个作为字符串的网络地址的点地址表示，返回一个代表该地址数值的整数。地址可以是4或8比特地址。

**INET_NTOA(expr)**:给定一个数字网络地址(4或8比特),返回作为字符串的该地址的电地址表示

**IS_FREE_LOCK(str)**:检查名为str的锁是否可以使用(换言之没有被封锁)。若锁可以使用，则返回1(没有人在用这个锁),若这个锁正在被使用，则返回0，出现错误则返回NULL(诸如不正确的参数)

**IS_USED_LOCK(str)**:检查名为str的锁是否正在被使用(换言之,被封锁)。若被封锁，则返回使用该锁的客户端的连接标识符。否则返回NULL。

**MASTER_POS_WAIT(log_name ,log_pos [,timeout])**:该函数对于控制主从同步很有用处。它会持续封锁，直到从设备阅读和应用主机记录中所有补充资料到指定的位置。返回值是其为到达指定位置而必须等待的记录事件的数目。若从设备SQL线程没有被启动、从设备主机信息尚未初始化、参数不正确或出现任何错误，则该函数返回NULL。若超时时间被超过，则返回-1。若在MASTER_POS_WAIT()等待期间，从设备SQL线程中止，则该函数返回NULL。若从设备由指定位置通过，则函数会立即返回结果。

**NAME_CONST(name,value)**:返回给定值。当用来产生一个结果集合列时,NAME_CONST()促使该列使用给定名称。

**RELEASE_LOCK(str)**:解开被GET_LOCK()获取的，用字符串str所命名的锁。若锁被解开，则返回1，若改线程尚未创建锁，则返回0(此时锁没有被解开),若命名的锁不存在，则返回NULL。若该锁从未被对GET_LOCK()的调用获取，或锁已经被提前解开，则该锁不存在。

**SLEEP(duration)**:睡眠(暂停)时间为duration参数给定的秒数，然后返回0。若SLEEP()被中断,它会返回1。duration或许或包括一个给定的以微秒为单位的分数部分。

**UUID()**:返回一个通用唯一标识符(UUID)，UUID被设计成一个在时间和空间上都独一无二的数字。2个对UUID()的调用应产生2个不同的值，即使这些调用的执行是在两个互不相连的单独电脑上进行。

**VALUES(col_name)**:在一个`INSERT … ON DUPLICATE KEY UPDATE … `语句中，你可以在UPDATE子句中使用VALUES(col_name)函数，用来访问来自该语句的INSERT部分的列值。换言之，UPDATE子句中的VALUES(col_name)访问需要被插入的col_name的值,并不会发生重复键冲突。这个函数在多行插入中特别有用。VALUES()函数只在INSERT ... UPDATE语句中有意义，而在其它情况下只会返回NULL


### 聚合函数

**AVG([DISTINCT] expr)**:返回expr的平均值。DISTINCT选项可用于返回expr的不同值的平均值。

**BIT_AND(expr)**:返回expr中所有比特的bitwise AND。计算执行的精确度为64比特(BIGINT)。若找不到匹配的行，则这个函数返回18446744073709551615。(这是无符号BIGINT值，所有比特被设置为1)。

**BIT_OR(expr)**:返回expr中所有比特的bitwise OR。计算执行的精确度为64比特(BIGINT)。若找不到匹配的行，则函数返回0。

**BIT_XOR(expr)**:返回expr中所有比特的bitwise XOR。计算执行的精确度为64比特(BIGINT)。若找不到匹配的行，则函数返回0。

**COUNT(expr)**:返回SELECT语句检索到的行中非NULL值的数目。若找不到匹配的行，则COUNT()返回0

**COUNT(DISTINCT expr ,[expr ...])**:返回不同的非NULL值数目。若找不到匹配的项，则COUNT(DISTINCT)返回0

**GROUP_CONCAT(expr)**:该函数返回带有来自一个组的连接的非NULL值的字符串结果。其完整的语法如下所示：
```sql
GROUP_CONCAT([DISTINCT] expr [,expr ...]

[ORDER BY {unsigned_integer | col_name | expr }

                 [ASC | DESC] [,col_name ...]]

             [SEPARATOR str_val ])
```

**MIN([DISTINCT] expr), MAX([DISTINCT] expr)**:返回expr的最小值和最大值。MIN()和MAX()的取值可以是一个字符串参数；在这些情况下，它们返回最小或最大字符串值。

**STD(expr),STDDEV(expr)**:返回expr的总体标准偏差。这是标准SQL的延伸。这个函数的STDDEV()形式用来提供和Oracle的兼容性。可使用标准SQL函数STDDEV_POP()进行代替

**STDDEV_POP(expr)**:返回expr的总体标准偏差(VAR_POP()的平方根)。你也可以使用STD()或STDDEV(),它们具有相同的意义，然而不是标准的SQL。若找不到匹配的行，则STDDEV_POP()返回NULL

**STDDEV_SAMP(expr)**:返回expr的样本标准差(VAR_SAMP()的平方根)。若找不到匹配的行，则STDDEV_SAMP()返回NULL

**SUM([DISTINCT] expr)**:返回expr的总数。若返回集合中无任何行，则SUM()返回NULL。DISTINCT关键词可用于MySQL5.1中，求得expr不同值的总和。若找不到匹配的行，则SUM()返回NULL

**VAR_POP(expr)**:返回expr总体标准方差。它将行视为总体，而不是一个样本，所以它将行数作为分母。你也可以使用VARIANCE(),它具有相同的意义然而不是标准的SQL

**VAR_SAMP(expr)**:返回expr的样本方差。更确切的说，分母的数字是行数减去1。若找不到匹配的行，则VAR_SAMP()返回NULL

**VARIANCE(expr)**:返回expr的总体标准方差。这是标准SQL的延伸。可使用标准SQL函数VAR_POP()进行代替。若找不到匹配的项，则VARIANCE()返回NULL

## MySQL锁

### 表级锁和行级锁了解吗？有什么区别？

MyISAM仅仅支持表级锁(table-level locking)，一锁就锁整张表，这在并发写的情况下性非常差。InnoDB不光支持表级锁(table-level locking)，还支持行级锁(row-level locking)，默认为行级锁。行级锁的粒度更小，仅对相关的记录上锁即可（对一行或者多行记录加锁），所以对于并发写入操作来说，InnoDB的性能更高。

**表级锁和行级锁对比**：

- **表级锁**:MySQL中锁定粒度最大的一种锁（全局锁除外），是针对非索引字段加的锁，对当前操作的整张表加锁，实现简单，资源消耗也比较少，加锁快，不会出现死锁。不过，触发锁冲突的概率最高，高并发下效率极低。表级锁和存储引擎无关，MyISAM和InnoDB引擎都支持表级锁。
- **行级锁**:MySQL中锁定粒度最小的一种锁，是针对索引字段加的锁，只针对当前操作的行记录进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。行级锁和存储引擎有关，是在存储引擎层面实现的。

### 行级锁的使用有什么注意事项？

InnoDB的行锁是针对索引字段加的锁，表级锁是针对非索引字段加的锁。当我们执行`UPDATE`、`DELETE`语句时，如果`WHERE`条件中字段没有命中唯一索引或者索引失效的话，就会导致扫描全表对表中的所有行记录进行加锁。这个在我们日常工作开发中经常会遇到，一定要多多注意！不过，很多时候即使用了索引也有可能会走全表扫描，这是因为MySQL优化器的原因。

### InnoDB有哪几类行锁？

InnoDB行锁是通过对索引数据页上的记录加锁实现的，MySQLInnoDB支持三种行锁定方式：

- **记录锁（RecordLock）**：也被称为记录锁，属于单个行记录上的锁。
- **间隙锁（GapLock）**：锁定一个范围，不包括记录本身。
- **临键锁（Next-KeyLock）**：RecordLock+GapLock，锁定一个范围，包含记录本身，主要目的是为了解决幻读问题（MySQL事务部分提到过）。记录锁只能锁住已经存在的记录，为了避免插入新记录，需要依赖间隙锁。

在InnoDB默认的隔离级别REPEATABLE-READ下，行锁默认使用的是Next-KeyLock。但是，如果操作的索引是唯一索引或主键，InnoDB会对Next-KeyLock进行优化，将其降级为RecordLock，即仅锁住索引本身，而不是范围。

> 一些大厂面试中可能会问到Next-KeyLock的加锁范围，这里推荐一篇文章：[MySQLnext-keylock加锁范围是什么？-程序员小航-2021](https://segmentfault.com/a/1190000040129107)。

### 共享锁和排他锁呢？

不论是表级锁还是行级锁，都存在共享锁（ShareLock，S锁）和排他锁（ExclusiveLock，X锁）这两类：

- **共享锁（S锁）**：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取（锁兼容）。
- **排他锁（X锁）**：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁（锁不兼容）。

排他锁与任何的锁都不兼容，共享锁仅和共享锁兼容。

|      | S 锁   | X 锁 |
| :--- | :----- | :--- |
| S 锁 | 不冲突 | 冲突 |
| X 锁 | 冲突   | 冲突 |

由于MVCC的存在，对于一般的SELECT语句，InnoDB不会加任何锁。不过，你可以通过以下语句显式加共享锁或排他锁。

```sql
# 共享锁
SELECT ... LOCK IN SHARE MODE;
# 排他锁
SELECT ... FOR UPDATE;
```

### 意向锁有什么作用？

如果需要用到表锁的话，如何判断表中的记录没有行锁呢，一行一行遍历肯定是不行，性能太差。我们需要用到一个叫做意向锁的东东来快速判断是否可以对某个表使用表锁。

意向锁是表级锁，共有两种：

- **意向共享锁（IntentionSharedLock，IS锁）**：事务有意向对表中的某些记录加共享锁（S锁），加共享锁前必须先取得该表的IS锁。
- **意向排他锁（IntentionExclusiveLock，IX锁）**：事务有意向对表中的某些记录加排他锁（X锁），加排他锁之前必须先取得该表的IX锁。

意向锁是有数据引擎自己维护的，用户无法手动操作意向锁，在为数据行加共享/排他锁之前，InooDB会先获取该数据行所在在数据表的对应意向锁。

意向锁之间是互相兼容的。

|       | IS 锁 | IX 锁 |
| ----- | ----- | ----- |
| IS 锁 | 兼容  | 兼容  |
| IX 锁 | 兼容  | 兼容  |

意向锁和共享锁和排它锁互斥（这里指的是表级别的共享锁和排他锁，意向锁不会与行级的共享锁和排他锁互斥）。

|      | IS 锁 | IX 锁 |
| ---- | ----- | ----- |
| S 锁 | 兼容  | 互斥  |
| X 锁 | 互斥  | 互斥  |

《MySQL技术内幕InnoDB存储引擎》这本书对应的描述应该是笔误了。

![img](https://oss.javaguide.cn/github/javaguide/mysql/image-20220511171419081.png)

### 当前读和快照读有什么区别？

**快照读**（一致性非锁定读）就是单纯的SELECT语句，但不包括下面这两类SELECT语句：

```sql
SELECT...FORUPDATE
SELECT...LOCKINSHAREMODE
```

快照即记录的历史版本，每行记录可能存在多个历史版本（多版本技术）。快照读的情况下，如果读取的记录正在执行UPDATE/DELETE操作，读取操作不会因此去等待记录上X锁的释放，而是会去读取行的一个快照。只有在事务隔离级别RC(读取已提交)和RR（可重读）下，InnoDB才会使用一致性非锁定读：

- 在RC级别下，对于快照数据，一致性非锁定读总是读取被锁定行的最新一份快照数据。
- 在RR级别下，对于快照数据，一致性非锁定读总是读取本事务开始时的行数据版本。

快照读比较适合对于数据一致性要求不是特别高且追求极致性能的业务场景。**当前读**（一致性锁定读）就是给行记录加X锁或S锁。当前读的一些常见SQL语句类型如下：

```sql
# 对读的记录加一个X锁
SELECT...FOR UPDATE
# 对读的记录加一个S锁
SELECT...LOCK IN SHARE MODE
# 对修改的记录加一个X锁
INSERT...
UPDATE...
DELETE...
```

### 自增锁有了解吗？

> 不太重要的一个知识点，简单了解即可。

关系型数据库设计表的时候，通常会有一列作为自增主键。InnoDB中的自增主键会涉及一种比较特殊的表级锁—**自增锁（AUTO-INC Locks）**。

```sql
CREATE TABLE `sequence_id` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `stub` char(10) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  UNIQUE KEY `stub` (`stub`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

更准确点来说，不仅仅是自增主键，AUTO_INCREMENT的列都会涉及到自增锁，毕竟非主键也可以设置自增长。如果一个事务正在插入数据到有自增列的表时，会先获取自增锁，拿不到就可能会被阻塞住。这里的阻塞行为只是自增锁行为的其中一种，可以理解为自增锁就是一个接口，其具体的实现有多种。具体的配置项为innodb_autoinc_lock_mode（MySQL5.1.22引入），可以选择的值如下：

| innodb_autoinc_lock_mode |              介绍              |
| :----------------------: | :----------------------------: |
|            0             |            传统模式            |
|            1             | 连续模式（MySQL 8.0 之前默认） |
|            2             |  交错模式(MySQL 8.0 之后默认)  |

交错模式下，所有的“INSERT-LIKE”语句（所有的插入语句，包括：`INSERT`、`REPLACE`、`INSERT…SELECT`、`REPLACE…SELECT`、`LOADDATA`等）都不使用表级锁，使用的是轻量级互斥锁实现，多条插入语句可以并发执行，速度更快，扩展性也更好。不过，如果你的MySQL数据库有主从同步需求并且bin log存储格式为Statement的话，不要将InnoDB自增锁模式设置为交叉模式，不然会有数据不一致性问题。这是因为并发情况下插入语句的执行顺序就无法得到保障。

> 如果MySQL采用的格式为Statement，那么MySQL的主从同步实际上同步的就是一条一条的SQL语句。

最后，再推荐一篇文章：[为什么MySQL的自增主键不单调也不连续](https://draveness.me/whys-the-design-mysql-auto-increment/)。

### 总结

#### 数据库中的锁

表级锁包括：表锁、元数据锁、意向锁。
对于表锁而言，当存储引擎不支持行级锁时，使用表锁。SQL语句没有匹配到索引时，使用表锁。
对于元数据锁而言，对表做增删改查时，会加上MDL读锁。对表结构做变更时，会加上MDL写锁。
对于意向锁而言，对表中的行记录加锁时，会用到意向锁。
而对于行级锁而言，增删改查匹配到索引时，会使用行级锁

全局锁就是对整个数据库实例加锁。MySQL提供了一个加全局读锁的方法，命令是Flush tables with read lock(FTWRL)。当你需要让整个库处于只读状态的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。你可以理解为，全局锁基本上把数据所所有的变更语句都锁住了。全局锁的典型场景应用场景是全库逻辑备份，也就是把整个库每个表都select出来存起来。上面说到全局锁会锁住所有变更语句，但这只是对于MyISAM存储引擎而言的。对于Innodb而言，其可以利用MVCC实现数据的一致性视图，从而不需要锁整个库就可以实现全库的数据备份

表级锁可以分为：表锁、元数据锁、意向锁三种

- 表锁，顾名思义就是对某个表加锁。那什么时候会使用表锁呢？一般情况是对应的存储引擎没有行级锁（例如：MyIASM），或者是对应的SQL语句没有匹配到索引。

- 元数据锁
  元数据，指的是我们的表结构这些元数据。元数据锁（Metadata Lock）自然是执行DDL表结构变更语句时，我们对表加上的一个锁了。那什么时候会使用元数据锁这个表级锁呢？当我们对一个表做增删改查操作的时候，会加上MDL读锁；当我们要对表结构做变更时，就会加MDL写锁。

- 意向锁
  意向锁，本质上就是空间换时间的产物，是为了提高行锁效率的一个东西。在InnoDB中，我们对某条记录进行锁定时，为了提高并发度，通常都只是锁定这一行记录，而不是锁定整个表。而当我们需要为整个表加X锁的时候，我们就需要遍历整个表的记录，如果每条记录都没有被加锁，才可以给整个表加X锁。而这个遍历过程就很费时间，这时候就有了意向锁的诞生。意向锁其实就是标记这个表有没有被锁，如果有某条记录被锁住了，那么就必须获取该表的意向锁。所以当我们需要判断这个表的记录有没有被加锁时，直接判断意向锁就可以了，减少了遍历的时间，提高了效率，是典型的用空间换时间的做法。那么什么时候会用到意向锁呢？很简单，就是在对表中的行记录加锁的时候，就会用到意向锁。

那么什么时候会使用行级锁呢？
当增删改查匹配到索引时，Innodb会使用行级锁。

> [MySQL啥时候用表锁，啥时候用行锁？](https://mp.weixin.qq.com/s/v4GVad1wgW0H7tmjyQS0QA)

**如果查询条件用了索引/主键，那么`select ..... for update`就会进行行锁。**
**如果是普通字段(没有索引/主键)，那么`select ..... for update`就会进行锁表。**


#### 如何尽可能避免死锁

1. 合理的设计索引，区分度高的列放到组合索引前面，使业务SQL尽可能通过索引定位更少的行，减少锁竞争。
2. 调整业务逻辑SQL执行顺序，避免update/delete长时间持有锁的SQL在事务前面。
3. 避免大事务，尽量将大事务拆成多个小事务来处理，小事务发生锁冲突的几率也更小。
4. 以固定的顺序访问表和行。比如两个更新数据的事务，事务A更新数据的顺序为1，2;事务B更新数据的顺序为2，1。这样更可能会造成死锁。
5. 在并发比较高的系统中，不要显式加锁，特别是是在事务里显式加锁。如select…for update语句，如果是在事务里（运行了start transaction或设置了auto commit等于0）,那么就会锁定所查找到的记录。
6. 尽量按主键/索引去查找记录，范围查找增加了锁冲突的可能性，也不要利用数据库做一些额外额度计算工作。比如有的程序会用到“select…where…order by rand();”这样的语句，由于类似这样的语句用不到索引，因此将导致整个表的数据都被锁住。
7. 优化SQL和表设计，减少同时占用太多资源的情况。比如说，减少连接的表，将复杂SQL分解为多个简单的SQL。

#### 相关文章

- [这六个MySQL死锁案例，能让你理解死锁的原因！](https://mp.weixin.qq.com/s/7BuvuRFuelBTI2rn2If6cA)
- [面试命中率90%的点：MySQL锁](https://mp.weixin.qq.com/s/3EU5Yfd3O6i1CA-YvhqoVg)
- [一文搞懂MySQL中各种锁，写的太好了](https://mp.weixin.qq.com/s/5lX30br8IBStcnk2J1PEXg)

## MySQL索引

### 索引介绍

**索引是一种用于快速查询和检索数据的数据结构，其本质可以看成是一种排序好的数据结构。**

索引的作用就相当于书的目录。打个比方:我们在查字典的时候，如果没有目录，那我们就只能一页一页的去找我们需要查的那个字，速度很慢。如果有目录了，我们只需要先去目录里查找字的位置，然后直接翻到那一页就行了。索引底层数据结构存在很多种类型，常见的索引结构有:B树，B+树和Hash、红黑树。在MySQL中，无论是Innodb还是MyIsam，都使用了B+树作为索引结构。

### 索引的优缺点

**优点**：

- 使用索引可以大大加快数据的检索速度（大大减少检索的数据量）,这也是创建索引的最主要的原因。
- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。

**缺点**：

- 创建索引和维护索引需要耗费许多时间。当对表中的数据进行增删改的时候，如果数据有索引，那么索引也需要动态的修改，会降低SQL执行效率。
- 索引需要使用物理文件存储，也会耗费一定空间。

但是，**使用索引一定能提高查询性能吗**?大多数情况下，索引查询都是比全表扫描要快的。但是如果数据库的数据量不大，那么使用索引也不一定能够带来很大提升。

### 索引的底层数据结构

#### Hash表

哈希表是键值对的集合，通过键(key)即可快速取出对应的值(value)，因此哈希表可以快速检索数据（接近O（1））。

**为何能够通过key快速取出value呢**？原因在于哈希算法（也叫散列算法）。通过哈希算法，我们可以快速找到key对应的index，找到了index也就找到了对应的value。


```java
hash = hashfunc(key)
index = hash % array_size
```

![img](https://oss.javaguide.cn/github/javaguide/database/mysql20210513092328171.png)

但是哈希算法有个**Hash冲突**问题，也就是说多个不同的key最后得到的index相同。通常情况下，我们常用的解决办法是**链地址法**。链地址法就是将哈希冲突数据存放在链表中。就比如JDK1.8之前HashMap就是通过链地址法来解决哈希冲突的。不过，JDK1.8以后HashMap为了减少链表过长的时候搜索时间过长引入了红黑树。

![img](https://oss.javaguide.cn/github/javaguide/database/mysql20210513092224836.png)

为了减少Hash冲突的发生，一个好的哈希函数应该“均匀地”将数据分布在整个可能的哈希值集合中。

既然哈希表这么快，**为什么MySQL没有使用其作为索引的数据结构呢**？主要是因为Hash索引不支持顺序和范围查询。假如我们要对表中的数据进行排序或者进行范围查询，那Hash索引可就不行了。并且，每次IO只能取一个。

试想一种情况:


```java
SELECT * FROM tb1 WHERE id < 500;
```

在这种范围查询中，优势非常大，直接遍历比500小的叶子节点就够了。而Hash索引是根据hash算法来定位的，难不成还要把1-499的数据，每个都进行一次hash计算来定位吗?这就是Hash最大的缺点了。

#### B树&B+树

B树也称B-树,全称为**多路平衡查找树**，B+树是B树的一种变体。B树和B+树中的B是Balanced（平衡）的意思。目前大部分数据库系统及文件系统都采用B-Tree或其变种B+Tree作为索引结构。

**B树&B+树两者有何异同呢？**

- B树的所有节点既存放键(key)也存放数据(data)，而B+树只有叶子节点存放key和data，其他内节点只存放key。
- B树的叶子节点都是独立的;B+树的叶子节点有一条引用链指向与它相邻的叶子节点。
- B树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了。而B+树的检索效率就很稳定了，任何查找都是从根节点到叶子节点的过程，叶子节点的顺序检索很明显。

在MySQL中，MyISAM引擎和InnoDB引擎都是使用B+Tree作为索引结构，但是，两者的实现方式不太一样。（下面的内容整理自《Java工程师修炼之道》）

> MyISAM引擎中，B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则取出其data域的值，然后以data域的值为地址读取相应的数据记录。这被称为“**非聚簇索引（非聚集索引）**”。
>
> InnoDB引擎中，其数据文件本身就是索引文件。相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。这被称为**聚簇索引（聚集索引）**，而其余的索引都作为**辅助索引**，辅助索引的data域存储相应记录主键的值而不是地址，这也是和MyISAM不同的地方。在根据主索引搜索时，直接找到key所在的节点即可取出数据；在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。因此，在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。

### 索引类型总结

**按照数据结构维度划分**：

- BTree索引：MySQL里默认和最常用的索引类型。只有叶子节点存储value，非叶子节点只有指针和key。存储引擎MyISAM和InnoDB实现BTree索引都是使用B+Tree，但二者实现方式不一样（前面已经介绍了）。
- 哈希索引：类似键值对的形式，一次即可定位。
- RTree索引：一般不会使用，仅支持geometry数据类型，优势在于范围查找，效率较低，通常使用搜索引擎如ElasticSearch代替。
- 全文索引：对文本的内容进行分词，进行搜索。目前只有`CHAR`、`VARCHAR`，`TEXT`列上可以创建全文索引。一般不会使用，效率较低，通常使用搜索引擎如ElasticSearch代替。

**按照底层存储方式角度划分**：

- 聚簇索引（聚集索引）：索引结构和数据一起存放的索引，InnoDB中的主键索引就属于聚簇索引。
- 非聚簇索引（非聚集索引）：索引结构和数据分开存放的索引，二级索引(辅助索引)就属于非聚簇索引。MySQL的MyISAM引擎，不管主键还是非主键，使用的都是非聚簇索引。

**按照应用维度划分**：

- 主键索引：加速查询+列值唯一（不可以有NULL）+表中只有一个。
- 普通索引：仅加速查询。
- 唯一索引：加速查询+列值唯一（可以有NULL）。
- 覆盖索引：一个索引包含（或者说覆盖）所有需要查询的字段的值。
- 联合索引：多列值组成一个索引，专门用于组合搜索，其效率大于索引合并。
- 全文索引：对文本的内容进行分词，进行搜索。目前只有`CHAR`、`VARCHAR`，`TEXT`列上可以创建全文索引。一般不会使用，效率较低，通常使用搜索引擎如ElasticSearch代替。

MySQL8.x中实现的索引新特性：

- 隐藏索引：也称为不可见索引，不会被优化器使用，但是仍然需要维护，通常会软删除和灰度发布的场景中使用。主键不能设置为隐藏（包括显式设置或隐式设置）。
- 降序索引：之前的版本就支持通过desc来指定索引为降序，但实际上创建的仍然是常规的升序索引。直到MySQL8.x版本才开始真正支持降序索引。另外，在MySQL8.x版本中，不再对GROUPBY语句进行隐式排序。
- 函数索引：从MySQL8.0.13版本开始支持在索引中使用函数或者表达式的值，也就是在索引中可以包含函数或者表达式。

### 主键索引(Primary Key)

数据表的主键列使用的就是主键索引。一张数据表有只能有一个主键，并且主键不能为null，不能重复。在MySQL的InnoDB的表中，当没有显示的指定表的主键时，InnoDB会自动先检查表中是否有唯一索引且不允许存在null值的字段，如果有，则选择该字段为默认的主键，否则InnoDB将会自动创建一个6Byte的自增主键。

![img](https://oss.javaguide.cn/github/javaguide/open-source-project/cluster-index.png)

### 二级索引

**二级索引（SecondaryIndex）又称为辅助索引，是因为二级索引的叶子节点存储的数据是主键。也就是说，通过二级索引，可以定位主键的位置**。

唯一索引，普通索引，前缀索引等索引属于二级索引。

1. **唯一索引（UniqueKey）**：唯一索引也是一种约束。唯一索引的属性列不能出现重复的数据，但是允许数据为NULL，一张表允许创建多个唯一索引。建立唯一索引的目的大部分时候都是为了该属性列的数据的唯一性，而不是为了查询效率。
2. **普通索引（Index）**：普通索引的唯一作用就是为了快速查询数据，一张表允许创建多个普通索引，并允许数据重复和NULL。
3. **前缀索引（Prefix）**：前缀索引只适用于字符串类型的数据。前缀索引是对文本的前几个字符创建索引，相比普通索引建立的数据更小，因为只取前几个字符。
4. **全文索引（FullText）**：全文索引主要是为了检索大文本数据中的关键字的信息，是目前搜索引擎数据库使用的一种技术。Mysql5.6之前只有MYISAM引擎支持全文索引，5.6之后InnoDB也支持了全文索引。

二级索引:

![img](https://oss.javaguide.cn/github/javaguide/open-source-project/no-cluster-index.png)

### 聚簇索引与非聚簇索引

#### 聚簇索引（聚集索引）

**聚簇索引介绍**

**聚簇索引（ClusteredIndex）即索引结构和数据一起存放的索引，并不是一种单独的索引类型。InnoDB中的主键索引就属于聚簇索引**。

在MySQL中，InnoDB引擎的表的`.ibd`文件就包含了该表的索引和数据，对于InnoDB引擎表来说，该表的索引(B+树)的每个非叶子节点存储索引，叶子节点存储索引和索引对应的数据。

**聚簇索引的优缺点**

**优点**：

- **查询速度非常快**：聚簇索引的查询速度非常的快，因为整个B+树本身就是一颗多叉平衡树，叶子节点也都是有序的，定位到索引的节点，就相当于定位到了数据。相比于非聚簇索引，聚簇索引少了一次读取数据的IO操作。
- **对排序查找和范围查找优化**：聚簇索引对于主键的排序查找和范围查找速度非常快。

**缺点**：

- **依赖于有序的数据**：因为B+树是多路平衡树，如果索引的数据不是有序的，那么就需要在插入时排序，如果数据是整型还好，否则类似于字符串或UUID这种又长又难比较的数据，插入或查找的速度肯定比较慢。
- **更新代价大**：如果对索引列的数据被修改时，那么对应的索引也将会被修改，而且聚簇索引的叶子节点还存放着数据，修改代价肯定是较大的，所以对于主键索引来说，主键一般都是不可被修改的。

#### 非聚簇索引（非聚集索引）

**非聚簇索引介绍**

非聚簇索引(Non-ClusteredIndex)即索引结构和数据分开存放的索引，并不是一种单独的索引类型。二级索引(辅助索引)就属于非聚簇索引。MySQL的MyISAM引擎，不管主键还是非主键，使用的都是非聚簇索引。

非聚簇索引的叶子节点并不一定存放数据的指针，因为二级索引的叶子节点就存放的是主键，根据主键再回表查数据。

**非聚簇索引的优缺点**

**优点**：

更新代价比聚簇索引要小。非聚簇索引的更新代价就没有聚簇索引那么大了，非聚簇索引的叶子节点是不存放数据的

**缺点**：

- **依赖于有序的数据**：跟聚簇索引一样，非聚簇索引也依赖于有序的数据
- **可能会二次查询（回表）**：这应该是非聚簇索引最大的缺点了。当查到索引对应的指针或主键后，可能还需要根据指针或主键再到数据文件或表中查询。

这是MySQL的表的文件截图:

![img](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165311654.png)

聚簇索引和非聚簇索引:

![img](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165326946.png)

**非聚簇索引一定回表查询吗(覆盖索引)?非聚簇索引不一定回表查询**。试想一种情况，用户准备使用SQL查询用户名，而用户名字段正好建立了索引。


```sql
SELECT name FROM table WHERE name='guang19';
```

那么这个索引的key本身就是name，查到对应的name直接返回就行了，无需回表查询。即使是MYISAM也是这样，虽然MYISAM的主键索引确实需要回表，因为它的主键索引的叶子节点存放的是指针。但是！**如果SQL查的就是主键呢**？


```sql
SELECT id FROM table WHERE id=1;
```

主键索引本身的key就是主键，查到返回就行了。这种情况就称之为覆盖索引了。

### 覆盖索引和联合索引

#### 覆盖索引

如果一个索引包含（或者说覆盖）所有需要查询的字段的值，我们就称之为**覆盖索引（CoveringIndex）**。我们知道在InnoDB存储引擎中，如果不是主键索引，叶子节点存储的是主键+列值。最终还是要“回表”，也就是要通过主键再查找一次。这样就会比较慢覆盖索引就是把要查询出的列和索引是对应的，不做回表操作！覆盖索引即需要查询的字段正好是索引的字段，那么直接根据该索引，就可以查到数据了，而无需回表查询。

> 如主键索引，如果一条SQL需要查询主键，那么正好根据主键索引就可以查到主键。再如普通索引，如果一条SQL需要查询name，name字段正好有索引，那么直接根据这个索引就可以查到数据，也无需回表。

![覆盖索引](https://oss.javaguide.cn/github/javaguide/database/mysql20210420165341868.png)

我们这里简单演示一下覆盖索引的效果。

1、创建一个名为cus_order的表，来实际测试一下这种排序方式。为了测试方便，cus_order这张表只有id、score、name这3个字段。


```sql
CREATE TABLE cus_order (
  id int(11) unsigned NOT NULL AUTO_INCREMENT,
  score int(11) NOT NULL,
  name varchar(11) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=100000 DEFAULT CHARSET=utf8mb4;
```

2、定义一个简单的存储过程（PROCEDURE）来插入100w测试数据。

```sql
DELIMITER ;;
CREATE DEFINER=`root`@`%` PROCEDURE `BatchinsertDataToCusOder`(IN start_num INT,IN max_num INT)
BEGIN
      DECLARE i INT default start_num;
      WHILE i < max_num DO
          insert into cus_order(id, score, name)
          values (i,RAND() * 1000000,CONCAT('user', i));
          SET i = i + 1;
      END WHILE;
  END;;
DELIMITER ;
```

存储过程定义完成之后，我们执行存储过程即可！

```sql
-- 插入100w+的随机数据
CALL BatchinsertDataToCusOder(1, 1000000);
```

等待一会，100w的测试数据就插入完成了！

3、创建覆盖索引并使用EXPLAIN命令分析。

为了能够对这100w数据按照score进行排序，我们需要执行下面的SQL语句。

```sql
-- 降序排序
SELECT score,name FROM cus_order ORDER BY score DESC;
```

使用EXPLAIN命令分析这条SQL语句，通过Extra这一列的Using filesort，我们发现是没有用到覆盖索引的。

![img](https://oss.javaguide.cn/github/javaguide/mysql/not-using-covering-index-demo.png)

不过这也是理所应当，毕竟我们现在还没有创建索引呢！

我们这里以score和name两个字段建立联合索引：


```sql
ALTER TABLE cus_order ADD INDEX id_score_name(score, name);
```

创建完成之后，再用EXPLAIN命令分析再次分析这条SQL语句。

![img](https://oss.javaguide.cn/github/javaguide/mysql/using-covering-index-demo.png)

通过Extra这一列的Using index，说明这条SQL语句成功使用了覆盖索引。

> 关于EXPLAIN命令的详细介绍请看：[MySQL执行计划分析](https://javaguide.cn/database/mysql/mysql-query-execution-plan.html)这篇文章。

#### 联合索引

使用表中的多个字段创建索引，就是**联合索引**，也叫**组合索引**或**复合索引**。

以score和name两个字段建立联合索引：

```sql
ALTER TABLE `cus_order` ADD INDEX id_score_name(score, name);
```

#### 最左前缀匹配原则

最左前缀匹配原则指的是，在使用联合索引时，MySQL会根据联合索引中的字段顺序，从左到右依次到查询条件中去匹配，如果查询条件中存在与联合索引中最左侧字段相匹配的字段，则就会使用该字段过滤一批数据，直至联合索引中全部字段匹配完成，或者在执行过程中遇到范围查询（如">"、"<"）才会停止匹配。对于">="、"<="、"BETWEEN"、"like"前缀匹配的范围查询，并不会停止匹配。所以，我们在使用联合索引时，可以将区分度高的字段放在最左边，这也可以过滤更多数据。

> 相关阅读：[联合索引的最左匹配原则,全网都在说的一个错误结论](https://mp.weixin.qq.com/s/8qemhRg5MgXs1So5YCv0fQ)。

### 索引下推

索引下推（IndexConditionPushdown）是MySQL5.6版本中提供的一项索引优化功能，可以在非聚簇索引遍历过程中，对索引中包含的字段先做判断，过滤掉不符合条件的记录，减少回表次数。

### 正确使用索引的一些建议

**选择合适的字段创建索引**

- **不为NULL的字段**：索引字段的数据应该尽量不为NULL，因为对于数据为NULL的字段，数据库较难优化。如果字段频繁被查询，但又避免不了为NULL，建议使用0,1,true,false这样语义较为清晰的短值或短字符作为替代。
- **被频繁查询的字段**：我们创建索引的字段应该是查询操作非常频繁的字段。
- **被作为条件查询的字段**：被作为WHERE条件查询的字段，应该被考虑建立索引。
- **频繁需要排序的字段**：索引已经排序，这样查询可以利用索引的排序，加快排序查询时间。
- **被经常频繁用于连接的字段**：经常用于连接的字段可能是一些外键列，对于外键列并不一定要建立外键，只是说该列涉及到表与表的关系。对于频繁被连接查询的字段，可以考虑建立索引，提高多表连接查询的效率。

**被频繁更新的字段应该慎重建立索引**

虽然索引能带来查询上的效率，但是维护索引的成本也是不小的。如果一个字段不被经常查询，反而被经常修改，那么就更不应该在这种字段上建立索引了。

**限制每张表上的索引数量**

索引并不是越多越好，建议单张表索引不超过5个！索引可以提高效率同样可以降低效率。索引可以增加查询效率，但同样也会降低插入和更新的效率，甚至有些情况下会降低查询效率。因为MySQL优化器在选择如何优化查询时，会根据统一信息，对每一个可以用到的索引来进行评估，以生成出一个最好的执行计划，如果同时有很多个索引都可以用于查询，就会增加MySQL优化器生成执行计划的时间，同样会降低查询性能。

**尽可能的考虑建立联合索引而不是单列索引**

因为索引是需要占用磁盘空间的，可以简单理解为每个索引都对应着一颗B+树。如果一个表的字段过多，索引过多，那么当这个表的数据达到一个体量后，索引占用的空间也是很多的，且修改索引时，耗费的时间也是较多的。如果是联合索引，多个字段在一个索引上，那么将会节约很大磁盘空间，且修改数据的操作效率也会提升。

**注意避免冗余索引**

冗余索引指的是索引的功能相同，能够命中索引(a,b)就肯定能命中索引(a)，那么索引(a)就是冗余索引。如（name,city）和（name）这两个索引就是冗余索引，能够命中前者的查询肯定是能够命中后者的在大多数情况下，都应该尽量扩展已有的索引而不是创建新索引。

**字符串类型的字段使用前缀索引代替普通索引**

前缀索引仅限于字符串类型，较普通索引会占用更小的空间，所以可以考虑使用前缀索引带替普通索引。

**避免索引失效**

索引失效也是慢查询的主要原因之一，常见的导致索引失效的情况有下面这些：

- 使用SELECT*进行查询;
- 创建了组合索引，但查询条件未遵守最左匹配原则;
- 在索引列上进行计算、函数、类型转换等操作;
- 以%开头的LIKE查询比如like'%abc';
- 查询条件中使用or，且or的前后条件中有一个列没有索引，涉及的索引都不会被使用到;
- 发生[隐式转换](https://javaguide.cn/database/mysql/index-invalidation-caused-by-implicit-conversion.html);
- ......

**删除长期未使用的索引**

删除长期未使用的索引，不用的索引的存在会造成不必要的性能损耗。MySQL5.7可以通过查询sys库的schema_unused_indexes视图来查询哪些索引从未被使用。

**知道如何分析语句是否走索引查询**

我们可以使用EXPLAIN命令来分析SQL的执行计划，这样就知道语句是否命中索引了。执行计划是指一条SQL语句在经过MySQL查询优化器的优化会后，具体的执行方式。EXPLAIN并不会真的去执行相关的语句，而是通过查询优化器对语句进行分析，找出最优的查询方案，并显示对应的信息。EXPLAIN的输出格式如下：

```sql
mysql> EXPLAIN SELECT `score`,`name` FROM `cus_order` ORDER BY `score` DESC;
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
| id | select_type | table     | partitions | type | possible_keys | key  | key_len | ref  | rows   | filtered | Extra          |
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
|  1 | SIMPLE      | cus_order | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 997572 |   100.00 | Using filesort |
+----+-------------+-----------+------------+------+---------------+------+---------+------+--------+----------+----------------+
1 row in set, 1 warning (0.00 sec)
```

各个字段的含义如下：

|   **列名**    |                   **含义**                   |
| :-----------: | :------------------------------------------: |
|      id       |           SELECT 查询的序列标识符            |
|  select_type  |         SELECT 关键字对应的查询类型          |
|     table     |                  用到的表名                  |
|  partitions   |    匹配的分区，对于未分区的表，值为 NULL     |
|     type      |                 表的访问方法                 |
| possible_keys |                可能用到的索引                |
|      key      |                实际用到的索引                |
|    key_len    |                所选索引的长度                |
|      ref      | 当使用索引等值查询时，与索引作比较的列或常量 |
|     rows      |               预计要读取的行数               |
|   filtered    |     按表条件过滤后，留存的记录数的百分比     |
|     Extra     |                   附加信息                   |

> 篇幅问题，我这里只是简单介绍了一下MySQL执行计划，详细介绍请看：[MySQL执行计划分析](https://javaguide.cn/database/mysql/mysql-query-execution-plan.html)这篇文章
> [原文链接](https://javaguide.cn/database/mysql/mysql-index.html)


### 总结

索引什么情况下会失效
1. 使用!=或者<>或者is null判断导致索引失效
2. 类型不一致导致的索引失效，如主键为id，其他表的字段为这个表的主键,字段类型却不一样
3. 函数导致的索引失效，如SUBSTRING(name,1,2) = 'wise'需要全表扫描
4. 运算符导致的索引失效
5. OR引起的索引失效，因为如果OR判断的字段中有一个没有索引的话,引擎会放弃索引而产生全表扫描
6. 模糊搜索LIKE导致的索引失效
7. NOT IN、NOT EXISTS导致索引失效
8. IS NULL不走索引，IS NOT NULL走索引
9. 联合索引如果不遵循最左前缀原则，那么索引也将失效
10. 索引列如果使用了隐式转换也会导致索引失效

### 相关文章

| [如何防止MySQL索引失效](https://mp.weixin.qq.com/s/B1Dr_w3oeIsFTRtZCKS5Ow) | [你设计索引的原则是什么？怎么避免索引失效？](https://mp.weixin.qq.com/s/wyotVRKbBJ7LGdqFSxZOFg) | [超全的数据库建表/SQL/索引规范，建议贴在工位上！](https://mp.weixin.qq.com/s/k7AXALxURez5nS34KZZIxA) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL索引扫盲总结](https://mp.weixin.qq.com/s/hsvdbGXBcHPA0JOO7gkpjw) | [为什么Mysql的常用引擎都默认使用B+树作为索引？](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491020&idx=1&sn=f703a7a54cf9bccd8aed6dd384515945&source=41#wechat_redirect) | [为什么MySQL使用B+树，而不是B树或者Hash？](https://mp.weixin.qq.com/s/Ii9gF-TZZ84WghB8Xtwxig) |
| [MySQL索引底层：B+树详解](https://mp.weixin.qq.com/s/opbgvTX_GDytR-MS3-6yEA) | [为什么Mongodb索引用B树，而Mysql用B+树?](https://mp.weixin.qq.com/s/ZbkRWFT5rXIA9ZXzwTZTTA) | [腾讯面试官用「B+树」虐哭我了](https://mp.weixin.qq.com/s/I6g_O2lq3iI6DdnxFGGsuA) |
| [面试官:谈谈你对mysql索引的认识？](https://mp.weixin.qq.com/s?__biz=MzIwMDgzMjc3NA==&mid=2247484720&idx=1&sn=7bd7774058e7886eeb3dedb38aa8657a&chksm=96f66759a181ee4f4c177a755c3ac6b6e97fef148bbf4afea8616f4edec33bf6d4f18cda9f69&scene=21#wechat_redirect) | [面试的时候，如果你没掌握索引，绝对没戏！](https://mp.weixin.qq.com/s/RfA8r3IXECqmvvTJX4sz0w) | [什么是MySQ索引?](https://mp.weixin.qq.com/s/yxS4tpX_6fz9LBsh0UoHpw) |
| [MySQL的索引实现原理](https://mp.weixin.qq.com/s/1RdEIq4EDYpIge_84dunEw) | [浅入浅出MySQL索引](https://mp.weixin.qq.com/s/b5xc6mRzzqg5Bjd5scJ_ug) | [别再一知半解啦，索引其实就这么回事](https://mp.weixin.qq.com/s/F1EY0hY9WrzmeRp3C5C9Pw) |
| [MySQL索引凭什么让查询效率提高这么多？](https://mp.weixin.qq.com/s/HUy5RPBsxhu7g5Sr-l3XlQ) | [索引为什么能提高查询性能....](https://mp.weixin.qq.com/s/6N-Do7Av6y6VP_vK9CVK3A) | [常见索引类型](https://mp.weixin.qq.com/s/YRbHVNeJNjvQcgC4PunW6w) |
| [主键索引就是聚集索引？MySQL索引类型大梳理](https://mp.weixin.qq.com/s/iS8V65my03EQtOQAkxfMag) | [再聊MySQL聚簇索引](https://mp.weixin.qq.com/s/F0cEzIqecF4sWg7ZRmHKRQ) | [MySQL主键自增也有坑?innodb_autoinc_lock_mode](https://mp.weixin.qq.com/s/5EymS2IyB7yyYsxX5UWrtw) |
| [再有人问你MySQL索引原理，就把这篇文章甩给他！](https://mp.weixin.qq.com/s/9yeModGuGvDu5S0bW9sU6w) | [为什么索引可以让查询变快？终于有人说清楚了！](https://mp.weixin.qq.com/s/dKvPKUVDTM1OiP_qzNiFBg) | [面试官问我索引为什么这快？我好像解释不清楚了](https://mp.weixin.qq.com/s/EyYBqfZcBWP60Y1gAADeOg) |
| [一文讲清，MySQL中的二级索引](https://mp.weixin.qq.com/s/0_4Jq06LLaTEfjetLQP0iA) | [明明加了唯一索引，为什么还是产生重复数据？](https://mp.weixin.qq.com/s/_hdGKhB-A4ZOFZ42POmgjQ) | [面试官提问：什么是前缀索引？](https://mp.weixin.qq.com/s/4XfSv004Vy7hyvZsjlQXwQ) |
| [MySQL遵循最左前缀匹配原则！面试官：回去等通知吧](https://mp.weixin.qq.com/s/IYRTE00_3bXD6y3YBW9P6Q) | [MySQL索引15连问](https://mp.weixin.qq.com/s/N95kMxVduiA6ZA3TSexAtA) | [MySQL索引数据结构入门](https://mp.weixin.qq.com/s/QLQvMT2sPmmjVE_pXj5FIA) |
| [前缀索引，在性能和空间中寻找平衡](https://mp.weixin.qq.com/s/mqi7MyF183FUmgy2FXi3Tw) | [聊聊mysql优化神器：前缀索引！](https://mp.weixin.qq.com/s/bhlzlILDSpLkiRBfVyZj0w) | [全文索引](https://mp.weixin.qq.com/s/SLIrSYnrNYkTyioE4syhFA) |
| [MySQL模糊查询再也不用like+%了(全文索引)](https://mp.weixin.qq.com/s/L68fygoNmvmRuYSmOkHROA) |                                                              |                                                              |

## MySQL三大日志

MySQL日志主要包括错误日志、查询日志、慢查询日志、事务日志、二进制日志几大类。其中，比较重要的还要属二进制日志bin log（归档日志）和事务日志redo log（重做日志）和undo log（回滚日志）。

![img](https://oss.javaguide.cn/github/javaguide/01.png)

今天就来聊聊redo log（重做日志）、bin log（归档日志）、两阶段提交、undo log（回滚日志）。

### redo log

redo log（重做日志）是InnoDB存储引擎独有的，它让MySQL拥有了崩溃恢复能力。比如MySQL实例挂了或宕机了，重启时，InnoDB存储引擎会使用redo log恢复数据，保证数据的持久性与完整性。

![img](https://oss.javaguide.cn/github/javaguide/02.png)

MySQL中数据是以页为单位，你查询一条记录，会从硬盘把一页的数据加载出来，加载出来的数据叫数据页，会放入到Buffer Pool中。后续的查询都是先从Buffer Pool中找，没有命中再去硬盘加载，减少硬盘IO开销，提升性能。更新表数据的时候，也是如此，发现Buffer Pool里存在要更新的数据，就直接在Buffer Pool里更新。然后会把“在某个数据页上做了什么修改”记录到重做日志缓存（redo log buffer）里，接着刷盘到redo log文件里。

![img](https://oss.javaguide.cn/github/javaguide/03.png)

> 图片笔误提示：第4步“清空redo log buffe刷盘到redo日志中”这句话中的buffe应该是buffer。

理想情况，事务一提交就会进行刷盘操作，但实际上，刷盘的时机是根据策略来进行的。

> 小贴士：每条redo记录由“表空间号+数据页号+偏移量+修改数据长度+具体修改的数据”组成

#### 刷盘时机

InnoDB存储引擎为redo log的刷盘策略提供了innodb_flush_log_at_trx_commit参数，它支持三种策略：

- **0**：设置为0的时候，表示每次事务提交时不进行刷盘操作
- **1**：设置为1的时候，表示每次事务提交时都将进行刷盘操作（默认值）
- **2**：设置为2的时候，表示每次事务提交时都只把redo log buffer内容写入page cache

innodb_flush_log_at_trx_commit参数默认为1，也就是说当事务提交时会调用fsync对redo log进行刷盘。另外，InnoDB存储引擎有一个后台线程，每隔1秒，就会把redo log buffer中的内容写到文件系统缓存（page cache），然后调用fsync刷盘。

![img](https://oss.javaguide.cn/github/javaguide/04.png)

也就是说，一个没有提交事务的redo log记录，也可能会刷盘。

**为什么呢？**

因为在事务执行过程redo log记录是会写入redo log buffer中，这些redo log记录会被后台线程刷盘。

![img](https://oss.javaguide.cn/github/javaguide/05.png)

除了后台线程每秒1次的轮询操作，还有一种情况，当redo log buffer占用的空间即将达到innodb_log_buffer_size一半的时候，后台线程会主动刷盘。下面是不同刷盘策略的流程图。

**innodb_flush_log_at_trx_commit=0**

![img](https://oss.javaguide.cn/github/javaguide/06.png)

为0时，如果MySQL挂了或宕机可能会有1秒数据的丢失。

**innodb_flush_log_at_trx_commit=1**

![img](https://oss.javaguide.cn/github/javaguide/07.png)

为1时，只要事务提交成功，redo log记录就一定在硬盘里，不会有任何数据丢失。如果事务执行期间MySQL挂了或宕机，这部分日志丢了，但是事务并没有提交，所以日志丢了也不会有损失。

**innodb_flush_log_at_trx_commit=2**

![img](https://oss.javaguide.cn/github/javaguide/09.png)

为2时，只要事务提交成功，redo log buffer中的内容只写入文件系统缓存（page cache）。如果仅仅只是MySQL挂了不会有任何数据丢失，但是宕机可能会有1秒数据的丢失。

#### 日志文件组

硬盘上存储的redo log日志文件不只一个，而是以一个**日志文件组**的形式出现的，每个的redo日志文件大小都是一样的。比如可以配置为一组4个文件，每个文件的大小是1GB，整个redo log日志文件组可以记录4G的内容。它采用的是环形数组形式，从头开始写，写到末尾又回到头循环写，如下图所示。

![img](https://oss.javaguide.cn/github/javaguide/10.png)

在个**日志文件组**中还有两个重要的属性，分别是write pos、checkpoint

- **write pos**是当前记录的位置，一边写一边后移
- **checkpoint**是当前要擦除的位置，也是往后推移

每次刷盘redo log记录到**日志文件组**中，write pos位置就会后移更新。每次MySQL加载**日志文件组**恢复数据时，会清空加载过的redo log记录，并把checkpoint后移更新。write pos和checkpoint之间的还空着的部分可以用来写入新的redo log记录。

![img](https://oss.javaguide.cn/github/javaguide/11.png)

如果write pos追上checkpoint，表示**日志文件组**满了，这时候不能再写入新的redo log记录，MySQL得停下来，清空一些记录，把checkpoint推进一下。

![img](https://oss.javaguide.cn/github/javaguide/12.png)

#### redo log小结

相信大家都知道redo log的作用和它的刷盘时机、存储形式。现在我们来思考一个问题：**只要每次把修改后的数据页直接刷盘不就好了，还有redo log什么事**？它们不都是刷盘么？差别在哪里？


```java
1 Byte = 8bit
1 KB = 1024 Byte
1 MB = 1024 KB
1 GB = 1024 MB
1 TB = 1024 GB
```

实际上，数据页大小是16KB，刷盘比较耗时，可能就修改了数据页里的几Byte数据，有必要把完整的数据页刷盘吗？而且数据页刷盘是随机写，因为一个数据页对应的位置可能在硬盘文件的随机位置，所以性能是很差。如果是写redo log，一行记录可能就占几十Byte，只包含表空间号、数据页号、磁盘文件偏移量、更新值，再加上是顺序写，所以刷盘速度很快。所以用redo log形式记录修改内容，性能会远远超过刷数据页的方式，这也让数据库的并发能力更强。

> 其实内存的数据页在一定时机也会刷盘，我们把这称为页合并，讲Buffer Pool的时候会对这块细说

### bin log

redo log它是物理日志，记录内容是“在某个数据页上做了什么修改”，属于InnoDB存储引擎。而bin log是逻辑日志，记录内容是语句的原始逻辑，类似于“给ID=2这一行的c字段加1”，属于MySQL Server层。不管用什么存储引擎，只要发生了表数据更新，都会产生bin log日志。那bin log到底是用来干嘛的？可以说MySQL数据库的**数据备份、主备、主主、主从**都离不开bin log，需要依靠bin log来同步数据，保证数据一致性。

![img](https://oss.javaguide.cn/github/javaguide/01-20220305234724956.png)

bin log会记录所有涉及更新数据的逻辑操作，并且是顺序写。

#### 记录格式

bin log日志有三种格式，可以通过bin log_format参数指定。

- **statement**
- **row**
- **mixed**

指定statement，记录的内容是SQL语句原文，比如执行一条`update T set update_time=now( )where id=1`，记录的内容如下。

![img](https://oss.javaguide.cn/github/javaguide/02-20220305234738688.png)

同步数据时，会执行记录的SQL语句，但是有个问题，update_time=now()这里会获取当前系统时间，直接执行会导致与原库的数据不一致。为了解决这种问题，我们需要指定为row，记录的内容不再是简单的SQL语句了，还包含操作的具体数据，记录内容如下。

![img](https://oss.javaguide.cn/github/javaguide/03-20220305234742460.png)

row格式记录的内容看不到详细信息，要通过mysql bin log工具解析出来。update_time=now()变成了具体的时间update_time=1627112756247，条件后面的@1、@2、@3都是该行数据第1个~3个字段的原始值（**假设这张表只有3个字段**）。这样就能保证同步数据的一致性，通常情况下都是指定为row，这样可以为数据库的恢复与同步带来更好的可靠性。但是这种格式，需要更大的容量来记录，比较占用空间，恢复与同步时会更消耗IO资源，影响执行速度。所以就有了一种折中的方案，指定为mixed，记录的内容是前两者的混合。MySQL会判断这条SQL语句是否可能引起数据不一致，如果是，就用row格式，否则就用statement格式。

#### 写入机制

bin log的写入时机也非常简单，事务执行过程中，先把日志写到bin log cache，事务提交的时候，再把bin log cache写到bin log文件中。因为一个事务的bin log不能被拆开，无论这个事务多大，也要确保一次性写入，所以系统会给每个线程分配一个块内存作为bin log cache。我们可以通过bin log_cache_size参数控制单个线程bin log cache大小，如果存储内容超过了这个参数，就要暂存到磁盘（Swap）。bin log日志刷盘流程如下

![img](https://oss.javaguide.cn/github/javaguide/04-20220305234747840.png)

- **上图的write，是指把日志写入到文件系统的page cache，并没有把数据持久化到磁盘，所以速度比较快**
- **上图的fsync，才是将数据持久化到磁盘的操作**

write和fsync的时机，可以由参数sync_bin log控制，默认是0。为0的时候，表示每次提交事务都只write，由系统自行判断什么时候执行fsync。

![img](https://oss.javaguide.cn/github/javaguide/05-20220305234754405.png)

虽然性能得到提升，但是机器宕机，page cache里面的bin log会丢失。为了安全起见，可以设置为1，表示每次提交事务都会执行fsync，就如同**redo log日志刷盘流程**一样。最后还有一种折中方式，可以设置为N(N>1)，表示每次提交事务都write，但累积N个事务后才fsync。

![img](https://oss.javaguide.cn/github/javaguide/06-20220305234801592.png)

在出现IO瓶颈的场景里，将sync_bin log设置成一个比较大的值，可以提升性能。同样的，如果机器宕机，会丢失最近N个事务的bin log日志。

### 两阶段提交

redo log（重做日志）让InnoDB存储引擎拥有了崩溃恢复能力。

bin log（归档日志）保证了MySQL集群架构的数据一致性。

虽然它们都属于持久化的保证，但是侧重点不同。在执行更新语句过程，会记录redo log与bin log两块日志，以基本的事务为单位，redo log在事务执行过程中可以不断写入，而bin log只有在提交事务时才写入，所以redo log与bin log的写入时机不一样。

![img](https://oss.javaguide.cn/github/javaguide/01-20220305234816065.png)

回到正题，redo log与bin log两份日志之间的逻辑不一致，会出现什么问题？我们以update语句为例，假设id=2的记录，字段c值是0，把字段c值更新成1，SQL语句为`update T set c=1 where id=2`。假设执行过程中写完redo log日志后，bin log日志写期间发生了异常，会出现什么情况呢？

![img](https://oss.javaguide.cn/github/javaguide/02-20220305234828662.png)

由于bin log没写完就异常，这时候bin log里面没有对应的修改记录。因此，之后用bin log日志恢复数据时，就会少这一次更新，恢复出来的这一行c值是0，而原库因为redo log日志恢复，这一行c值是1，最终数据不一致。

![img](https://oss.javaguide.cn/github/javaguide/03-20220305235104445.png)

为了解决两份日志之间的逻辑一致问题，InnoDB存储引擎使用**两阶段提交**方案。原理很简单，将redo log的写入拆成了两个步骤prepare和commit，这就是**两阶段提交**。

![img](https://oss.javaguide.cn/github/javaguide/04-20220305234956774.png)

使用**两阶段提交**后，写入bin log时发生异常也不会有影响，因为MySQL根据redo log日志恢复数据时，发现redo log还处于prepare阶段，并且没有对应bin log日志，就会回滚该事务。

![img](https://oss.javaguide.cn/github/javaguide/05-20220305234937243.png)

再看一个场景，redo log设置commit阶段发生异常，那会不会回滚事务呢？

![img](https://oss.javaguide.cn/github/javaguide/06-20220305234907651.png)

并不会回滚事务，它会执行上图框住的逻辑，虽然redo log是处于prepare阶段，但是能通过事务id找到对应的bin log日志，所以MySQL认为是完整的，就会提交事务恢复数据。

### undo log

> 我们知道如果想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行回滚，在MySQL中，恢复机制是通过回滚日志（undo log）实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后再执行相关的操作。如果执行过程中遇到异常的话，我们直接利用回滚日志中的信息将数据回滚到修改之前的样子即可！并且，回滚日志会先于数据持久化到磁盘上。这样就保证了即使遇到数据库突然宕机等情况，当用户再次启动数据库的时候，数据库还能够通过查询回滚日志来回滚将之前未完成的事务。

另外，MVCC的实现依赖于：**隐藏字段、ReadView、undolog**。在内部实现中，InnoDB通过数据行的DB_TRX_ID和ReadView来判断数据的可见性，如不可见，则通过数据行的DB_ROLL_PTR找到undo log中的历史版本。每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建ReadView之前已经提交的修改和该事务本身做的修改

### 总结三大日志

> MySQLInnoDB引擎使用redo log(重做日志)保证事务的持久性，使用undolog(回滚日志)来保证事务的原子性。
> MySQL数据库的数据备份、主备、主主、主从都离不开bin log，需要依靠bin log来同步数据，保证数据一致性。
> [原文链接](https://javaguide.cn/database/mysql/mysql-logs.html)

bin log,redo log,undo log

**bin log**
主要使用场景有两个，分别是主从复制和数据恢复
bin log记录了数据库表结构和表数据变更，比如update/delete/insert/truncate/create。它不会记录select（因为这没有对表没有进行变更）

**redo log**
包括两部分：一个是内存中的日志缓冲(redo log buffer)，另一个是磁盘上的日志文件(redo log file)。Mysql的基本存储结构是页(记录都存在页里边)，所以MySQL是先把这条记录所在的页找到，然后把该页加载到内存中，将对应记录进行修改。现在就可能存在一个问题：如果在内存中把数据改了，还没来得及落磁盘，而此时的数据库挂了怎么办？显然这次更改就丢了。所以引入redo log解决。MySQL每执行一条DML语句，先将记录写入redo log buffer（缓冲），后续某个时间点再一次性将多个操作记录写到redo log file（磁盘）。这种先写日志，再写磁盘的技术就是MySQL里经常说到的WAL(Write-Ahead Logging)技术。redo log也是写到磁盘，不过是顺序IO所以速度很快。

**bin log和 redo log区别**
1. 存储的内容
bin log记载的是update/delete/insert这样的SQL语句，而redo log记载的是物理修改的内容（xxxx页修改了xxx）。
所以在搜索资料的时候会有这样的说法：redo log记录的是数据的物理变化，bin log记录的是数据的逻辑变化
2. 功能
redo log的作用是为持久化而生的。写完内存，如果数据库挂了，那我们可以通过redo log来恢复内存还没来得及刷到磁盘的数据，将redo log加载到内存里边，那内存就能恢复到挂掉之前的数据了。
bin log的作用是复制和恢复而生的。主从服务器需要保持数据的一致性，通过bin log来同步数据。如果整个数据库的数据都被删除了，bin log存储着所有的数据变更情况，那么可以通过bin log来对数据进行恢复。看到这里，你会想：”如果整个数据库的数据都被删除了，那我可以用redo log的记录来恢复吗？“不能,因为功能的不同，redo log存储的是物理数据的变更，如果我们内存的数据已经刷到了磁盘了，那redo log的数据就无效了。所以redo log不会存储着历史所有数据的变更，文件的内容会被覆盖的。
3. redo log是MySQL的InnoDB引擎所产生的。bin log无论MySQL用什么引擎，都会有的。

**undo log**
主要有两个作用：回滚和多版本控制(MVCC)
在数据修改的时候，不仅记录了redo log，还记录undo log，如果因为某些原因导致事务失败或回滚了，可以用undo log进行回滚，undo log主要存储的也是逻辑日志，比如我们要insert一条数据了，那undo log会记录的一条对应的delete日志。我们要update一条记录时，它会记录一条对应相反的update记录。

### 相关文章

| [MySQL undo log、bin log、redo log](https://mp.weixin.qq.com/s/0z6GmUp0Lb1hDUo0EyYiUg) | [必须了解的mysql三大日志-bin log、redo log和undo log](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494134&idx=1&sn=241d91929e6a8b7b5f92db1528286645&source=41#wechat_redirect) | [MySQL日志(redo log 和 undo log)都是什么鬼？](https://mp.weixin.qq.com/s/JLV8GEcF1z4D8oiDhFo6-g) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL不会丢失数据的秘密，就藏在它的7种日志里](https://mp.weixin.qq.com/s/d_CRZGbC6qpxdP4BYetFcA) | [Java代码中，如何监控Mysql的bin log？](https://mp.weixin.qq.com/s/IEG6O3xNv62moh5DYxPL2Q) | [手把手教你玩MySQL删库不跑路，直接把MySQL的bin log玩溜！](https://mp.weixin.qq.com/s/w4vPFHJkog2nbl68_0LOnw) |
| [MySQL的bin log的三种格式这么好玩！](https://mp.weixin.qq.com/s/FlhZsx9MBGY0ypXQMqe6wA) | [讲一讲MySQL数据备份杀手锏bin log](https://mp.weixin.qq.com/s/0dBwL3nwcldbS7hv7O5rOQ) | [3000帧动画图解MySQL为什么需要bin log、redo log和undo log](https://mp.weixin.qq.com/s/lvw89Ix73oTqc2juF4X5sg) |
| [MySQL bin log的三个业务应用场景](https://mp.weixin.qq.com/s/kqfMxkpYZ3ICkHfbv8ET1g) | [MySQL中bin log与redo log的区别](https://zhuanlan.zhihu.com/p/299512401) | [听我讲完redo log、bin log原理，面试官老脸一红](https://mp.weixin.qq.com/s/PvQxdDMdC98mpwSb7pyXvw) |
| [bin log恢复误删除的数据](https://mp.weixin.qq.com/s/JYuh2c3Tad8C_DP7LZO5bg) | [聊聊redo log是什么？](https://mp.weixin.qq.com/s/ayI190ZxhAoUracU3K_HTA) | [MySQL为什么需要redo log？](https://mp.weixin.qq.com/s/cXQ3RFwi-4JgH_x-3HJQnA) |


## InnoDB存储引擎对MVCC的实现

### 一致性非锁定读和锁定读

#### 一致性非锁定读

对于[一致性非锁定读（Consistent Nonlocking Reads）](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)的实现，通常做法是加一个版本号或者时间戳字段，在更新数据的同时版本号+1或者更新时间戳。查询时，将当前可见的版本号与对应记录的版本号进行比对，如果记录的版本小于可见版本，则表示该记录可见。在InnoDB存储引擎中，[多版本控制(multi versioning)](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html)就是对非锁定读的实现。如果读取的行正在执行DELETE或UPDATE操作，这时读取操作不会去等待行上锁的释放。相反地，InnoDB存储引擎会去读取行的一个快照数据，对于这种读取历史数据的方式，我们叫它快照读(snapshot read)。在`Repeatable Read`和`Read Committed`两个隔离级别下，如果是执行普通的select语句（不包括select...lock in share mode,select...for update）则会使用一致性非锁定读（MVCC）。并且在Repeatable Read下MVCC实现了可重复读和防止部分幻读

#### 锁定读

如果执行的是下列语句，就是[锁定读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)

- select ... lock in share mode
- select ... for update
- insert、update、delete操作

在锁定读下，读取的是数据的最新版本，这种读也被称为当前读（current read）。锁定读会对读取到的记录加锁：

- select ... lock in share mode：对记录加S锁，其它事务也可以加S锁，如果加x锁则会被阻塞
- select ... for update、insert、update、delete：对记录加X锁，且其它事务不能加任何锁

在一致性非锁定读下，即使读取的记录已被其它事务加上X锁，这时记录也是可以被读取的，即读取的快照数据。上面说了，在Repeatable Read下MVCC防止了部分幻读，这边的“部分”是指在一致性非锁定读情况下，只能读取到第一次查询之前所插入的数据（根据Read View判断数据可见性，Read View在第一次查询时生成）。但是！如果是当前读，每次读取的都是最新数据，这时如果两次查询中间有其它事务插入数据，就会产生幻读。所以，InnoDB在实现Repeatable Read时，如果执行的是当前读，则会对读取的记录使用Next-key Lock，来防止其它事务在间隙间插入数据

### InnoDB对MVCC的实现

MVCC的实现依赖于：**隐藏字段、Read View、undo log**。在内部实现中，InnoDB通过数据行的DB_TRX_ID和Read View来判断数据的可见性，如不可见，则通过数据行的DB_ROLL_PTR找到undo log中的历史版本。每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建Read View之前已经提交的修改和该事务本身做的修改

#### 隐藏字段

在内部，InnoDB存储引擎为每行数据添加了三个[隐藏字段](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html)：

- DB_TRX_ID（6字节）：表示最后一次插入或更新该行的事务id。此外，delete操作在内部被视为更新，只不过会在记录头Record header中的deleted_flag字段将其标记为已删除
- DB_ROLL_PTR（7字节）回滚指针，指向该行的undo log。如果该行未被更新，则为空
- DB_ROW_ID（6字节）：如果没有设置主键且该表没有唯一非空索引时，InnoDB会使用该id来生成聚簇索引

#### ReadView

```c
class ReadView {
  /* ... */
private:
  trx_id_t m_low_limit_id;      /* 大于等于这个ID的事务均不可见 */

  trx_id_t m_up_limit_id;       /* 小于这个ID的事务均可见 */

  trx_id_t m_creator_trx_id;    /* 创建该Read View的事务ID */

  trx_id_t m_low_limit_no;      /* 事务Number,小于该Number的Undo Logs均可以被Purge */

  ids_t m_ids;                  /* 创建Read View时的活跃事务列表 */

  m_closed;                     /* 标记Read View是否close */
}
```

[Read View](https://github.com/facebook/mysql-8.0/blob/8.0/storage/innobase/include/read0types.h#L298)主要是用来做可见性判断，里面保存了“当前对本事务不可见的其他活跃事务”

主要有以下字段：

- m_low_limit_id：目前出现过的最大的事务ID+1，即下一个将被分配的事务ID。大于等于这个ID的数据版本均不可见
- m_up_limit_id：活跃事务列表m_ids中最小的事务ID，如果m_ids为空，则m_up_limit_id为m_low_limit_id。小于这个ID的数据版本均可见
- m_ids：Read View创建时其他未提交的活跃事务ID列表。创建Read View时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。m_ids不包括当前事务自己和已提交的事务（正在内存中）
- m_creator_trx_id：创建该Read View的事务ID

#### undo-log

undo log主要有两个作用：

- 当事务回滚时用于将数据恢复到修改前的样子
- 另一个作用是MVCC，当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，则可以通过undo log读取之前的版本数据，以此实现非锁定读

在InnoDB存储引擎中undo log分为两种：insert undo log和update undo log：

1. **insert undo log**：指在insert操作中产生的undo log。因为insert操作的记录只对事务本身可见，对其他事务不可见，故该undo log可以在事务提交后直接删除。不需要进行purge操作。insert时的数据初始状态：

2. **update undo log**：update或delete操作中产生的undo log。该undo log可能需要提供MVCC机制，因此不能在事务提交时就进行删除。提交时放入undo log链表，等待purge线程进行最后的删除

不同事务或者相同事务的对同一记录行的修改，会使该记录行的undo log成为一条链表，链首就是最新的记录，链尾就是最早的旧记录。

#### 数据可见性算法

在InnoDB存储引擎中，创建一个新事务后，执行每个select语句前，都会创建一个快照（ReadView），**快照中保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号**。其实简单的说保存的是系统中当前不应该被本事务看到的其他事务ID列表（即m_ids）。当用户在这个事务中要读取某个记录行的时候，InnoDB会将该记录行的DB_TRX_ID与Read View中的一些变量及当前事务ID进行比较，判断是否满足可见性条件。

1. 如果记录DB_TRX_ID< m_up_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之前就提交了，所以该记录行的值对当前事务是可见的
2. 如果DB_TRX_ID>=m_low_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之后才修改该行，所以该记录行的值对当前事务不可见。跳到步骤5
3. m_ids为空，则表明在当前事务创建快照之前，修改该行的事务就已经提交了，所以该记录行的值对当前事务是可见的
4. 如果m_up_limit_id<=DB_TRX_ID< m_low_limit_id，表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照的时候可能处于“活动状态”或者“已提交状态”；所以就要对活跃事务列表m_ids进行查找（源码中是用的二分查找，因为是有序的）
    - 如果在活跃事务列表m_ids中能找到DB_TRX_ID，表明：①在当前事务创建快照前，该记录行的值被事务ID为DB_TRX_ID的事务修改了，但没有提交；或者②在当前事务创建快照后，该记录行的值被事务ID为DB_TRX_ID的事务修改了。这些情况下，这个记录行的值对当前事务都是不可见的。跳到步骤5
    - 在活跃事务列表中找不到，则表明“id为trx_id的事务”在修改“该记录行的值”后，在“当前事务”创建快照前就已经提交了，所以记录行对当前事务可见
5. 在该记录行的DB_ROLL_PTR指针所指向的undolog取出快照记录，用快照记录的DB_TRX_ID跳到步骤1重新开始判断，直到找到满足的快照版本或返回空

### RC和RR隔离级别下MVCC的差异

在事务隔离级别RC和RR（InnoDB存储引擎的默认事务隔离级别）下，InnoDB存储引擎使用MVCC（非锁定一致性读），但它们生成Read View的时机却不同

- 在RC隔离级别下的每次select查询前都生成一个Read View(m_ids列表)
- 在RR隔离级别下只在事务开始后第一次select数据前生成一个Read View（m_ids列表）

### MVCC解决不可重复读问题

虽然RC和RR都通过MVCC来读取快照数据，但由于生成Read View时机不同，从而在RR级别下实现可重复读。

#### 在RC下Read View生成情况

**1.假设时间线来到T4，那么此时数据行id=1的版本链为**：

![img](https://javaguide.cn/assets/a3fd1ec6-8f37-42fa-b090-7446d488fd04.bf41f07c.png)

由于RC级别下每次查询都会生成Read View，并且事务101、102并未提交，此时103事务生成的Read View中活跃的事务m_ids为：[101,102]，m_low_limit_id为：104，m_up_limit_id为：101，m_creator_trx_id为：103

- 此时最新记录的DB_TRX_ID为101，m_up_limit_id<=101< m_low_limit_id，所以要在m_ids列表中查找，发现DB_TRX_ID存在列表中，那么这个记录不可见
- 根据DB_ROLL_PTR找到undo log中的上一版本记录，上一条记录的DB_TRX_ID还是101，不可见
- 继续找上一条DB_TRX_ID为1，满足1< m_up_limit_id，可见，所以事务103查询到数据为name=菜花

**2.时间线来到T6，数据的版本链为**：

![img](https://javaguide.cn/assets/528559e9-dae8-4d14-b78d-a5b657c88391.2ff79120.png)

因为在RC级别下，重新生成Read View，这时事务101已经提交，102并未提交，所以此时Read View中活跃的事务m_ids：[102]，m_low_limit_id为：104，m_up_limit_id为：102，m_creator_trx_id为：103

- 此时最新记录的DB_TRX_ID为102，m_up_limit_id<=102< m_low_limit_id，所以要在m_ids列表中查找，发现DB_TRX_ID存在列表中，那么这个记录不可见
- 根据DB_ROLL_PTR找到undo log中的上一版本记录，上一条记录的DB_TRX_ID为101，满足101< m_up_limit_id，记录可见，所以在T6时间点查询到数据为name=李四，与时间T4查询到的结果不一致，不可重复读！

**3.时间线来到T9，数据的版本链为**：

![img](https://javaguide.cn/assets/6f82703c-36a1-4458-90fe-d7f4edbac71a.c8de5ed7.png)

重新生成ReadView，这时事务101和102都已经提交，所以m_ids为空，则m_up_limit_id=m_low_limit_id=104，最新版本事务ID为102，满足102< m_low_limit_id，可见，查询结果为name=赵六

> 总结：在RC隔离级别下，事务在每次查询开始时都会生成并设置新的Read View，所以导致不可重复读

#### 在RR下Read View生成情况

在可重复读级别下，只会在事务开始后第一次读取数据时生成一个Read View（m_ids列表）

**1.在T4情况下的版本链为**：

![img](https://javaguide.cn/assets/0e906b95-c916-4f30-beda-9cb3e49746bf.3a363d10.png)

在当前执行select语句时生成一个Read View，此时m_ids：[101,102]，m_low_limit_id为：104，m_up_limit_id为：101，m_creator_trx_id为：103

此时和RC级别下一样：

- 最新记录的DB_TRX_ID为101，m_up_limit_id<=101< m_low_limit_id，所以要在m_ids列表中查找，发现DB_TRX_ID存在列表中，那么这个记录不可见
- 根据DB_ROLL_PTR找到undolog中的上一版本记录，上一条记录的DB_TRX_ID还是101，不可见
- 继续找上一条DB_TRX_ID为1，满足1< m_up_limit_id，可见，所以事务103查询到数据为name=菜花

**2.时间点T6情况下**：

![img](https://javaguide.cn/assets/79ed6142-7664-4e0b-9023-cf546586aa39.9c5cd303.png)

在RR级别下只会生成一次Read View，所以此时依然沿用m_ids：[101,102]，m_low_limit_id为：104，m_up_limit_id为：101，m_creator_trx_id为：103

- 最新记录的DB_TRX_ID为102，m_up_limit_id<=102< m_low_limit_id，所以要在m_ids列表中查找，发现DB_TRX_ID存在列表中，那么这个记录不可见
- 根据DB_ROLL_PTR找到undo log中的上一版本记录，上一条记录的DB_TRX_ID为101，不可见
- 继续根据DB_ROLL_PTR找到undo log中的上一版本记录，上一条记录的DB_TRX_ID还是101，不可见
- 继续找上一条DB_TRX_ID为1，满足1< m_up_limit_id，可见，所以事务103查询到数据为name=菜花

**3.时间点T9情况下**：

![img](https://javaguide.cn/assets/cbbedbc5-0e3c-4711-aafd-7f3d68a4ed4e.7b4a86c0.png)

此时情况跟T6完全一样，由于已经生成了Read View，此时依然沿用m_ids：[101,102]，所以查询结果依然是name=菜花

### MVCC➕Next-key-Lock防止幻读

InnoDB存储引擎在RR级别下通过MVCC和Next-keyLock来解决幻读问题：

**1、执行普通select，此时会以MVCC快照读的方式读取数据**

在快照读的情况下，RR隔离级别只会在事务开启后的第一次查询生成Read View，并使用至事务提交。所以在生成Read View之后其它事务所做的更新、插入记录版本对当前事务并不可见，实现了可重复读和防止快照读下的“幻读”

**2、执行select...for update/lock in share mode、insert、update、delete等当前读**

在当前读下，读取的都是最新的数据，如果其它事务有插入新的记录，并且刚好在当前事务查询范围内，就会产生幻读！InnoDB使用[Next-keyLock](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-next-key-locks)来防止这种情况。当执行当前读时，会锁定读取到的记录的同时，锁定它们的间隙，防止其它事务在查询范围内插入数据。只要我不让你插入，就不会发生幻读

> [原文链接](https://javaguide.cn/database/mysql/innodb-implementation-of-mvcc.html)
> [面试中的MVCC到底是什么意思？](https://mp.weixin.qq.com/s/wE43jx9svrHg5U16sq8CYw)

## SQL语句在MySQL中的执行过程

### MySQL基础架构分析

#### MySQL基本架构概览

下图是MySQL的一个简要架构图，从下图你可以很清晰的看到用户的SQL语句在MySQL内部是如何执行的。

先简单介绍一下下图涉及的一些组件的基本作用帮助大家理解这幅图，在下节中会详细介绍到这些组件的作用。

- **连接器**:身份认证和权限相关(登录MySQL的时候)。
- **查询缓存**:执行查询语句的时候，会先查询缓存（MySQL8.0版本后移除，因为这个功能不太实用）。
- **分析器**:没有命中缓存的话，SQL语句就会经过分析器，分析器说白了就是要先看你的SQL语句要干嘛，再检查你的SQL语句语法是否正确。
- **优化器**:按照MySQL认为最优的方案去执行。
- **执行器**:执行语句，然后从存储引擎返回数据。

![img](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

简单来说MySQL主要分为Server层和存储引擎层：

- **Server层**：主要包括连接器、查询缓存、分析器、优化器、执行器等，所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图，函数等，还有一个通用的日志模块bin log日志模块。
- **存储引擎**：主要负责数据的存储和读取，采用可以替换的插件式架构，支持InnoDB、MyISAM、Memory等多个存储引擎，其中InnoDB引擎有自有的日志模块redo log模块。现在最常用的存储引擎是InnoDB，它从MySQL5.5版本开始就被当做默认存储引擎了。

#### Server层基本组件介绍

##### 连接器

连接器主要和身份认证和权限相关的功能相关，就好比一个级别很高的门卫一样。

主要负责用户登录数据库，进行用户的身份认证，包括校验账户密码，权限等操作，如果用户账户密码已通过，连接器会到权限表中查询该用户的所有权限，之后在这个连接里的权限逻辑判断都是会依赖此时读取到的权限数据，也就是说，后续只要这个连接不断开，即使管理员修改了该用户的权限，该用户也是不受影响的。

##### 查询缓存(MySQL8.0版本后移除)

查询缓存主要用来缓存我们所执行的SELECT语句以及该语句的结果集。

连接建立后，执行查询语句的时候，会先查询缓存，MySQL会先校验这个SQL是否执行过，以Key-Value的形式缓存在内存中，Key是查询预计，Value是结果集。如果缓存key被命中，就会直接返回给客户端，如果没有命中，就会执行后续的操作，完成后也会把结果缓存起来，方便下一次调用。当然在真正执行缓存查询的时候还是会校验用户的权限，是否有该表的查询条件。

MySQL查询不建议使用缓存，因为查询缓存失效在实际业务场景中可能会非常频繁，假如你对一个表更新的话，这个表上的所有的查询缓存都会被清空。对于不经常更新的数据来说，使用缓存还是可以的。

所以，一般在大多数情况下我们都是不推荐去使用查询缓存的。

MySQL8.0版本后删除了缓存的功能，官方也是认为该功能在实际的应用场景比较少，所以干脆直接删掉了。

##### 分析器

MySQL没有命中缓存，那么就会进入分析器，分析器主要是用来分析SQL语句是来干嘛的，分析器也会分为几步：

**第一步，词法分析**，一条SQL语句有多个字符串组成，首先要提取关键字，比如select，提出查询的表，提出字段名，提出查询条件等等。做完这些操作后，就会进入第二步。

**第二步，语法分析**，主要就是判断你输入的SQL是否正确，是否符合MySQL的语法。

完成这2步之后，MySQL就准备开始执行了，但是如何执行，怎么执行是最好的结果呢？这个时候就需要优化器上场了。

##### 优化器

优化器的作用就是它认为的最优的执行方案去执行（有时候可能也不是最优，这篇文章涉及对这部分知识的深入讲解），比如多个索引的时候该如何选择索引，多表查询的时候如何选择关联顺序等。

可以说，经过了优化器之后可以说这个语句具体该如何执行就已经定下来。

##### 执行器

当选择了执行方案后，MySQL就准备开始执行了，首先执行前会校验该用户有没有权限，如果没有权限，就会返回错误信息，如果有权限，就会去调用引擎的接口，返回接口执行的结果。

### 二语句分析

#### 查询语句

说了以上这么多，那么究竟一条SQL语句是如何执行的呢？其实我们的SQL可以分为两种，一种是查询，一种是更新（增加，修改，删除）。我们先分析下查询语句，语句如下：


```sql
select * from tb_student A where A.age='18' and A.name='张三';
```

结合上面的说明，我们分析下这个语句的执行流程：

- 先检查该语句是否有权限，如果没有权限，直接返回错误信息，如果有权限，在MySQL8.0版本以前，会先查询缓存，以这条SQL语句为key在内存中查询是否有结果，如果有直接缓存，如果没有，执行下一步。

- 通过分析器进行词法分析，提取SQL语句的关键元素，比如提取上面这个语句是查询select，提取需要查询的表名为tb_student，需要查询所有的列，查询条件是这个表的id='1'。然后判断这个SQL语句是否有语法错误，比如关键词是否正确等等，如果检查没问题就执行下一步。

- 接下来就是优化器进行确定执行方案，上面的SQL语句，可以有两种执行方案：

```
a. 先查询学生表中姓名为“张三”的学生，然后判断是否年龄是18。
b. 先找出学生中年龄18岁的学生，然后再查询姓名为“张三”的学生。
```

那么优化器根据自己的优化算法进行选择执行效率最好的一个方案（优化器认为，有时候不一定最好）。那么确认了执行计划后就准备开始执行了。

- 进行权限校验，如果没有权限就会返回错误信息，如果有权限就会调用数据库引擎接口，返回引擎的执行结果。

#### 更新语句

以上就是一条查询SQL的执行流程，那么接下来我们看看一条更新语句如何执行的呢？SQL语句如下：

```sql
update tb_student A set A.age='19' where A.name='张三';
```

我们来给张三修改下年龄，在实际数据库肯定不会设置年龄这个字段的，不然要被技术负责人打的。其实这条语句也基本上会沿着上一个查询的流程走，只不过执行更新的时候肯定要记录日志啦，这就会引入日志模块了，MySQL自带的日志模块是**bin log（归档日志）**，所有的存储引擎都可以使用，我们常用的InnoDB引擎还自带了一个日志模块**redo log（重做日志）**，我们就以InnoDB模式下来探讨这个语句的执行流程。流程如下：

- 先查询到张三这一条数据，如果有缓存，也是会用到缓存。
- 然后拿到查询的语句，把age改为19，然后调用引擎API接口，写入这一行数据，InnoDB引擎把数据保存在内存中，同时记录redo log，此时redo log进入prepare状态，然后告诉执行器，执行完成了，随时可以提交。
- 执行器收到通知后记录bin log，然后调用引擎接口，提交redo log为提交状态。
- 更新完成。

**这里肯定有同学会问，为什么要用两个日志模块，用一个日志模块不行吗**？

这是因为最开始MySQL并没有InnoDB引擎（InnoDB引擎是其他公司以插件形式插入MySQL的），MySQL自带的引擎是MyISAM，但是我们知道redo log是InnoDB引擎特有的，其他存储引擎都没有，这就导致会没有crash-safe的能力(crash-safe的能力即使数据库发生异常重启，之前提交的记录都不会丢失)，bin log日志只能用来归档。

并不是说只用一个日志模块不可以，只是InnoDB引擎就是通过redo log来支持事务的。那么，又会有同学问，我用两个日志模块，但是不要这么复杂行不行，为什么redo log要引入prepare预提交状态？这里我们用反证法来说明下为什么要这么做？

- **先写redo log直接提交，然后写bin log**，假设写完redo log后，机器挂了，bin log日志没有被写入，那么机器重启后，这台机器会通过redo log恢复数据，但是这个时候bin log并没有记录该数据，后续进行机器备份的时候，就会丢失这一条数据，同时主从同步也会丢失这一条数据。
- **先写binlog，然后写redo log**，假设写完了bin log，机器异常重启了，由于没有redo log，本机是无法恢复这一条记录的，但是bin log又有记录，那么和上面同样的道理，就会产生数据不一致的情况。

如果采用redo log两阶段提交的方式就不一样了，写完bin log后，然后再提交redo log就会防止出现上述的问题，从而保证了数据的一致性。那么问题来了，有没有一个极端的情况呢？假设redo log处于预提交状态，bin log也已经写完了，这个时候发生了异常重启会怎么样呢？这个就要依赖于MySQL的处理机制了，MySQL的处理过程如下：

- 判断redo log是否完整，如果判断是完整的，就立即提交。
- 如果redo log只是预提交但不是commit状态，这个时候就会去判断bin log是否完整，如果完整就提交redo log,不完整就回滚事务。

这样就解决了数据一致性的问题。

### 总结

- MySQL主要分为Server层和引擎层，Server层主要包括连接器、查询缓存、分析器、优化器、执行器，同时还有一个日志模块（bin log），这个日志模块所有执行引擎都可以共用，redo log只有InnoDB有。
- 引擎层是插件式的，目前主要包括，MyISAM,InnoDB,Memory等。
- 查询语句的执行流程如下：权限校验（如果命中缓存）--->查询缓存--->分析器--->优化器--->权限校验--->执行器--->引擎
- 更新语句执行流程如下：分析器---->权限校验---->执行器--->引擎---redo log(prepare状态)--->bin log--->redo log(commit状态)

> [原文链接](https://javaguide.cn/database/mysql/how-sql-executed-in-mysql.html)

### sql执行顺序
```
from、on、join、where、group by、(avg,sum)、having、select、distinct、union、order by、limit
```
### 相关文章

| [面试官：你来讲讲一条查询语句的具体执行过程](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247490897&idx=1&sn=e5eaa58d426255c2c1401578d5606138&source=41#wechat_redirect) | [一文读懂MySQL查询语句的执行过程](https://mp.weixin.qq.com/s/dLJet2geb-dQ9hLE0oL6Qw) | [MySQL的执行过程及执行顺序](https://mp.weixin.qq.com/s/DS4-ObUFeGsU2l6FvF0Pcw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [图解SQL执行顺序，通俗易懂！](https://mp.weixin.qq.com/s/k1Zjr6ucFZzI-w6wC2Hz3Q) |                                                              |                                                              |

## MySQL查询缓存详解

缓存是一个有效且实用的系统性能优化的手段，不论是操作系统还是各种软件和网站或多或少都用到了缓存。然而，有经验的DBA都建议生产环境中把MySQL自带的QueryCache（查询缓存）给关掉。而且，从MySQL5.7.20开始，就已经默认弃用查询缓存了。在MySQL8.0及之后，更是直接删除了查询缓存的功能。这又是为什么呢？查询缓存真就这么鸡肋么?带着如下几个问题，我们正式进入本文。

- MySQL查询缓存是什么？适用范围？
- MySQL缓存规则是什么？
- MySQL缓存的优缺点是什么？
- MySQL缓存对性能有什么影响？

### MySQL查询缓存介绍

MySQL体系架构如下图所示：

![img](https://oss.javaguide.cn/github/javaguide/mysql/mysql-architecture.png)

为了提高完全相同的查询语句的响应速度，MySQL Server会对查询语句进行Hash计算得到一个Hash值。MySQL Server不会对SQL做任何处理，SQL必须完全一致Hash值才会一样。得到Hash值之后，通过该Hash值到查询缓存中匹配该查询的结果。

- 如果匹配（命中），则将查询的结果集直接返回给客户端，不必再解析、执行查询。
- 如果没有匹配（未命中），则将Hash值和结果集保存在查询缓存中，以便以后使用。

也就是说，**一个查询语句（select）到了MySQLServer之后，会先到查询缓存看看，如果曾经执行过的话，就直接返回结果集给客户端**。

![img](https://oss.javaguide.cn/javaguide/13526879-3037b144ed09eb88.png)

### MySQL查询缓存管理和配置

通过`showvariableslike'%query_cache%'`命令可以查看查询缓存相关的信息。

8.0版本之前的话，打印的信息可能是下面这样的：

```bash
mysql> show variables like '%query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| have_query_cache             | YES     |
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 599040  |
| query_cache_type             | ON      |
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
6 rows in set (0.02 sec)
```

8.0以及之后版本之后，打印的信息是下面这样的：


```bash
mysql> show variables like '%query_cache%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| have_query_cache | NO    |
+------------------+-------+
1 row in set (0.01 sec)
```

我们这里对8.0版本之前show variables like '%query_cache%';命令打印出来的信息进行解释。

- **have_query_cache**:该MySQL Server是否支持查询缓存，如果是YES表示支持，否则则是不支持。
- **query_cache_limit**:MySQL查询缓存的最大查询结果，查询结果大于该值时不会被缓存。
- **query_cache_min_res_unit**:查询缓存分配的最小块的大小(字节)。当查询进行的时候，MySQL把查询结果保存在查询缓存中，但如果要保存的结果比较大，超过query_cache_min_res_unit的值，这时候MySQL将一边检索结果，一边进行保存结果，也就是说，有可能在一次查询中，MySQL要进行多次内存分配的操作。适当的调节query_cache_min_res_unit可以优化内存。
- **query_cache_size**:为缓存查询结果分配的内存的数量，单位是字节，且数值必须是1024的整数倍。默认值是0，即禁用查询缓存。
- **query_cache_type**:设置查询缓存类型，默认为ON。设置GLOBAL值可以设置后面的所有客户端连接的类型。客户端可以设置SESSION值以影响他们自己对查询缓存的使用。
- **query_cache_wlock_invalidate**:如果某个表被锁住，是否返回缓存中的数据，默认关闭，也是建议的。

query_cache_type可能的值(修改query_cache_type需要重启MySQLServer)：

- 0或OFF：关闭查询功能。
- 1或ON：开启查询缓存功能，但不缓存SelectSQL_NO_CACHE开头的查询。
- 2或DEMAND：开启查询缓存功能，但仅缓存SelectSQL_CACHE开头的查询。

**建议**：

- query_cache_size不建议设置的过大。过大的空间不但挤占实例其他内存结构的空间，而且会增加在缓存中搜索的开销。建议根据实例规格，初始值设置为10MB到100MB之间的值，而后根据运行使用情况调整。
- 建议通过调整query_cache_size的值来开启、关闭查询缓存，因为修改query_cache_type参数需要重启MySQL Server生效。

8.0版本之前，my.cnf加入以下配置，重启MySQL开启查询缓存

```properties
query_cache_type=1
query_cache_size=600000
```

或者，MySQL执行以下命令也可以开启查询缓存

```properties
set global query_cache_type=1;
set global query_cache_size=600000;
```

手动清理缓存可以使用下面三个SQL：

- flush query cache;：清理查询缓存内存碎片。
- reset query cache;：从查询缓存中移除所有查询。
- flush tables；关闭所有打开的表，同时该操作会清空查询缓存中的内容。

### MySQL缓存机制

**缓存规则**

- 查询缓存会将查询语句和结果集保存到内存（一般是key-value的形式，key是查询语句，value是查询的结果集），下次再查直接从内存中取。
- 缓存的结果是通过sessions共享的，所以一个client查询的缓存结果，另一个client也可以使用。
- SQL必须完全一致才会导致查询缓存命中（大小写、空格、使用的数据库、协议版本、字符集等必须一致）。检查查询缓存时，MySQL Server不会对SQL做任何处理，它精确的使用客户端传来的查询。
- 不缓存查询中的子查询结果集，仅缓存查询最终结果集。
- 不确定的函数将永远不会被缓存,比如now()、curdate()、last_insert_id()、rand()等。
- 不缓存产生告警（Warnings）的查询。
- 太大的结果集不会被缓存(< query_cache_limit)。
- 如果查询中包含任何用户自定义函数、存储函数、用户变量、临时表、MySQL库中的系统表，其查询结果也不会被缓存。
- 缓存建立之后，MySQL的查询缓存系统会跟踪查询中涉及的每张表，如果这些表（数据或结构）发生变化，那么和这张表相关的所有缓存数据都将失效。
- MySQL缓存在分库分表环境下是不起作用的。
- 不缓存使用SQL_NO_CACHE的查询。
- ......

查询缓存SELECT选项示例：


```sql
-- 会缓存
SELECT SQL_CACHE id, name FROM customer;
-- 不会缓存
SELECT SQL_NO_CACHE id, name FROM customer;
```

**缓存机制中的内存管理**

查询缓存是完全存储在内存中的，所以在配置和使用它之前，我们需要先了解它是如何使用内存的。MySQL查询缓存使用内存池技术，自己管理内存释放和分配，而不是通过操作系统。内存池使用的基本单位是变长的block,用来存储类型、大小、数据等信息。一个结果集的缓存通过链表把这些block串起来。block最短长度为`query_cache_min_res_unit`。当服务器启动的时候，会初始化缓存需要的内存，是一个完整的空闲块。当查询结果需要缓存的时候，先从空闲块中申请一个数据块为参数`query_cache_min_res_unit`配置的空间，即使缓存数据很小，申请数据块也是这个，因为查询开始返回结果的时候就分配空间，此时无法预知结果多大。分配内存块需要先锁住空间块，所以操作很慢，MySQL会尽量避免这个操作，选择尽可能小的内存块，如果不够，继续申请，如果存储完时有空余则释放多余的。但是如果并发的操作，余下的需要回收的空间很小，小于`query_cache_min_res_unit`，不能再次被使用，就会产生碎片。

### MySQL查询缓存的优缺点

**优点**

- 查询缓存的查询，发生在MySQL接收到客户端的查询请求、查询权限验证之后和查询SQL解析之前。也就是说，当MySQL接收到客户端的查询SQL之后，仅仅只需要对其进行相应的权限验证之后，就会通过查询缓存来查找结果，甚至都不需要经过Optimizer模块进行执行计划的分析优化，更不需要发生任何存储引擎的交互。
- 由于查询缓存是基于内存的，直接从内存中返回相应的查询结果，因此减少了大量的磁盘I/O和CPU计算，导致效率非常高。

**缺点**

- MySQL会对每条接收到的SELECT类型的查询进行Hash计算，然后查找这个查询的缓存结果是否存在。虽然Hash计算和查找的效率已经足够高了，一条查询语句所带来的开销可以忽略，但一旦涉及到高并发，有成千上万条查询语句时，hash计算和查找所带来的开销就必须重视了。
- 查询缓存的失效问题。如果表的变更比较频繁，则会造成查询缓存的失效率非常高。表的变更不仅仅指表中的数据发生变化，还包括表结构或者索引的任何变化。
- 查询语句不同，但查询结果相同的查询都会被缓存，这样便会造成内存资源的过度消耗。查询语句的字符大小写、空格或者注释的不同，查询缓存都会认为是不同的查询（因为他们的Hash值会不同）。
- 相关系统变量设置不合理会造成大量的内存碎片，这样便会导致查询缓存频繁清理内存。

### MySQL查询缓存对性能的影响

在MySQL Server中打开查询缓存对数据库的读和写都会带来额外的消耗:

- 读查询开始之前必须检查是否命中缓存。
- 如果读查询可以缓存，那么执行完查询操作后，会查询结果和查询语句写入缓存。
- 当向某个表写入数据的时候，必须将这个表所有的缓存设置为失效，如果缓存空间很大，则消耗也会很大，可能使系统僵死一段时间，因为这个操作是靠全局锁操作来保护的。
- 对InnoDB表，当修改一个表时，设置了缓存失效，但是多版本特性会暂时将这修改对其他事务屏蔽，在这个事务提交之前，所有查询都无法使用缓存，直到这个事务被提交，所以长时间的事务，会大大降低查询缓存的命中。

### 总结

MySQL中的查询缓存虽然能够提升数据库的查询性能，但是查询同时也带来了额外的开销，每次查询后都要做一次缓存操作，失效后还要销毁。查询缓存是一个适用较少情况的缓存机制。如果你的应用对数据库的更新很少，那么查询缓存将会作用显著。比较典型的如博客系统，一般博客更新相对较慢，数据表相对稳定不变，这时候查询缓存的作用会比较明显。简单总结一下查询缓存的适用场景：

- 表数据修改不频繁、数据较静态。
- 查询（Select）重复度高。
- 查询结果集小于1MB。

对于一个更新频繁的系统来说，查询缓存缓存的作用是很微小的，在某些情况下开启查询缓存会带来性能的下降。简单总结一下查询缓存不适用的场景：

- 表中的数据、表结构或者索引变动频繁
- 重复的查询很少
- 查询的结果集很大

《高性能MySQL》这样写到：

> 根据我们的经验，在高并发压力环境中查询缓存会导致系统性能的下降，甚至僵死。如果你一定要使用查询缓存，那么不要设置太大内存，而且只有在明确收益的时候才使用（数据库内容修改次数较少）。

确实是这样的！实际项目中，更建议使用本地缓存（比如Caffeine）或者分布式缓存（比如Redis），性能更好，更通用一些
> [原文链接](https://javaguide.cn/database/mysql/mysql-query-cache.html)

## Explain执行计划

### 什么是执行计划？

**执行计划**是指一条SQL语句在经过**MySQL查询优化器**的优化会后，具体的执行方式。执行计划通常用于SQL性能分析、优化等场景。通过EXPLAIN的结果，可以了解到如数据表的查询顺序、数据查询操作的操作类型、哪些索引可以被命中、哪些索引实际会命中、每个数据表有多少行记录被查询等信息。

### 如何获取执行计划？

MySQL为我们提供了EXPLAIN命令，来获取执行计划的相关信息。需要注意的是，EXPLAIN语句并不会真的去执行相关的语句，而是通过查询优化器对语句进行分析，找出最优的查询方案，并显示对应的信息。EXPLAIN执行计划支持SELECT、DELETE、INSERT、REPLACE以及UPDATE语句。我们一般多用于分析SELECT查询语句，使用起来非常简单，语法如下：


```sql
EXPLAIN + SELECT 查询语句；
```

我们简单来看下一条查询语句的执行计划：


```sql
mysql> explain SELECT * FROM dept_emp WHERE emp_no IN (SELECT emp_no FROM dept_emp GROUP BY emp_no HAVING COUNT(emp_no)>1);
+----+-------------+----------+------------+-------+-----------------+---------+---------+------+--------+----------+-------------+
| id | select_type | table    | partitions | type  | possible_keys   | key     | key_len | ref  | rows   | filtered | Extra       |
+----+-------------+----------+------------+-------+-----------------+---------+---------+------+--------+----------+-------------+
|  1 | PRIMARY     | dept_emp | NULL       | ALL   | NULL            | NULL    | NULL    | NULL | 331143 |   100.00 | Using where |
|  2 | SUBQUERY    | dept_emp | NULL       | index | PRIMARY,dept_no | PRIMARY | 16      | NULL | 331143 |   100.00 | Using index |
+----+-------------+----------+------------+-------+-----------------+---------+---------+------+--------+----------+-------------+
```

可以看到，执行计划结果中共有12列，各列代表的含义总结如下表：

| **列名**      | **含义**                                     |
| ------------- | -------------------------------------------- |
| id            | SELECT查询的序列标识符                       |
| select_type   | SELECT关键字对应的查询类型                   |
| table         | 用到的表名                                   |
| partitions    | 匹配的分区，对于未分区的表，值为 NULL        |
| type          | 表的访问方法                                 |
| possible_keys | 可能用到的索引                               |
| key           | 实际用到的索引                               |
| key_len       | 所选索引的长度                               |
| ref           | 当使用索引等值查询时，与索引作比较的列或常量 |
| rows          | 预计要读取的行数                             |
| filtered      | 按表条件过滤后，留存的记录数的百分比         |
| Extra         | 附加信息                                     |

#### 如何分析EXPLAIN结果？

为了分析EXPLAIN语句的执行结果，我们需要搞懂执行计划中的重要字段。

#### id

SELECT标识符，是查询中SELECT的序号，用来标识整个查询中SELELCT语句的顺序。

id如果相同，从上往下依次执行。id不同，id值越大，执行优先级越高，如果行引用其他行的并集结果，则该值可以为NULL。

#### select_type

查询的类型，主要用于区分普通查询、联合查询、子查询等复杂的查询，常见的值有：

- **SIMPLE**：简单查询，不包含UNION或者子查询。
- **PRIMARY**：查询中如果包含子查询或其他部分，外层的SELECT将被标记为PRIMARY。
- **SUBQUERY**：子查询中的第一个SELECT。
- **UNION**：在UNION语句中，UNION之后出现的SELECT。
- **DERIVED**：在FROM中出现的子查询将被标记为DERIVED。
- **UNIONRESULT**：UNION查询的结果。

#### table

查询用到的表名，每行都有对应的表名，表名除了正常的表之外，也可能是以下列出的值：

- **<unionM,N>**:本行引用了id为M和N的行的UNION结果；
- **< derivedN>**:本行引用了id为N的表所产生的的派生表结果。派生表有可能产生自FROM语句中的子查询。
- **< subqueryN>**:本行引用了id为N的表所产生的的物化子查询结果。

#### type（重要）

查询执行的类型，描述了查询是如何执行的。所有值的顺序从最优到最差排序为：system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>index>ALL

常见的几种类型具体含义如下：

- **system**：如果表使用的引擎对于表行数统计是精确的（如：MyISAM），且表中只有一行记录的情况下，访问方法是system，是const的一种特例。
- **const**：表中最多只有一行匹配的记录，一次查询就可以找到，常用于使用主键或唯一索引的所有字段作为查询条件。
- **eq_ref**：当连表查询时，前一张表的行在当前这张表中只有一行与之对应。是除了system与const之外最好的join方式，常用于使用主键或唯一索引的所有字段作为连表条件。
- **ref**：使用普通索引作为查询条件，查询结果可能找到多个符合条件的行。
- **index_merge**：当查询条件使用了多个索引时，表示开启了Index Merge优化，此时执行计划中的key列列出了使用到的索引。
- **range**：对索引列进行范围查询，执行计划中的key列表示哪个索引被使用了。
- **index**：查询遍历了整棵索引树，与ALL类似，只不过扫描的是索引，而索引一般在内存中，速度更快。
- **ALL**：全表扫描。

#### possible_keys

possible_keys列表示MySQL执行查询时可能用到的索引。如果这一列为NULL，则表示没有可能用到的索引；这种情况下，需要检查WHERE语句中所使用的的列，看是否可以通过给这些列中某个或多个添加索引的方法来提高查询性能。

#### key（重要）

key列表示MySQL实际使用到的索引。如果为NULL，则表示未用到索引。

#### key_len

key_len列表示MySQL实际使用的索引的最大长度；当使用到联合索引时，有可能是多个列的长度和。在满足需求的前提下越短越好。如果key列显示NULL，则key_len列也显示NULL。

#### rows

rows列表示根据表统计信息及选用情况，大致估算出找到所需的记录或所需读取的行数，数值越小越好。

#### Extra（重要）

这列包含了MySQL解析查询的额外信息，通过这些信息，可以更准确的理解MySQL到底是如何执行查询的。常见的值如下：

- **Using filesort**：在排序时使用了外部的索引排序，没有用到表内索引进行排序。
- **Using temporary**：MySQL需要创建临时表来存储查询的结果，常见于ORDERBY和GROUPBY。
- **Using index**：表明查询使用了覆盖索引，不用回表，查询效率非常高。
- **Using index condition**：表示查询优化器选择使用了索引条件下推这个特性。
- **Using where**：表明查询使用了WHERE子句进行条件过滤。一般在没有使用到索引的时候会出现。
- **Using join buffer（Block Nested Loop）**：连表查询的方式，表示当被驱动表的没有使用索引的时候，MySQL会先将驱动表读出来放到joinbuffer中，再遍历被驱动表与驱动表进行查询。

这里提醒下，当Extra列包含Using file sort或Using temporary时，MySQL的性能可能会存在问题，需要尽可能避免

> [原文链接](https://javaguide.cn/database/mysql/mysql-query-execution-plan.html)

### 总结

1. id: 是用来顺序标识整个查询中select语句的，在嵌套查询中id越大的语句越先执
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
4. type:访问类型，表示数据库引擎查找表的方式。常见的type类型有：all,index,range,ref,eq_ref,const。
- all :全表扫描，表示sql语句会把表中所有表数据全部读取读取扫描一遍。效率最低，我们应尽量避免
- index :全索引扫描,表示sql语句将会把整颗二级索引树全部读取扫描一遍，因为二级索引树的数据量比全表数据量小，所以效率比all高一些。一般查询语句中查询字段为索引字段，且无where子句时，type会为index,类似全表扫描，只是扫描表的时候按照索引次序进行而不是行。主要优点就是避免了排序,但是开销仍然非常大。如在Extra列看到Using index，说明正在使用覆盖索引，只扫描索引的数据，它比按索引次序全表扫描的开销要小很多
- range :部分索引扫描，当查询为区间查询，且查询字段为索引字段时，这时会根据where条件对索引进行部分扫描,范围扫描，一个有限制的索引扫描。key列显示使用了哪个索引。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时,可以使用range
- unique_subquery: 在使用in查询的情况下会取代eq_ref
- index_merge: 此连接类型表示使用了索引合并优化。在这种情况下，输出行中的key列包含使用的索引列表，key_len包含所用索引的最长key部分列表
- ref :出现于where操作符为‘=’，且where字段为非唯一索引的单表查询或联表查询。通过普通索引查询匹配的很多行时的类型
- ref_or_null: 跟ref类似的效果，不过多一个列不能null的条件
- eq_ref :出现于where操作符为‘=’，且where字段为唯一索引的联表查询。最多只返回一条符合条件的记录，通过使用在两个表有关联字段的时候
- const :出现于where操作符为‘=’，且where字段为唯一索引的单表查询,此时最多只会匹配到一行,当确定最多只会有一行匹配的时候，MySQL优化器会在查询前读取它而且只读取一次，因此非常快。使用主键查询往往就是const级别的，非常高效
- system:const的一种特例，表中只有一行数据
fulltext: 全文索引
**单从type字段考虑效率，const > eq_ref > ref > range > index > all**
5. possible_keys:查询可能用到的索引,MySQL 可能采用的索引，但是并不一定使用
6. key:mysql决定采用的索引来优化查询,真正使用的索引名称
sql语句实际执行时使用的索引列，有时候mysql可能会选择优化效果不是最好的索引，这时，我们可以在select语句中使用force index(INDEXNAME)来强制mysql使用指定索引或使用ignore index(INDEXNAME)强制mysql忽略指定索引
7. key_len: 索引key的长度
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

### 相关文章

| [explain都不懂，还好意思说会SQL调优?](https://mp.weixin.qq.com/s/aTkGwVU5D9u1RAbMHQconw) | [MySQL之Explain输出分析](https://mp.weixin.qq.com/s/NwRVW6Z8AMZ_QClTXH71ow) | [什么是MySQL的执行计划（Explain关键字）？](https://mp.weixin.qq.com/s/E8wJQvldwEAzxK5mEuFhog) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL究竟是怎么执行的(explain)](https://mp.weixin.qq.com/s/kYcrHtE82-sOqNOp_qM4Ig) | [EXPLAIN进行索引分析和优化](https://mp.weixin.qq.com/s/-YrZFxTbutLdEbkfa9aOKQ) |                                                              |

## MySQL运维

### 获取基本配置信息

|           说明           |                   执行语句                    |
| :----------------------: | :-------------------------------------------: |
|    是否区分表名大小写    | SHOW VARIABLES LIKE 'lower_case_table_names'; |
|      MySQL服务端口       |          SHOW VARIABLES LIKE 'port';          |
|   MySQL socket文件路径   |         SHOW VARIABLES LIKE 'socket';         |
|     是否跳过域名解析     |   SHOW VARIABLES LIKE 'skip_name_resolve';    |
|       数据库字符集       |  SHOW VARIABLES LIKE 'character_set_server';  |
|   服务器默认的校对规则   |    SHOW VARIABLES LIKE 'collation_server';    |
|  交互式连接超时时间(秒)  |  SHOW VARIABLES LIKE 'interactive_timeout';   |
| 非交互式连接超时时间(秒) |      SHOW VARIABLES LIKE 'wait_timeout';      |
|       事务隔离级别       |      SHOW VARIABLES LIKE 'tx_isolation';      |
|     数据文件存放位置     |       SHOW VARIABLES LIKE '%datadir%';        |
|   允许的最大数据包大小   |   SHOW VARIABLES LIKE 'max_allowed_packet';   |
|    每个线程堆栈的大小    |      SHOW VARIABLES LIKE 'thread_stack';      |
|      线程缓存的大小      |   SHOW VARIABLES LIKE 'thread_cache_size';    |
|    可以打开的表的数量    |    SHOW VARIABLES LIKE 'table_open_cache';    |
|   内部临时表的最大大小   |     SHOW VARIABLES LIKE 'tmp_table_size';     |
|  内存中HEAP表的最大大小  |  SHOW VARIABLES LIKE 'max_heap_table_size';   |
|  MySQL主进程文件的位置   |        SHOW VARIABLES LIKE 'pid_file';        |
|    错误日志文件的位置    |       SHOW VARIABLES LIKE 'log_error';        |
|   慢查询日志文件的位置   |  SHOW VARIABLES LIKE 'slow_query_log_file';   |
|      慢查询时间阈值      |    SHOW VARIABLES LIKE 'long_query_time';     |

### 获取连接数配置信息

|               说明               |                    执行语句                     |
| :------------------------------: | :---------------------------------------------: |
|            最大连接数            |     SHOW VARIABLES LIKE 'max_connections';      |
| 服务器启动以来已使用的最大连接数 |    SHOW status LIKE 'Max_used_connections';     |
|      达到最大连接数的时间点      |  SHOW STATUS LIKE 'Max_used_connections_time';  |
|         连接失败次数上限         |    SHOW VARIABLES LIKE 'max_connect_errors';    |
| 通过MySQLXProtocol已接受的连接数 | SHOW STATUS LIKE 'Mysqlx_connections_accepted'; |
| 通过MySQLXProtocol已关闭的连接数 |  SHOW STATUS LIKE 'Mysqlx_connections_closed';  |

### 获取Binlog配置信息

|        说明         |                执行语句                 |
| :-----------------: | :-------------------------------------: |
|   Binlog文件列表    |            SHOW BINARY LOGS;            |
|   是否启用Binlog    |     SHOW VARIABLES LIKE 'log_bin';      |
|   Binlog同步频率    |   SHOW VARIABLES LIKE 'sync_binlog';    |
|     Binlog格式      |  SHOW VARIABLES LIKE 'binlog_format';   |
|      文件路径       |     SHOW VARIABLES LIKE 'log-bin';      |
| Binlog索引文件位置  |  SHOW VARIABLES LIKE 'log_bin_index';   |
| 单个Binlog最大大小  | SHOW VARIABLES LIKE 'max_binlog_size';  |
|   Binlog基本名字    | SHOW VARIABLES LIKE 'log_bin_basename'; |
| Binlog文件过期时间  | SHOW VARIABLES LIKE 'expire_logs_days'; |
| MySQL服务器的唯一ID |    SHOW VARIABLES LIKE 'server_id';     |


### 获取GTID配置信息

|          说明          |                    执行语句                     |
| :--------------------: | :---------------------------------------------: |
|    是否开启GTID模式    |        SHOW VARIABLES LIKE 'gtid_mode';         |
|   是否强制GTID一致性   | SHOW VARIABLES LIKE 'enforce_gtid_consistency'; |
| 查询GTID执行的详细情况 |       SELECT * FROM mysql.gtid_executed;        |
|    是否开启级联复制    |    SHOW VARIABLES LIKE 'log_slave_updates';     |

### InnoDB配置信息

|               说明               |                       执行语句                        |
| :------------------------------: | :---------------------------------------------------: |
|            InnoDB版本            |         SHOW VARIABLES LIKE 'innodb_version';         |
| 是否为每个InnoDB表创建单独的文件 |     SHOW VARTABLES LIKE 'innodb_file_per_table';      |
|         InnoDB缓存池大小         |    SHOW VARIABLES LIKE 'innodb_buffer_pool_size';     |
|        InnoDB日志文件大小        |      SHOW VARIABLES LIKE 'innodb_log_file_size';      |
|   InnoDB可以有多少线程同时运行   |   SHOW VARIABLES LIKE 'innodb_thread_concurrency';    |
|     缓冲池中脏页的最大百分比     |   SHOW VARIABLES LIKE 'innodb_max_dirty_pages_pct';   |
|      InnoDB的最大写入IO能力      |     SHOW VARIABLES LIKE 'innodb_io_capacity_max';     |
|          日志缓冲区大小          |     SHOW VARIABLES LIKE 'innodb_log_buffer_size';     |
|     事务提交时刷新日志的频率     | SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit'; |
|          InnoDB IO容量           |       SHOW VARIABLES LIKE 'innodb_io_capacity';       |
|           缓冲池实例数           |  SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';  |

### 监控指标

|                 说明                 |                 执行语句                 |
| :----------------------------------: | :--------------------------------------: |
|             全局状态信息             |           SHOW GLOBAL STATUS;            |
|          当前所有线程的信息          |            SHOW PROCESSLIST;             |
|           当前打开的连接数           |  SHOW STATUS LIKE 'Threads_connected';   |
|     当前线程缓存中有多少空闲线程     |    SHOW STATUS LIKE 'Threads_cached';    |
| 从最近一次服务启动，已创建线程的数量 |   SHOW STATUS LIKE 'Threads_created';    |
|     当前激活的(非睡眠状态)线程数     |   SHOW STATUS LIKE 'Threads_running';    |
|              慢查询次数              |     SHOW status LIKE 'Slow_queries';     |
|           InnoDB读取的行数           |   SHOW status LIKE 'Innodb_rows_read';   |
|           InnoDB插入的行数           | SHOW status LIKE 'Innodb_rows_inserted'; |
|           InnoDB更新的行数           | SHOW status LIKE 'Innodb_rows_updated';  |
|           InnoDB删除的行数           | SHOW status LIKE 'Innodb_rows_deleted';  |

### 关键字、常用操作

#### exists

表A（小表），表B（大表）
```sql
select * from A where c in (select c from B) -- ，用到了A表上c列的索引；
select * from A where exists(select c from B where c=A.c) -- 效率高，用到了B表上c列的索引。
-- 相反的
select * from B where c in (select c from A) -- 效率高，用到了B表上c列的索引
select * from B where exists(select c from A where c=B.c) -- 效率低，用到了A表上c列的索引。
```
> [MySQL多表联合查询有何讲究？](https://mp.weixin.qq.com/s/O121Y0sywAtt4jk9vy4b0w)

#### 逃离符escape

如果要查%或者\_，怎么办呢？使用escape，转义字符后面的%或\_就不作为通配符了，注意前面没有转义字符的%和\_仍然起通配符作用

```sql
select username from user where username like '%xiao/_%' escape '/';
select username from user where username like '%xiao/%%' escape '/';
```

#### 分组排序

```sql
select a.id,a.class,a.source
from student a left join student b on a.class=b.class and a.source<=b.source
group by a.class,a.source
order by a.class,a.source
```
> [阿里二面：group by怎么优化？](https://mp.weixin.qq.com/s/igGKepCsf-SUKChco4P6Lw)
> [MySQL中的distinct和group by哪个效率更高？](https://mp.weixin.qq.com/s/zc7_PZxwaQAwwqNsRG2_KQ)


#### 有就更新,没有就插入

```sql
INSERT INTO table (name, gender)  VALUES ('Jerry', 'boy')  ON DUPLICATE KEY UPDATE name='Jerry'，gender='girl';
-- 或
REPLACE INTO table(name, gender) VALUES ('Jerry', 'girl');
-- 或
REPLACE INTO table(name, gender) SELECT 'Jerry', 'girl';
-- 或
REPLACE INTO test SET name='Jerry', gender='girl';
```

#### 复制表

```sql
-- 不保留数据
create table 表名 as select 列名 from 表名 where 1=2;
create table 表名(列名) as select 列名 from 表名 where 1=2;
-- 保留数据
insert into 表名 ( ) select a.* from ()a
```

#### 重复数据
```sql
-- 显示重复
select * from tablename group by id having count(*)>1
-- 不显示重复
select * from tablename group by id having count(*)=1
```
> [避免MySQL插入重复数据的4种方式](https://mp.weixin.qq.com/s/_hgA89R6GWwJgxC9KBb63A)

#### 分页

limit(start,size) start:从第几条记录开始 size:读取几条记录


#### 定时备份

```shell
#db用户名
dbuser=root
#db密码
dbpasswd="123456"
#ip地址
dbip=127.0.0.1
#备份的数据库名字前缀
pre_name="test"

#备份操作的日志文件
bakfile=/data/sqlbak/log.txt
#备份数据的目录
bakdatadir=/data/sqlbak/bakdata
#备份数据的目录-持久化备份
persist_path=/data/sqlbak/persistdata

#备份保存时间,单位:天
dateoutday=15

#备份单个数据库
bak_single()
{
  #db name
  sdb=$1
  #日期字符串
  datestr=$2

  echo "bak_single ${sdb}_${datestr}.sql.gz BEGIN..." >> ${bakfile}
  #备份
  mysqldump -u${dbuser} -h ${dbip} -p${dbpasswd} --single-transaction  --no-create-db -R -C -B ${sdb} > ${sdb}_${datestr}.sql
  #注释掉sql脚本中 USE DATABASE 语句
  sed -i "s/USE/-- USE/" ${sdb}_${datestr}.sql
  #压缩
  gzip ${sdb}_${datestr}.sql
  #记录日志
  echo "bak_single ${sdb}_${datestr}.sql.gz  COMPLETE..." >> ${bakfile}
}

#备份所有数据库
bak_all()
{
  #日期字符串
  all_datestr=$(date "+%Y_%m_%d_%H_%M_%S")

  date_dir=$(date "+%Y%m%d")

  echo "bak_all begin ${all_datestr} ====================" >> ${bakfile}
  #所有的数据库
  alldb=`mysql -u${dbuser} -h ${dbip} -p${dbpasswd} -e "show databases"`
  for dbname in ${alldb}; do
    {
    #只备份指定前缀的数据库
    if [[ ${dbname} =~ ${pre_name} ]]; then
      bak_single ${dbname} ${all_datestr} 
      fi
    }&
    done
           
    #等待所有数据库备份完成
    wait
    #压缩
    tar czvf dbbak_${all_datestr}.tar *_${all_datestr}.sql.gz
    #删除
    rm *_${all_datestr}.sql.gz
    #是否存在当天日期命名的目录
    if [ ! -d ${bakdatadir}/${date_dir} ]; then
      mkdir -p ${bakdatadir}/${date_dir}
      fi
      #移动到当天日期命名的目录中
      mv dbbak_${all_datestr}.tar ${bakdatadir}/${date_dir}
      echo "bak_all finish dbbak_${all_datestr}.tar ===============" >> ${bakfile}
}

#备份所有数据库-不会定时删除
bak_all_persist()
{
  #日期字符串
  all_datestr=$(date "+%Y_%m_%d_%H_%M_%S")

  echo "bak_all_persist begin ${all_datestr} ====================" >> ${bakfile}
  #所有的数据库
  alldb=`mysql -u${dbuser} -h ${dbip} -p${dbpasswd} -e "show databases"`
    for dbname in ${alldb}; do
      {
        #只备份指定前缀的数据库
        if [[ ${dbname} == ${pre_name} ]]; then
          bak_single ${dbname} ${all_datestr}
          fi
      }&
      done
           
      #等待所有数据库备份完成
      wait

      #压缩
      tar czvf dbpersistbak_${all_datestr}.tar *_${all_datestr}.sql.gz
      #删除
      rm *_${all_datestr}.sql.gz
      #是否存在当天日期命名的目录
      if [ ! -d ${persist_path} ]; then
        mkdir -p ${persist_path}
        fi
        #移动持久化备份的目录中
        mv dbpersistbak_${all_datestr}.tar ${persist_path}
        #
        echo "bak_all_persist finish dbbak_${all_datestr}.tar ====================" >> ${bakfile}
}

#检查过期备份
check_date_out()
{
  #当前目录
  curpath=`pwd`
  #当前日期
  curdate=$(date "+%Y%m%d")
  #最早的保存日期
  lastdate=`date -d "${curdate} - ${dateoutday} day" +%Y%m%d`
  #进入备份目录
  cd ${bakdatadir}
  #目录列表
  pathlst=`ls`
  #检查目录是否过期，删除已过期的目录
  for tmpdate in ${pathlst[*]}; do
    if [[ ${tmpdate} -le ${lastdate} ]]; then
      rm -rf ${tmpdate}
      echo "check_date_out, curdate:${curdate} delete ${tmpdate} " >> ${bakfile}
      fi
      done
      #回到当前目录
      cd ${curpath}
}

case "$1" in
   s)
      bak_single $2 $3
      ;;
   a)
      bak_all
      ;;
   p)
      bak_all_persist
      ;;
  chk) 
      check_date_out
      ;;
   *)
    echo "Please use correct command..."
      ;;
esac

```
**函数功能**
备份是以数据库为单位进行备份的，先备份单个数据库，然后再把所有的备份数据库打包一起

bak_single函数表示备份单个数据，传入参数是需要备份的数据库名字和日期字符串，备份文件名由这两个参数构成，也就是说，备份的文件名由数据库名+日期组成，比如test1_2021_08_16_10_05_30.sql表示test1数据库的备份文件，备份的时间2021_08_16_10_05_30

bak_all函数表示备份所有的数据库，不需要传入参数，先执行SQL语句show databases查询出所有的数据库，然后过滤出我们需要备份的数据库，脚本中需要备份的数据库名都是以test开头的，具体的过滤规则可以按照各自的需求自行修改

bak_all函数for循环体中{和}以及它们后面的&表示启动一个新进程并执行大括号中间的命令，也即每个数据库启动一个进程进行备份，for循环结束之后的wait命令表示等待for循环中所有进程结束，也就是等待所有数据库备份全部完成之后，才会执行wait后面的命令,全部备份完成之后，会创建一个以当前日期命名的目录，并打包所有备份的数据库的SQL脚本，放到此目录中，同时删除原始的备份文件

这里采用的是分库备份，分库备份的好处是：如果所有库都备份成一个备份文件时，恢复其中一个库的数据是比较麻烦的，所以采用分库备份，利于恢复单库数据

check_date_out函数是检查备份目录是否过期，如果过期的话，直接删除过期的目录，脚本开头的变量dateoutday指定了备份保留的天数

bak_all_persist函数是持久备份，备份过程和bak_all函数一样，只不过这里是备份到另一个持久数据的目录中，脚本开头的变量persist_path指定了持久备份目录，目录里面的备份文件不会自动删除，需要手工去删除

持久化目录的主要应用场景：有时线上数据库表有数据校正或者表格结构有变动，为了防止误操作，再执行操作之前，先调用bak_all_persist函数备份下数据库，这样，即使出现误操作，还能恢复数据

备份参数说明
--single-transaction
此选项会将隔离级别设置为可重复读(REPEATABLE READ),让整个数据在dump过程中保证数据的一致性，且不会锁表，这个选项对导出InnoDB的数据表很有用

--no-create-db
正常导出的SQL脚本中会有类似CREATE DATABASE语句，加了--no-create-db选项之后就没有此语句了

-C
服务器传给客户端的过程中先压缩再传递

-R
导出存储过程以及自定义函数

在mysqldump导出数据库SQL脚本之后，sed -i "s/USE/-- USE/" ${sdb}_${datestr}.sql命令的作用是注释掉SQL脚本中的USE DATABASE XXX语句，这个也比较实用的，有时候线上数据会导入到内网，重现线上的一些BUG，但是内网可能已经有一个同名的数据库了，如果注释了这行语句，就可以导入到其他数据库，否则，需要先手工处理SQL脚本，然后再导入

**如何使用**
假如备份MySQL脚本的名字是bak.sh,下面是脚本的使用方法

备份对单个数据库
```shell
# 备份logindb数据库
./bak.sh logindb "2021_08_16_10_05_30"
# 执行上述命令后，会在当前目录下生成名为logindb_2021_08_16_10_05_30.sql.gz的文件
```
备份所有数据库
```./bak.sh a```
检查备份保留时间
```./bak.sh chk```
持久化备份
```./bak.sh p```
添加定时任务
要实现自动备份功能，还需要添加定时任务，间隔指定时间调用备份脚本，执行ctrontab -e命令，输入以下语句

```shell
*/10 * * * * /data/sqlbak/bak.sh a
*/15 * * * * /data/sqlbak/bak.sh chk
```
上述定时任务是每10分钟备份一次所有数据库，每15分钟检查一次过期的备份，当然，具体的备份策略根据具体的场景不同，可以根据实际情况调整

> [一个自动备份MySQL的脚本](https://zhuanlan.zhihu.com/p/427491577)

#### 查看数据大小

```sql
-- 进入information_schema数据库（存放了其他的数据库的信息）
use information_schema;

mysql> use information_schema
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

-- 查询所有数据库数据大小
select concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables;

-- 查看指定数据库数据的大小
select concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables where table_schema='database_name';

-- 查看所有数据库的大小,以MB为单位
SELECT table_schema AS "Database",ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (MB)"
FROM information_schema.tables GROUP BY table_schema;

-- 查看指定数据库的某个表的数据大小
select concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables where table_schema='database_name' and table_name='table_name';

-- 查询索引+数据大小
select concat(round(sum(data_length/1024/1024),2),'MB') as '数据大小' , concat(round(sum(index_length/1024/1024),2),'MB') as '索引大小', round(sum(data_length/1024/1024),2)+round(sum(index_length/1024/1024),2) as 'all' from information_schema.tables where table_schema='数据库名称';

-- 查看数据表的大小并根据大小排序
-- 查询所有表的空间使用情况
SELECT 
    table_schema AS `Database`,
    table_name AS `Table`,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS `Size (MB)`
FROM 
    information_schema.TABLES
ORDER BY 
    (data_length + index_length) DESC
-- 针对特定数据库查询
SELECT 
    table_name AS `Table`,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS `Size (MB)`
FROM 
    information_schema.TABLES
WHERE 
    table_schema = 'your_database_name'
ORDER BY 
    (data_length + index_length) DESC
-- 或者`SHOW TABLE STATUS FROM your_database_name;``
```

#### 分析CPU使用率过高

```sql
-- 1. 使用SHOW VARIABLES LIKE 'performance_schema'; 确保performance_schema是开启的。这个模式可以为你提供关于服务器内部操作的详细信息，这对于诊断性能问题非常有用。
-- 2. 查看数据库连接数
show status like '%threads_connected%';
-- 3. 查看进程,高CPU使用的查询可能会在这里显示。
SHOW PROCESSLIST; -- 或SHOW FULL PROCESSLIST
-- 4. EXPLAIN执行计划
```
使用慢查询日志。请注意，慢查询日志可能会对数据库性能产生一定的影响，因为它需要记录和写入日志文件。因此，在生产环境中使用慢查询日志时，应该谨慎考虑其影响，并根据需要进行适当的调整。要使用MySQL的慢查询日志，可以按照以下步骤进行操作：
1. 首先，确保MySQL的慢查询日志功能已经启用。你可以通过编辑MySQL的配置文件（通常是my.cnf或my.ini）来实现。在[mysqld]部分下添加或修改以下行：

```
# slow_query_log参数用于启用或禁用慢查询日志功能。1表示启用，0表示禁用
slow_query_log = 1
# slow_query_log_file参数用于指定慢查询日志文件的路径
slow_query_log_file = /path/to/your/logfile.log
# long_query_time参数用于设置查询执行时间的阈值（以秒为单位）
long_query_time = 2  # 设置阈值为2秒，你可以根据需要调整这个值
```

2. 重启MySQL服务：修改配置文件后，需要重启MySQL服务以使更改生效。根据你的操作系统和MySQL的安装方式，可以使用相应的命令来重启MySQL服务。
3. 记录慢查询日志：一旦慢查询日志功能启用，MySQL将会记录执行时间超过阈值的查询。这些查询将被写入你在配置文件中指定的日志文件中。你可以使用任何文本编辑器或日志查看工具来查看和查看慢查询日志文件。
4. 分析慢查询日志：一旦你有了慢查询日志文件，可以使用工具如mysqldumpslow来分析日志文件，找出执行缓慢的查询。mysqldumpslow是一个Perl脚本，可以用来分析慢查询日志文件并统计各种查询的执行情况。你可以在终端中运行以下命令来使用mysqldumpslow：

```perl
mysqldumpslow /path/to/your/logfile.log
```
这将显示慢查询日志中的统计信息，包括执行次数、总执行时间、平均执行时间等。你可以根据这些信息找出需要优化的查询。

## 相关文章

### InnoDB

| [InnoDB自增原理都搞不清楚，还怎么CRUD？](https://mp.weixin.qq.com/s/FHoOdlWyefQLJ34MkQgFvA) | [InnoDB原理篇：聊聊数据页变成索引这件事](https://mp.weixin.qq.com/s/V0BX5tRlrrZhfe2-ui2aRA) | [两万字详解！InnoDB锁专题！](https://mp.weixin.qq.com/s/Io64o3tbEcf150dS5_ltbQ) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL存储引擎InnoDB详解](https://mp.weixin.qq.com/s/6BoGlaYpdDjzZy19YhInEw) | [innodb是如何存数据的](https://mp.weixin.qq.com/s/sr5tQF7lNjTcvwg7Fr6HjQ) | [MySQL是如何查询数据的](https://mp.weixin.qq.com/s/ymWeGlaBYWYmfogVDFHo5w) |
| [MySQL是如何实现ACID的?](https://mp.weixin.qq.com/s/KbOiJ8SKJ_wFZcIyDVGD9g) | [一文讲清，MySQL数据库一行数据在磁盘上是怎么存储的？](https://mp.weixin.qq.com/s/qJINIZ3QcXODHSMm1RTPKQ) | [一文讲述MySQL所有的存储引擎](https://mp.weixin.qq.com/s/Oiefzj2b3NZxTwjnRdAFRw) |
| [MySQL的InnoDB引擎原来是这样的](https://mp.weixin.qq.com/s/s2c9L2p5kGC_InjorgDRnQ) | [InnoDB事务隔离级别及其实现原理](https://mp.weixin.qq.com/s/7Xx7rdl7dabbTNJOOLlFVg) |                                                              |

### 主从复制

| [MySQL的主从如何配置](https://mp.weixin.qq.com/s/OHhX9XCQJnrBYdA27ipHHg) | [MySQL主从复制](https://mp.weixin.qq.com/s/xES94DmApf_GGYvT1Ku5QQ) | [面试官：Mysql中主库跑太快，从库追不上怎么整？](https://mp.weixin.qq.com/s/2JtNBnKShIe-0pgt3nYGkg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL定时备份的几种方式，这下稳了！](https://mp.weixin.qq.com/s/gUoDR4WLeHQYzyGtmR5K3w) | [MySQL怎么保证备份数据的一致性？](https://mp.weixin.qq.com/s/zOOBkcl8Ns7Kk-k-WaXNhw) | [删库不跑路！我含泪写下了MySQL数据恢复大法…](https://mp.weixin.qq.com/s/jgM0aZY7UJSA2NMnkmKO3Q) |
| [京东一面：MySQL主备延迟有哪些坑？主备切换策略](https://mp.weixin.qq.com/s/SXnwM_n8UVMkKF04cDVNtw) | [MySQL主从数据不一致，怎么办？](https://mp.weixin.qq.com/s/ALcivJKxtTZUHS-HTnSsXA) | [一文讲解MySQL的主从复制](https://mp.weixin.qq.com/s/bq-sMw6mJg4I_TCvHZ-Zcw) |

### other

| [MySQL自增主键为何不是连续的呢？](https://mp.weixin.qq.com/s/zv25dNB8FXn3KrnN3uJ3uQ) | [线上MySQL的自增id用尽怎么办？](https://mp.weixin.qq.com/s/c6Dx1xEh2BNIuGpxQahAuw) | [MySQL的自增id](https://mp.weixin.qq.com/s/BHE7YTOlo6UZ2WQzRieDxA) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [MySQL用limit为什么会影响性能？](https://mp.weixin.qq.com/s/5o-sAOFpmjRSqIoLUNesOQ) | [MySQL查询limit 1000,10和limit 10速度一样快吗？深度分页如何破解](https://mp.weixin.qq.com/s/1pmVqHBS0CyFJW9ykySF_Q) | [MySQL体系架构简介](https://mp.weixin.qq.com/s/HdE5QiK5Qp3_sGd_cXABIg) |
| [炸裂！MySQL82张图带你飞！※](https://mp.weixin.qq.com/s/HWQbWSQE4O5ndI-NG1Hwwg) | [MySQL面试必会！](https://mp.weixin.qq.com/s/9Ex9tzoQCAEUZNU6iXLHgQ) | [MySQL都不清楚？直接挂了!](https://mp.weixin.qq.com/s/FUyRHpPgw0LFWEOPTPmT0g) |
| [最全91道MySQL面试题](https://mp.weixin.qq.com/s/_Gd-lBfJnhwczNQA4IJyGg) | [138张图带你MySQL入门](https://mp.weixin.qq.com/s/XcNZeHdaMgx35dFoB4_n4A) | [专治MySQL乱码，再也不想看到� �！](https://mp.weixin.qq.com/s/ar35RqaoDgasO-OY0TnDpQ) |
| [MySQL的varchar水太深了，你真的会用吗？](https://mp.weixin.qq.com/s/ImB0O-Z8IXldvt6YQk80jg) | [聊聊MySQL的10大经典错误](https://mp.weixin.qq.com/s/vdz0aS8qgHHHhvZWPuaE8A) | [MySQ8.0推出直方图，性能大大提升！](https://mp.weixin.qq.com/s/3gc8tMfnih7WQoPwTtN8IA) |
| [MySQL模糊查询再也用不着like+%了！](https://mp.weixin.qq.com/s/ZAzRtRnYGhvie8MDZvy1Gg) | [MySQL最大建议行数2000w,靠谱吗？](https://mp.weixin.qq.com/s/PeZ8Py3NNb4CU2BUtxpygg) | [图解MVCC！](https://mp.weixin.qq.com/s/1XmvVNYqt5KycmD8pJanug) |
| [再有人问你什么是MVCC，就把这篇文章发给他！](https://mp.weixin.qq.com/s/WZa5UKYgU-pYKbvovLt1YQ) | [MySQL最朴素的监控方式](https://mp.weixin.qq.com/s/meS5Au1o9qdrSv7505mIcA) | [1亿条数据批量插入MySQL，哪种方式最快](https://mp.weixin.qq.com/s/c71ATJLT6_KXtb_iiUlMjg) |
| [6种MySQL数据库平滑扩容方案剖析](https://mp.weixin.qq.com/s/LvJCi-8cF6HuLo6UXY0Ing) | [什么是插入意向锁？](https://mp.weixin.qq.com/s/rDdUBw803tvjHkJALY5P2w) | [从MySQL读取100w数据进行处理，应该怎么做](https://mp.weixin.qq.com/s/XbSADUXIz1aw0kqp7p5urQ) |
| [MySQL误删数据不用跑路了，快速恢复指南来了](https://mp.weixin.qq.com/s/vyrLOH1NRXXQd0oEnv0M1Q) | [MySQL单表数据最大不要超过多少行？为什么](https://mp.weixin.qq.com/s/ZCM2FYzKw24Fk8yv4a9b0w) | [Mysql如何做表分区](https://mp.weixin.qq.com/s/w2wrccywJWM-zXceCR_JXw) |
| [大白话讲解Mysql中SQL语句的执行流程](https://mp.weixin.qq.com/s/wSnniPaj5s1-cJp_VICXzw) | [盘点Mysql对SQL语句的一些隐式优化技术！](https://mp.weixin.qq.com/s/iCZA97Vm-lxDKEcR4EJPRQ) | [一文带你快速入门Mysql基础知识](https://mp.weixin.qq.com/s/ZwdcTNXKhOEA-T1qUjw-8w) |
| [主键使用自增还是UUID?(uuid_to_bin(UUID())](https://mp.weixin.qq.com/s/edzWAy0A_bTzWWtTzI3q3g) | [突然忘记Mysql登陆密码，怎么办？](https://mp.weixin.qq.com/s/1YQvVj2Uv7yKjk_nuvwLRQ) |                                                              |