+++
title = "Basic-Mybatis(三)：insert、update问题"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Spring"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/a0b7d80da9a7)

MyBatis insert操作返回主键——
[手动封装到bean中，不推荐](https://www.cnblogs.com/sxdcgaq8080/p/10869336.html)  
[官方使用：sqlmap-xml](https://mybatis.org/mybatis-3/sqlmap-xml.html)  
[深入浅出mybatis之返回主键ID](https://www.cnblogs.com/nuccch/p/9067305.html)  

在mapper接口楼中对insert操作进行声明，类似 `@Options(useGeneratedKeys = true, keyProperty = "id")` 

或者使用下面的方式——
```
  <insert id="create" useGeneratedKeys="true" keyProperty="id">
        <selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO SIN_Manager (Account,Password,Name,Phone,Deleted)
        VALUES (#{account}, #{password},#{name},#{phone},0)
    </insert>
```

- updateByPrimaryKey 将为空的字段在数据库中置为NULL
- updateByPrimaryKeySelective 更新新的model中不为空的字段。

后者可以在对应的mapper.xml文件中看到，类似——
```
<update id="updateByPrimaryKeySelective" parameterType="com.xxx.yyy.test.platform.entity.SUResult" >
    update su_result
    <set >
      <if test="groupName != null" >
        group_name = #{groupName,jdbcType=VARCHAR},
      </if>
      <if test="projectName != null" >
        project_name = #{projectName,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
```
