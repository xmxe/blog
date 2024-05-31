---
title: MyBatis相关
categories: Java
img: https://pic1.zhimg.com/v2-b581c1513e4d3f51a7f281a2243de302.jpg

---

##  缓存

一级缓存是默认开启的，它在一个sqlSession会话里面的所有查询操作都会保存到缓存中，一般来说一个请求中的所有增删改查操作都是在同一个sqlSession里面的，所以我们可以认为每个请求都有自己的一级缓存，如果同一个sqlSession会话中2个查询中间有一个insert、update或delete语句，那么之前查询的所有缓存都会清空。

二级缓存是全局的，也就是说；多个请求可以共用一个缓存，二级缓存需要手动开启，有2种方式配置二级缓存，缓存会先放在一级缓存中，当sqlSession会话提交或者关闭时才会将一级缓存刷新到二级缓存中；开启二级缓存后，用户查询时，会先去二级缓存中找，找不到在去一级缓存中找。
MyBatis的一级缓存是基于数据库会话(SqlSession 对象)的，默认开启。二级缓存是基于全局(nameSpace)的，开启需要配置。

开启二级缓存：
1. yml文件中配置
```yaml
mybatis:
  configuration:
    cache-enabled: true
```

2. 在需要开启的mapper.xml中，添加以下代码（在<mapper namespace\>下方）
```xml
<!-- 开启本mapper所在namespace的二级缓存 -->
<cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/> 
<!-- 或者直接 -->
<cache />
```

3. 所有的mapper都开启二级缓存，在mybatis-config.xml中加入以下配置即可
```xml
<settings>
   <!-- 开启所有mapper的二级缓存 -->
   <setting name="cacheEnabled" value="true" />
</settings>

```

