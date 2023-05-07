# 自定义类型处理器

在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时，都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。说人话，也就是java类型与jdbc类型之间的处换器，诸如最常见StringTypeHandler

自定义类型处理器， 处理User对象的Name字段，做java和jdbc之间的处理

```kotlin
package com.steelzheng.app.domain  
  
class User {  
    var id: Int? = null  
    var name: Name? = null  
    var password: String? = null  
  
    constructor(id: Int?, name: Name?, password: String?) {  
        this.id = id  
        this.name = name  
        this.password = password  
    }  
  
    override fun toString(): String {  
        return "User(id=$id, name=$name, password=$password)"  
    }  
}
```

```kotlin
package com.steelzheng.app.domain  
  
class Name {  
    var firstName: String? = null  
    var lastName: String? = null  
  
    constructor(firstName: String?, lastName: String?) {  
        this.firstName = firstName  
        this.lastName = lastName  
    }  
  
    override fun toString(): String {  
        return "Name(firstName=$firstName, lastName=$lastName)"  
    }  
}
```

创建类型处理器，继承BaseTypeHandler，泛型传入Name(即，针对该类型的对象进行处理)

```kotlin
package com.steelzheng.app.handler  
  
import com.steelzheng.app.domain.Name  
import org.apache.ibatis.type.BaseTypeHandler  
import org.apache.ibatis.type.JdbcType  
import java.sql.CallableStatement  
import java.sql.PreparedStatement  
import java.sql.ResultSet  
  
class NameTypeHandler : BaseTypeHandler<Name>() {  

    /**  
	 *  写操作，把java类型转jdbc
	 */
    override fun setNonNullParameter(ps: PreparedStatement, i: Int, parameter: Name, jdbcType: JdbcType?) {  
        ps.setString(i, parameter.firstName + "-" + parameter.lastName)  
    }  

	/**  
	 *  读操作： 把jdbc转java类型
	 */
    override fun getNullableResult(rs: ResultSet, columnName: String): Name {  
        val content = rs.getString(columnName)  
        val tokens = content.split("-")  
        return Name(tokens[0], tokens[1])  
    }  

	/**  
	 *  读操作： 把jdbc转java类型
	 */
    override fun getNullableResult(rs: ResultSet, columnIndex: Int): Name {  
        val content = rs.getString(columnIndex)  
        val tokens = content.split("-")  
        return Name(tokens[0], tokens[1])  
    }  

	/**  
	 *  读操作： 把jdbc转java类型
	 */
    override fun getNullableResult(cs: CallableStatement, columnIndex: Int): Name {  
        val content = cs.getString(columnIndex)  
        val tokens = content.split("-")  
        return Name(tokens[0], tokens[1])  
    }  
}
```

在mybatis-config.xml中定义：

```XML
<typeHandlers>  
    <typeHandler handler="com.steelzheng.app.handler.NameTypeHandler" jdbcType="VARCHAR" javaType="com.steelzheng.app.domain.Name"/>  
</typeHandlers>
```
