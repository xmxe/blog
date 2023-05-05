---
title: Oracle相关及常用函数
categories: 数据库
index_img: /assert/oracle.jpg
img: https://gimg2.baidu.com/image_search/src=http%3A%2F%2F29520719.s21i.faiusr.com%2F2%2FABUIABACGAAgpufK7QUoj7DrqQIw6Ac4_QQ%21800x800.jpg&refer=http%3A%2F%2F29520719.s21i.faiusr.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1669537603&t=31ddd7a1992896268a14e72c35ed60ed

---

### Sys和System用户区别

- Sys:拥有DBA、SysDBA、Sysoper（系统操作员）角色或权限，是Oracle权限最高的用户，只能以SysDBA或Sysoper登录，不能以Normal形式登录。
- System:拥有DBA、Sysdba权限或角色，可以以普通用户的身份登录。

### Sysdba、Sysoper、DBA区别

- Sysdba用户:可以改变字符集、创建删除数据库、登录之后用户是SYS（shutdown、startup）
- Sysoper:用户不可改变字符集、不能创、删数据库、登陆之后用户是PUBLIC（shutdown、startup）
- DBA用户：只有在启动数据库后才能执行各种管理工作。
- Sysdba> Sysoper>普通的DBA

### Oracle中的角色

1. CONNECT
2. RESOURCE
3. DBA
4. EXP_FULL_DATABASE
5. IMP_FULL_DATABASE
6. DELETE_CATALOG_ROLE
7. EXECUTE_CATALOG_ROLE
8. SELECT_CATALOG_ROLE

**CONNECT角色**：--是授予最终用户的典型权利，最基本的

```sql
1. ALTER SESSION --修改会话
2. CREATE CLUSTER --建立聚簇
3. CREATE DATABASE LINK --建立数据库链接
4. CREATE SEQUENCE --建立序列
5. CREATE SESSION --建立会话
6. CREATE SYNONYM --建立同义词
7. CREATE VIEW --建立视图www_bitscn_com中国.网管联盟
```
**RESOURCE角色**： --是授予开发人员的

```sql
1. CREATE CLUSTER --建立聚簇
2. CREATE PROCEDURE --建立过程
3. CREATE SEQUENCE --建立序列
4. CREATE TABLE --建表
5. CREATE TRIGGER --建立触发器
6. CREATE TYPE --建立类型
```
**DBA角色**：拥有系统所有系统级权限（系统管理员）

**IMP_FULL_DATABASE角色、EXP_FULL_DATABASE角色**：

```sql
1. BACKUP ANY TABLE --备份任何表
2. EXECUTE ANY PROCEDURE --执行任何操作
3. SELECT ANY TABLE --查询任何表
```

**SELECT_CATALOG_ROLE角色**具有从数据字典查询的权利，
(尤其是此处数据字典：就是数据库运行时的各种信息参考：Oracle 11g体系结构--数据字典)

**EXECUTE_CATALOG_ROLE**角色具有从数据字典中执行部分过程和函数的权利。


### 分区表

当表中的数据量不断增大，查询数据的速度就会变慢，应用程序的性能就会下降，这时就应该考虑对表进行分区。表进行分区后，逻辑上表仍然是一张完整的表，只是将表中的数据在物理上存放到多个表空间(物理文件上)，这样查询数据时，不至于每次都扫描整张表。

### PL/SQL /Oracle乱码问题解决方案

```sql
Select userenv(‘language’) from dual; -- 查看服务器端编码

select * from V$NLS_PARAMETERS；-- 查看NLS_LANGUAGE的值与第一个的查询结果是否一致，假如不一致需要设置环境变量，变量名：NLS_LANG 变量值：第1个查到的值 重启PL/SQL(假如在乱码之前已经插入数据，那么配置环境变量后依然乱码，需要删除数据重新导入)

```
### Oracle的操作

1. cmd进入oracle 
```sql
sqlplus 账户名/密码 as 角色名 --（sys用户必须带as sysdba）例:sqlplus sys/admin as sysdba

exit --退出
```
2. 用户账号相关操作
```sql
-- 创建用户
create user xxx identified by xxx;
-- 授权
grant create session, connect, resource to xxx;
-- 删除用户 加cascade可以一同删除用户数据
drop user username cascade;
-- 解锁登陆账号
alter user scott account unlock;
-- 冻结登陆账号
alter user dbaName account lock;
-- 修改登录账号密码
alter user dbaName identified by "password";

-- 查看所有用户相关信息
SELECT * FROM DBA_USERS;-- 查询DBA用户
SELECT * FROM ALL_USERS;-- 查询所有用户
SELECT * FROM USER_USERS;-- 查询系统用户

-- 查看用户系统权限
SELECT * FROM DBA_SYS_PRIVS;
SELECT * FROM USER_SYS_PRIVS;

-- 查看用户对象或角色权限
SELECT * FROM DBA_TAB_PRIVS;
SELECT * FROM ALL_TAB_PRIVS;
SELECT * FROM USER_TAB_PRIVS;

-- 查看所有角色
SELECT * FROM DBA_ROLES;

-- 查看用户或角色所拥有的角色
SELECT * FROM DBA_ROLE_PRIVS;
SELECT * FROM USER_ROLE_PRIVS;
select * from role_sys_privs
```
3. 表空间相关
```sql
Select * FROM Dba_Tablespaces; -- 查看所有的表空间
Select * FROM DBA_DATA_FILES; -- 查看所有的表空间以及对应的地址

-- 创建临时表空间
create temporary tablespace user_temp
tempfile 'D:\APP\ADMINISTRATOR\ORADATA\ORCL\user_temp.dbf'
size 50m  --初始空间50m
autoextend on -- on为表空间自动扩展
next 50m maxsize 20480m -- 每次50m最大2048m
extent management local; -- 本地管理表空间

-- 创建数据表空间
create tablespace user_data
logging
datafile 'D:\APP\ADMINISTRATOR\ORADATA\ORCL\user_data.dbf'
size 50m
autoextend on
next 50m maxsize 20480m
extent management local;

-- 创建用户指定表空间
create user 用户名 identified by 密码 default tablespace 表空间名；
```

