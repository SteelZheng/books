# MyBatis配置详解

### 创建配置文件

文件名随便定义，一般建议定义为：mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
<configuration>  

	<!-- 起别名的作用-->
    <typeAliases>        
        <!-- <typeAlias type="com.steelzheng.app.domain.News"/> -->  
        <package name="com.steelzheng.app.domain"/>  
    </typeAliases>  
    
    <typeHandlers>        
    <typeHandler handler="com.steelzheng.app.handler.NameTypeHandler" 
	    jdbcType="VARCHAR" javaType="com.steelzheng.app.domain.Name"/>  
    </typeHandlers>  

	<!-- 配置对象工厂，对象工厂的作用是用于给指定的对象封装一些属性值 -->
    <objectFactory type="com.steelzheng.app.factory.FkObjectFactory">  
        <property name="author" value="steelzheng"/>  
    </objectFactory>  
    
    <!-- default指定默认的数据库环境为mysql -->  
    <environments default="mysql">  
        <!-- 配置名为mysql（该名字任意）的环境 -->  
        <environment id="mysql">  
            <!-- 配置事务管理器，JDBC代表使用JDBC自带的事务提交或回滚 -->  
            <transactionManager type="JDBC"/>  
            <!-- database配置数据源，此处是使用Mybatis内置的数据源  -->  
            <dataSource type="POOLED">  
                <!-- 配置链接数据库的驱动、URL、用户名、密码  -->  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC"/>  
                <property name="username" value="root"/>  
                <property name="password" value="root"/>  
            </dataSource>
        </environment>
    </environments>
    
    <mappers>        
	    <!-- 配置mybatis需要加载的Mapper -->  
        <!--
        <mapper resource="com/steelzheng/app/mapper/NewsMapper.xml"/>        
        <mapper resource="com/steelzheng/app/mapper/UserMapper.xml"/>
        -->        
        <package name="com.steelzheng.app.mapper"/>  
    </mappers>
</configuration>
```

### 代码加载配置文件

```kotlin
val resource = "mybatis-config.xml"  
val inputStream = Resources.getResourceAsStream(resource)  
val sqlSessionFactory = SqlSessionFactoryBuilder().build(inputStream)  
val session: SqlSession = sqlSessionFactory.openSession()
```

通过获取到的session，就可以做相关数据库的操作了 例如：

* session.insert
* session.update
* session.delete
* session.select
* session.selectList

或者获取相关Mapper

```kotlin
fun <T> getMapper(cls: Class<T>): T {  
    return session.getMapper(cls)  
}
```

### 配置自定义对象工厂

```xml
<!-- 配置对象工厂，对象工厂的作用是用于给指定的对象封装一些属性值 -->
<objectFactory type="com.steelzheng.app.factory.FkObjectFactory">  
	<property name="author" value="steelzheng"/>  
</objectFactory>  
```

详细介绍：\[\[自定义对象工厂]]

### 配置自定义类型处理器

```XML
<typeHandlers>  
    <typeHandler handler="com.steelzheng.app.handler.NameTypeHandler" jdbcType="VARCHAR" javaType="com.steelzheng.app.domain.Name"/>  
</typeHandlers>
```

详细介绍： \[\[自定义类型处理器]]

### 配置Mapper

详细介绍：\[\[创建、配置、使用Mapper]]
