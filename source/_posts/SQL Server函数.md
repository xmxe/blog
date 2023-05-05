---
title: SQL Server函数
categories: 数据库
index_img: /assert/sqlserver.jpg
img: https://picx.zhimg.com/v2-499cbcaf31cf035a9850972edb24939f_1440w.jpg

---

## sql server行转列

类似MySQL group_concat()使用stuff()

stuff()将字符串插入到另一个字符串中。它从第一个字符串的开始位置删除指定长度的字符；然后将第二个字符串插入到第一个字符串的开始位置。

```sql
select stuff('ABCDEFG',2,3,''hijk) = AhijkEFG
SELECT id, value = stuff((SELECT ',' + value FROM temp t WHERE t.id = temp.id FOR xml path('')),1,1,'') FROM temp GROUP BY id

-- 实例
SELECT t.unit_id,t.value_param,t.code_param,conf.state
FROM
(SELECT 
unit_id,
STUFF((SELECT ',' + val1 FROM gs_pss_config b WHERE b.unit_id = a.unit_id AND b.conf_type = #{conf_type} FOR xml path('')),1,1,'')AS value_param,
STUFF((SELECT ',' + param_code FROM	gs_pss_config c	WHERE c.unit_id = a.unit_id	AND c.conf_type = #{conf_type} FOR xml path('')),1,1,'')AS code_param
FROM
gs_pss_config a
WHERE a.conf_type = #{conf_type} GROUP BY a.unit_id)t
LEFT JOIN 
gs_yctp_unit_config conf on t.unit_id = conf.unit_id 
```


## 统计数据考核密度

```sql
select s.real_tag,s.tag_name,s.unit_id,
sum(case when s.density = 0 then 1 else 0 end ) as density0,
sum(case when s.density > 0 and s.density <=20 then 1 else 0 end ) as density20,
sum(case when s.density > 20 and s.density <=50 then 1 else 0 end ) as density50,
sum(case when s.density > 50 and s.density <=70 then 1 else 0 end ) as density70,
sum(case when s.density > 70 and s.density < 100 then 1 else 0 end ) as density99,
sum(case when s.density <=100 then 1 else 0 end ) as density100
from
(select * from gs_job_density where 1 =1 and start_time >=#{start_time} and end_time <=#{end_time}) s
group by s.real_tag,s.tag_name,s.unit_id having 1 =1 and charindex(#{real_tag},real_tag)>0
order by s.unit_id desc
OFFSET ${start} ROW FETCH NEXT ${limit} rows only
```

## 批量添加/更新

```sql
insert into tbl(a,b) values (1,2),(3,4),(5,6);
update gs_job_pfr_mx SET val = 
CASE tag_code
WHEN 'cals.GD_SH05_UUB00W0101Y' THEN '1.0'
WHEN 'cals.GD_SH05_UUB00W0102Y' THEN '1.1'
WHEN 'cals.GD_SH05_UUB00W0101D' THEN '1.2'
WHEN 'cals.GD_SH05_UUB00W0102D' THEN '1.3'
END,
TIME = GETDATE()
WHERE tag_code IN('cals.GD_SH05_UUB00W0101Y', 'cals.GD_SH05_UUB00W0102Y', 'cals.GD_SH05_UUB00W0101D', 'cals.GD_SH05_UUB00W0102D')
```

## 分组排序

```sql
select row_number() over(partition BY 分组字段 order by 排序字段) as rowNums,* from 表名

-- 例：需要筛选两条数据，一条是每个区间段所有数据中最大的，另外一条是个数*10%的数值
select
c.count,c.maxfhl,c.qjd,c.type,b.rownum,b.tag_name,b.fhl
from
(select count(qjd) as count,max(fhl) as maxfhl,qjd,type from gs_job_grade2 where tag_name = #{tag_name} GROUP BY qjd,type)c
join
(select row_number() over(partition BY qjd,type order by fhl desc) as rownum,* from gs_job_grade2
where tag_name = #{tag_name}) b
on c.qjd = b.qjd and c.type = b.type
where b.rownum = CEILING(c.count*0.1) and c.type = #{type} and c.qjd = #{qjd}
order by c.qjd,c.type

-- 排序时添加序号列
select ROW_NUMBER()OVER(ORDER BY 用来排序的列的列名),XXX,XXX from XXX
```