### Oracle相关函数

#### wm_concat()行转列
wm_concat()函数是oracle中独有的,mysql中有一个group_concat()函数。这两个函数的作用是相同的，它们的功能是：实现行转列功能，即将查询出的某一列值使用逗号进行隔开拼接，成为一条数据

```sql
-- 根据年龄获取学生的分数
select age,to_char(wm_concat(name)) as name,to_char(wm_concat(score)) as score from student t group by age;

-- age  name     score
-- 18  张三,李四  81,82
-- 19  王五,赵六  67,90
```

#### decode

decode(c1,c2,c3)/decode(c1,c2,c3,c4)/decode(c1,c2,c3,c4,c5)/decode(c1,[c2,c3],[c4,c5],[c6,c7]....,[C2x,C2x+1] [,C2x+2])
从c1之后开始,每两个参数看做是一组数,拿每组数的第一个参数和c1比较,如果相同则返回第二个参数:比如 如果c2==c1 return c3如果该组数的第一个参数和c1不相同,则比较下一组:比如 如果c2<>C1继续判断C4==C1? 相同return c5
例  decode(type,'a','11','b','12','c','13','d','14','e','15',type)

#### 树形查询

```sql
select area_code from (select * from tableA where isvalid=1) start with area_code = 5002 connect by  prior parent_area_code = area_code;

-- 父子关系查询（start with connect by prior）
select * from tabname t where 条件 start with t.org_parent_code='10000008' connect by t.org_code = prior t.org_parent_code
```

#### 数值函数

1. mod(x,y) 取模 求余数
2. nvl(x,y) 如果x的值为空,则返回y
```sql
select ename,sal,comm,sal+nvl(comm,0) from emp;
-- 数值列可以直接做加减乘除运算,如果数值列值为null则加减乘除后也是null需要使用nvl函数去空
```


#### 精确指定位数的函数

1. round(c1,c2) 能够四舍五入
2. trunc(c1,c2) 直接舍去
3. c2>0 : c2表示有多少位小数
```sql
select round(21.23512,2) from dual;   --  21.24
select trunc(21.23512,2) from dual;   --  21.23
```
4. c2<0 : c2表示小数点向左精确|c2|位
```sql
select round(2163.512,-2) from dual;  --  2200
select trunc(2163.512,-2) from dual;  --  2100
```
5. c2如果不存在,表示只保留整数部分
```sql
select round(2163.512,0) from dual;  --  2164
select round(2163.512) from dual;    -- 2164
```

#### 字符函数

- length() 求字符串长度
```sql
-- 查询姓名长度为5个字符的员工姓名
select ename,length(ename) from emp where length(ename)=5;
```
- lower() 全部转为小写
- upper() 全部转为大写
- initcap() 单词首字母大写其他字母小写
```sql
select ename,lower(ename),upper(ename),initcap(ename) from emp
```
- 截取字符串:substr(c1,c2[,c3]) c1: 原字符串 c2: 从哪个位置开始截取 c3: 截取长度(默认截取到最后)
```sql
-- 查询员工姓名,截取员工姓名的第一个字符,再截取姓名的最后一个字符
select ename,sal,substr(ename,1,1),substr(ename,length(ename)) from emp;
-- 查询员工姓名,截取员工姓名中最中间的一个字符(偶数个截取后一位)
select ename, substr(ename,trunc(length(ename)/2)+1,1) from emp;
```

- 索引字符串:instr(c1,c2,c3,c4) c1: 原字符串 c2: 要查找的字符串 c3: 从哪个位置开始查找 默认值1 c4: 第几次出现 默认值1
```sql
-- 查询员工姓名中带有E字符的员工
select * from emp where ename like '%E%';
select * from emp where instr(ename,'E')<>0;

select instr('sanhaoxuesheng','aow') from dual;
-- instr能够用来替代like实现模糊查询
select * from emp where instr(ename,'%')<>0;--
```

