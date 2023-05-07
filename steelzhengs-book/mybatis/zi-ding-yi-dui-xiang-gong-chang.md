# 自定义对象工厂

在mybatis-config.xml中配置:

```xml
<!-- 配置对象工厂，对象工厂的作用是用于给指定的对象封装一些属性值 -->
<objectFactory type="com.steelzheng.app.factory.FkObjectFactory">
    <!-- 定义参数值 -->
	<property name="author" value="steelzheng"/>  
</objectFactory>  
```

创建自定义工厂类， 为News对象封装数据

```kotlin
package com.steelzheng.app.factory  
  
import com.steelzheng.app.domain.News  
import org.apache.ibatis.reflection.factory.DefaultObjectFactory  
import java.util.*  
  
  
class FkObjectFactory : DefaultObjectFactory() {  
  
    private val serialVersionUID = 1L  
  
    private var author: String? = null  
  
    override fun <T : Any?> create(type: Class<T>?): T {  
        val t = super.create(type)  
        return processObject(t) as T  
    }  
  
    override fun <T : Any?> create(  
        type: Class<T>?, constructorArgTypes: MutableList<Class<*>>?, constructorArgs: MutableList<Any>?  
    ): T {  
        val t = super.create(type, constructorArgTypes, constructorArgs)  
        return processObject(t) as T  
    }  

	/*
	 * 读取配置对象工厂时定义的参数值
	 */
    override fun setProperties(properties: Properties) {  
        super.setProperties(properties)  
        author = properties.getProperty("author")  
    }  

	/*
	 * 为News对象的meta字段，填充数据
	 */
    private fun <T> processObject(obj: T): T {  
        if (obj is News) {  
            obj.meta["author"] = author  
        }  
        return obj  
    }  
}
```

```kotlin
package com.steelzheng.app.domain  
import com.steelzheng.app.annotations.NoArg  
  
@NoArg  
class News (var id: Int? = null, var title: String?, var content: String?) {  
  
    var meta: MutableMap<String, Any?> = HashMap()  
        get() {  
            if (field == null) {  
                field = HashMap()  
            }  
            return field  
        }  
  
    override fun toString(): String {  
        return "News(id=$id, title=$title, content=$content, meta=$meta)"  
    }  
}
```