## 差集、交集

```sql
-- 取差集
select Name from Person1
except
select Name from Person2

-- 取交集
select * from Person1
InterSect
select * from Person2
```


## 函数

```sql
floor() -- 返回小于或等于所给数字表达式的最大整数
floor(1.1)=1 floor(2)=2

ceiling() -- 返回大于或等于所给数字表达式的最小整数。
ceiling(1.1)=2 ceiling(2)=2

round() -- 四舍五入

substring()
select substring('abdcsef',1,3) abd

reverse(express) -- 将字符串从尾部到头部排序

charindex(expression1,expression2[,start_location])获取某字符第一次出现的位置
expression1 必需 ---要查找的子字符串
expression2 必需 ---父字符串
start_location 可选 ---指定从父字符串开始查找的位置，默认位置从1开始
charindex(#{tag_name},table.tagname)
```

## SQL关联更新

```sql
-- SQL关联使用聚合函数更新sql存在更新不存在添加

if exists (select * from dbo.users s where s.name='张三')
update users set sex='男' where name = '张三'
ELSE
insert into users (name,sex) values ('张三','女')
```

## 转换数据类型的两种方式

```sql
select * from TableName order by cast(colName as int);

select * from TableName order by convert(int,colName);
```

## 树形查询

```sql
with cte_child(id,areaName,pid,level)
as
(
 --起始条件
select id,areaName,pid,0 as level from erp_area where id = 1 -- 优先列出第一节点查询条件或子节点查询条件

union all
--递归条件
select a.id,a.areaName,a.pid,b.level+1 from erp_area a
inner join cte_child b
on a.pid=b.id
)
select * from cte_child 
```

## 将文件中的数据插入数据库


```sql
BULK INSERT  [ schema_name ] . [ table_name ]
FROM 'data_file'
[ WITH (Arguments)]

-- 例
bulk insert dbo.tablename
from 'D:\abc.txt'
WITH(
    FIELDTERMINATOR = ',',
    ROWTERMINATOR = '\n',
    DATAFILETYPE ='widechar',
    KEEPNULLS
)
```
**Arguments**

- data_file：指定数据文件的full path，bulk insert命令将数据从该文件导入到Target Table中
- ROWTERMINATOR = 'row_terminator'： 指定分隔行的字符，使用该字符来分割行（Row）
- FIELDTERMINATOR = 'field_terminator'：指定分隔字段的字符，使用该字符来分割字段（Field或Column）
- DATAFILETYPE = { 'char' | 'native'| 'widechar' | 'widenative' }：指定data file编码（Encoding）的类型，推荐使用widechar编码
- CODEPAGE = { 'ACP' | 'OEM' | 'RAW' | 'code_page' }：如果data file中含有单字节（char或varchar）字符数据，使用CodePage参数指定字符列的CodePage
- BATCHSIZE = batch_size：指定一个batch包含的数据行数量，在将数据复制到Table中时，每一个Batch作为一个单独的事务，如果一个batch复制失败，那么事务回滚。默认情况下，data file中的所有数据作为一个batch
- CHECK_CONSTRAINTS：指定在执行bulk insert操作期间，必须检查插入的数据是否满足Target Table上的所有约束。如果没有指定CHECK_CONSTRAINTS选项，则所有CHECK和FOREIGN KEY约束都将被忽略，并且，在此操作之后，表上的所有约束将标记为不可信（not-trusted）
- FIRE_TRIGGERS：指定是否启动Insert触发器，如果指定该选项，每个batch成功插入后，会执行Insert触发器；如果不指定该选项，不会执行Insert触发器
- KEEPIDENTITY：指定将data file中的标识值插入到标识列（Identity Column）中，如果不指定KeepIdentity选项，Target Table中的ID列会自动分配唯一的标识值
- KEEPNULLS：指定在执行bulk insert操作期间，空列（Empty Columns）应保留NULL值，而不是插入列的默认值
- TABLOCK：指定在执行bulk insert操作期间，获取一个表级锁，持有表级锁，能够减少锁竞争(Lock Contention)，提高导入性能