- 拼接字符串: 符号: || 函数: concat(c1,c2) 将c1 c2拼接为一个字符串
```sql
select concat(concat('123','abc'),'eee') from dual;
select '123' || 'abc' || 'eee' from dual;
```

#### 日期函数
两个日期之间可以做减法,单位是天 两个日期之间不能做加法
- add_months(c1,c2) c1日期类型,c2整数在c1日期的基础上增减c2个月份
```sql
-- 查询当前系统时间之前的一个月
select add_months(sysdate,-1) from dual;
-- 查询十年后的今天
select add_months(sysdate,120) from dual;
```
- months_between(c1,c2) c1 c2都是日期类型 计算两个日期之间相差多少个月份
```sql
select round(months_between(sysdate,hiredate),2) from emp;
```
- last_day(c)
```sql
-- 查询给定日期所在月份的最后一天
select last_day(sysdate) from dual;
-- 查询下一个月的第一天 
select last_day(sysdate)+1 from dual;
-- 查询上一个月的第一天 
select last_day(add_months(sysdate,-2))+1 from dual;
```
- 计算日期:
round(c1,c2)  四舍五入 trunc(c1,c2)  直接舍去
c1:日期类型的值  c2:字符类型,日期格式 在c1日期的基础上,精确到c2指定的日期格式 如果比c2低的日期格式,默认是初始值

```sql
-- 将当前系统时间精确到年份
select round(sysdate,'yyyy') from dual;
-- 如果当前时间是: 2017-2-11 13:00:00 //2017-1-1 00:00:00
```
5. 查询本年度的一季度的起始日期
```sql
select trunc(sysdate,'yyyy'),add_months(trunc(sysdate,'yyyy'),3) from dual;
```
6. next_day(c1,c2) c1: 日期类型  c2:字符或整数: 周期值 从c1日期来时,查找c2指定的周期的日期
```sql
-- 查询当前系统时间之后的第一个星期一
select sysdate,next_day(sysdate,'星期四') from dual;
```

#### 转换函数
1. to_date(c1,c2)

2. to_timestamp(c1,c2) c1: 字符类型的日期 c2:日期格式
能够根据c2指定的格式将c1字符类型的日期转变为日期类型
```sql
insert into emp(empno,hiredate) values(1001, to_date('2017-1-1','yyyy-dd-mm') );
insert into emp(empno,hiredate) values(1002, to_date('2017-1-1 12:21:20','yyyy-dd-mm hh24:mi:ss') );
```

3. to_char()
```sql
-- 将一个数值转变为一个字符串
select * from emp where '1'=1;
select * from emp where '1'=to_char(1);

-- 格式化字符串或数值
select to_char(2341212412,'$999,999,999,999.00') from dual;
-- 第二个参数的长度必须大于第一个参数
select to_char(2341212412,'L999,999,999,999.00') from dual;

-- 与日期的转换
-- to_char(c1,c2) c1: 日期类型的值 c2:日期格式 按照c2指定的日期格式,从c1中取值

-- 查询当前系统时间的月份
select to_char(sysdate,'month') from dual;
-- 查询当前系统时间的年月日
select to_char(sysdate,'yyyy-mm-dd') from dual;
-- 查询当前系统时间是本年度第几天
select to_char(sysdate,'yyyy-ddd-dd-d') from dual;
```
4. 对比to_date 和to_char:
to_date('2017','yyyy') 将字符串转变成日期 第一个参数是:字符串类型的值
to_char(sysdate,'yyyy') 按照指定的格式将日期转变为字符串 第一个参数是:日期值


### 关键字

- minus从第一个查询结果中,减去第二个查询结果中重复出现的数据 剩余:第一个查询结果中的部分数据
```sql
select * from emp where ename like '%E%'
minus 
select * from emp where ename like '%S%';
```
- intersect 交集
```sql
select * from emp where ename like '%E%'
intersect
select * from emp where ename like '%S%';
```

### 创建表同时复制数据

```sql
create table 新表名(列名) as select 列名 from 旧表 where 条件;
-- 如果表已存在，可以向已存在的表中插入数据：
insert into 表名(列名) select 列名 from 旧表 where 条件;
```


### 有则更新 无则新增
根据源表对目标表进行匹配查询，匹配成功时更新，不成功时插入

语法
MERGE INTO 目标表 a USING 源表 b ON (a.字段1 = b.字段2 and a.字段n = b.字段n)
WHEN MATCHED THEN UPDATE SET a.新字段 = b.字段 WHERE 限制条件 WHEN NOT MATCHED THEN INSERT (a.字段名1，a.字段名n) VALUES(b.字段值1, b.字段值n) WHERE 限制条件123456789
例子:
```sql
MERGE INTO T_SCHE_PYTHON_JOB T1
USING (SELECT '1001' AS JOB_DEF_ID FROM dual) T2
ON ( T1.JOB_DEF_ID=T2.JOB_DEF_ID)
WHEN MATCHED THEN
    UPDATE SET T1.param = ${param}
WHEN NOT MATCHED THEN 
    INSERT (JOB_DEF_ID,param) VALUES(1001,${param}); 
```