| [说说MyBatis二级缓存？关联刷新实现?](https://mp.weixin.qq.com/s/pXGJGRuVWWmH5uf2qVMITg) | [为什么不推荐使用MyBatis二级缓存](https://mp.weixin.qq.com/s/LHNatyT9jdydJmHAzuHpmA) | [Mybatis的一级缓存与二级缓存](https://mp.weixin.qq.com/s/ssj2vAvI4uOlxjHI5jZNZQ) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |

## MyBatis连接数据库

```java
String mybatisConfig= "mybatis-config.xml";
InputStream is = Resources.getResourceAsStream(mybatisConfig);
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(is);
SqlSession session = sessionFactory.openSession();
session.getMapper(Mapper.class).findByid();
```
## Mapper

Dao接口即Mapper接口。接口的全限名，就是映射文件中的namespace的值；接口的方法名，就是映射文件中Mapper的Statement的id值；接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MapperStatement。在Mybatis中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个MapperStatement对象。举例：`com.mapper.StudentDao.findStudentById`，可以唯一找到namespace为com.mapper.StudentDao下面id为findStudentById的MapperStatement。
Mapper接口里的方法，是不能重载的，因为是使用全限名+方法名的保存和寻找策略。Mapper接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的sql，然后将sql执行结果返回。


## foreach批量添加/更新写法

```xml
<insert id="insertList" parameterType="map">
 begin try
  insert into gs_job_pfr_mx (tag_code,val,time) values
  <foreach collection="list" item="item" separator=","  index="index">
   (#{item.tag},#{item.value},GETDATE())
  </foreach>
 end try
 begin catch
  update gs_job_pfr_mx set val =
<foreach collection="list" item="item" separator=" "open="case tag_code" close="end">
  when #{item.tag} then #{item.value}
 </foreach>,
  time = GETDATE()
  where tag_code in
 <foreach collection="list" item="item" separator="," open="(" close=")">
    #{item.tag}
  </foreach>
 end catch
 </insert>

<!-- trim标签-->
<update id="updateYsz" parameterType="tblYsz">
    update tblysz set
        <trim suffixOverrides=",">
            <if test="name!=null">name=#{name},</if>
            <if test="jgdm!=null">jgdm=#{jgdm},</if>
            <if test="xz!=null">xz=#{xz},</if>
        </trim>
     <where>id=#{id}</where>
 </update>

<!-- choose标签 -->
<select id="selectByIdOrName" resultMap="BaseResultMap" parameterType="com.homejim.mybatis.entity.Student">
    select
    <include refid="Base_Column_List" />
    from student
    where 1=1
    <choose>
      <when test="studentId != null">
        and student_id=#{studentId}
      </when>
      <when test="name != null and name != ''">
        and name=#{name}
      </when>
      <otherwise>
        and 1=2
      </otherwise>
    </choose>
  </select>

<!-- resultMap-->
<resultMap id="BaseResultMap"type="com.xmxe.entity.User">
<id property="id" column="id" jdbcType="INTEGER" javaType="java.lang.IntegerINTEGER"/>
<result property="username" column="username" jdbcType="VARCHAR" javaType="java.lang.String"/>
<result property="password" column="password" jdbcType="VARCHAR" javaType="java.lang.String"/>
</resultMap>

<sql id="commonsSql">
	id,username,password
</sql>

```

> [MyBatis动态SQL](https://mp.weixin.qq.com/s/-aIum139UB_dt7OIaiccfA)
> [10种超好用的MyBatis写法](https://mp.weixin.qq.com/s/G7JpSRbEAw0dDbSj3lneOw)


## 文章

| [MyBatis一个简单配置搞定加密、解密](https://mp.weixin.qq.com/s/XPmXbCnJPYHOqPAzpKxmQw) | [MyBatis的执行流程，写得太好了！](https://mp.weixin.qq.com/s/JwFw8Yi-5Miap83i99LX2A) | [Mybatis接口Mapper内的方法为啥不能重载？](https://mp.weixin.qq.com/s/rQvhsLBo90uclM3i3jobIA) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [面试官问你MyBatis SQL是如何执行的？把这篇文章甩给他](https://mp.weixin.qq.com/s/3eBU2c2AlcsOvZYpAD337A) | [mybatis的mapper为啥只有接口没有实现类，它却能工作？](https://mp.weixin.qq.com/s/Aet3yVkcGZmgRwuRcfP8KQ) | [MyBatis常见面试题总结](https://mp.weixin.qq.com/s/NjeIGSGiXgp-TDF2IBm-Xg) |
| [Mybatis中的DAO接口和XML文件里的SQL是如何建立关系的？](https://mp.weixin.qq.com/s/EE01Vyhjrw_Fcfx4-b2SJA) | [MyBatis多条件查询、动态SQL、多表操作、注解开发，应有尽有，一网打尽！](https://mp.weixin.qq.com/s/fxYp_UPiJHVyrb5si76I9Q) | [详解MyBatis事务管理，彻底颠覆你对事务的理解](https://mp.weixin.qq.com/s/3mBW8D70ogXEO0Naon2uqw) |
| [2万多行MyBatis源码中有多少设计模式](https://mp.weixin.qq.com/s/Q-Im8ip3w1N-ISFtV9MBHA) | [MyBatis如何实现流式查询](https://mp.weixin.qq.com/s/srMRJeSQ_9T4l4j4g-RGUA) | [Mybatis的延迟加载，你知道是怎么实现的么？](https://mp.weixin.qq.com/s/fFQ6av72FITCuuvxjK37eA) |
| [想让代码更优雅？Mybatis类型处理器了解一下！](https://mp.weixin.qq.com/s/dT4pZxjpjV095UtTpmKsaA) | [1亿条数据批量插入MySQL，哪种方式最快？](https://mp.weixin.qq.com/s/TLqu384RS4FI6T3I1IydOQ) | [MyBatis批量插入别再乱用foreach了，5000条数据花了14分钟](https://mp.weixin.qq.com/s/V5WO0fQFXW8vgDQZzZ4n7w) |
| [MyBatis批量插入的五种方式](https://mp.weixin.qq.com/s/baFO97l6772jDKcWkLhx_A) | [13秒插入30万条数据，这才是批量插入正确的姿势！](https://mp.weixin.qq.com/s/yjlam0vVHvveIqG20_sBqg) | [MyBatis架构与原理深入解析，面试随便问！](https://mp.weixin.qq.com/s/xZ9MJgPa6vYUfYLcJcgSlw) |
| [7张图解锁MyBatis整体脉络，让你轻松拿捏面试官](https://mp.weixin.qq.com/s/aBKVdkhPG53ergM1CIcUUw) |                                                              |                                                              |

