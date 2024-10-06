

```kotlin
JsonModel.parseObject (response.optJSONObject ("data"), FamilyOperateItem:: class. java)
```

由于项目中的 JsonModel 解析方法是通过反射实现的，在生成对象时用的是无参构造，所以如果在有参构造函数中去为属性赋值并不会执行


JsonModel 解析对象的区别
一、Bean
这种写法会给属性赋初始值
```kotlin

class FamilyOperateItem: JsonModel() {  
	val type: String = DEFAULT  
	val name: String = ""  
	val icon: String = ""  
	val jumpUrl: String = ""  
	val openType: Int = 2  
	val browserRatio: Double = 1.6
}
```



二、Param
用 JsonModel 解析的时候，这样写也不会给 FamilyOperateItem 对象赋初始值，而且 JsonModel 会给数据赋值 null
```kotlin

class FamilyOperateItem(  
    val type: String = DEFAULT,  
    val name: String,  
    val icon: String,  
    val jumpUrl: String,  
    val openType: Int = 2,  
    val browserRatio: Double = 1.6  
): JsonModel()

```


如果用 JsonModel 解析服务端数据，可以用写法 1；如果用于自己创建对象，用写法 2。