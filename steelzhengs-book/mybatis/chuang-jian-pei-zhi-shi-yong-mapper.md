# 创建、配置、使用Mapper

### 定义接口

```kotlin
package com.steelzheng.app.mapper  
  
import com.steelzheng.app.domain.News  
import org.apache.ibatis.annotations.Param  
  
  
interface NewsMapper {  
  
    fun saveNews(news: News): Int  
  
    fun updateNews(news: News): Int  
  
    fun deleteNews(id: Int)  
  
    fun selectAllNews(): List<News>  
  
    fun selectAllNews2(): List<News>  
  
    fun findNews(id: Int): News?  
  
    fun updateNewsInRange(@Param("n") n: News, @Param("start") start: Int, @Param("end") end: Int): Int  
}
```

### 定义XML文件

```XML

<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
  
<mapper namespace="com.steelzheng.app.mapper.NewsMapper">  
    <!-- 插入数据并返回数据库id值 -->
    <insert id="saveNews" useGeneratedKeys="true" keyProperty="id">  
        insert into news_inf values (null, #{title}, #{content})
    </insert>  
  
    <update id="updateNews">  
        update news_inf set news_title=#{title}, news_content=#{content}
        where news_id = #{id}    
    </update>  
  
    <update id="updateNewsInRange">  
        update news_inf  set news_title=#{n.title}, news_content=#{n.content}        
        where news_id between #{start} and #{end}
        </update>  
  
    <delete id="deleteNews">  
        delete from news_inf where news_id = #{id}    
    </delete>  


    <!-- resultType用法 -->
    <select id="selectAllNews" resultType="com.steelzheng.app.domain.News">  
        select news_id id, news_title title, news_content content  
        from news_inf    
    </select>  


	<!-- resultMap用法 -->
    <select id="selectAllNews2" resultMap="newsMap">  
        select * from news_inf  
    </select>  
  
    <resultMap id="newsMap" type="News">  
        <id property="id" column="news_id"/>  
        <result property="title" column="news_title"/>  
        <result property="content" column="news_content"/>  
    </resultMap>  


    <!-- include复用标签， 可以复用sql语句-->
    <sql id="newsColumns">  
        ${tb}.news_id id, ${tb}.news_title title, ${tb}.news_content content  
    </sql>  
  
    <select id="findNews" resultType="com.steelzheng.app.domain.News">  
        select  
        <include refid="newsColumns">  
            <property name="tb" value="news"/>  
        </include>  
        from news_inf news  
        where news_id = #{id}    
    </select>  
</mapper>
```

### 在mybatis-config.xml中配置Mapper

两种配置方法：

* 使用`mapper`标签： 单独配置某一个使用的Mapper xml文件
* 使用`package`标签：指定所有Mapper的包路径

一般推荐使用package，避免Mapper文件较多时的麻烦

```xml
<mappers>  
    <!-- 配置mybatis需要加载的Mapper -->  
    <!--
    <mapper resource="com/steelzheng/app/mapper/NewsMapper.xml"/>    
    <mapper resource="com/steelzheng/app/mapper/UserMapper.xml"/>
    -->    
    <package name="com.steelzheng.app.mapper"/>  
</mappers>
```

### 如何使用Mapper

配置完mapper后，如何使用呢？我们可以通过mapper进行增删改查的功能

下面来写一个单元测试，获取NewsMapper，并执行相关操作：

1. 先封装要给BaseMapperTest类，用于获取Mapper文件

```kotlin
package com.steelzheng.app  
  
import org.apache.ibatis.io.Resources  
import org.apache.ibatis.session.SqlSession  
import org.apache.ibatis.session.SqlSessionFactoryBuilder  
import org.junit.After  
import org.junit.Before  
import java.io.IOException  
  
open class BaseMapperTest {  
    lateinit var session: SqlSession  
  
    @Before  
    @Throws(IOException::class)  
    fun setup() {  
        val resource = "mybatis-config.xml"  
        val inputStream = Resources.getResourceAsStream(resource)  
        val sqlSessionFactory = SqlSessionFactoryBuilder().build(inputStream)  
        session = sqlSessionFactory.openSession()  
    }  
  
    @After  
    fun tearDown() {  
        session.commit()  
        session.close()  
    }  
  
    fun <T> getMapper(cls: Class<T>): T {  
        session.  
        return session.getMapper(cls)  
    }  
}
```

2. 创建NewsMapperTest类

```kotlin
package com.steelzheng.app  
  
import com.steelzheng.app.domain.News  
import com.steelzheng.app.mapper.NewsMapper  
import org.apache.log4j.Logger  
import org.junit.Test  
  
  
class NewsMapperTest : BaseMapperTest() {  
  
    private val newsMapper by lazy {  
        getMapper(NewsMapper::class.java)  
    }  
  
    private val log by lazy {  
        Logger.getLogger(NewsMapper::class.java)  
    }  
  
    @Test  
    fun saveNews() {  
        val news = News(null,"这是标题1", "这是内容1")  
        newsMapper.saveNews(news)  
        log.info(news.id)  
    }  
  
    @Test  
    fun updateNews() {  
        val news = News(1, "这是标题1 修改", "这是内容1 修改")  
        newsMapper.updateNews(news)  
    }  
  
    @Test  
    fun updateNewsInRange() {  
        val news = News(null, "这是标题2 修改", "这是内容2 修改")  
        newsMapper.updateNewsInRange(news, 1, 10)  
    }  
  
  
    @Test  
    fun testSelectAllNews() {  
        val news = newsMapper.selectAllNews()  
        log.info(news)  
    }  
  
    @Test  
    fun testSelectAllNews2() {  
        val news = newsMapper.selectAllNews2()  
        log.info(news)  
    }  
  
    @Test  
    fun testFindNews() {  
        val news = newsMapper.findNews(3)  
        log.info(news)  
    }  
  
    @Test  
    fun testDeleteNews() {  
        newsMapper.deleteNews(1)  
    }  
}
```

案例使用的数据库

```sql
drop database if exists mybatis;  
create database mybatis;  
use mybatis;  
create table news_inf  
(  
   news_id integer primary key auto_increment,  
   news_title varchar(255),  
   news_content varchar(355)  
);  
  
create table user_inf  
(  
   user_id integer primary key auto_increment,  
   user_name varchar(255),  
   password varchar(255)  
);  
  
select * from news_inf;  
select * from user_inf;
```
