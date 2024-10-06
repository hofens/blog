
如果需要向服务端 Post  JSONObject 数据，不能像这样，因为 `"{\"ext\":{\"createGroup\":1}}"` 是字符串；而目前参数只支持 Map 或者 String，所以如果想通过 String 传 JSONObject 是行不通的。

``` json
{"groupId":"65bb3404caf73403c2fa44e6","source":"{\"ext\":{\"createGroup\":1}}","serial_number":98}
```

如果参数要是 JSONObject 类型，得通过 Map 来，Map 的 Value 可以是 JSONObject

``` kotlin
val sourceMap = mutableMapOf<String, Any>().apply {  
    put("groupId", mGroupId)  
    intentData?.bizSource?.let {  
        put("source", JSONObject(it))  
    }  
}

@TcpSend("44177/11")  
fun reqEnterFamily(@FiledMap map: Map<String, Any>): TcpCall<JSONObject>
```

> "{\"ext\":{\"createGroup\": 1}}"是 String 类型，但可以通过 JSONObject (it)变成 JSONObject 类型


--- 


# EasyTcp 增加 JSONObject 传参

## 一、现状
目前 EasyTcp 只支持 string 和 map 的参数类型，导致当接口需要传 JSONObject 的时候会比较麻烦，需要通过 map 来实现，如 source 字段

```kotlin
服务端要求格式
{
    "groupId": "",
    "source": {
		"biz": "reason": "",
        "ext": {
    		"type":!	
    	}
	}
}

客户端接口定义
@TcpSend("44177/11")  
fun reqEnterFamily(@FiledMap map: Map<String, Any>): TcpCall<JSONObject>


参数构造
val sourceMap = mutableMapOf<String, Any>().apply {  
    put("groupId", mGroupId)  
    intentData?.bizSource?.let {  
        put("source", JSONObject(it))  
    }  
}


```

## 二、优化新增@FiledJson

新支持一种 JSONObject 类型传参，经测试 value 为 JSONArray、JSONObject、string 以及嵌套都支持

```kotlin

@TcpSend("44177/11")  
fun reqEnterFamily(@Filed("groupId") groupId: String, @FiledJson obj: JSONObject): TcpCall<JSONObject>


参数构造如下
val json: JSONObject = {
  "effects": [
    {
      "pc_align": 0,
      "mix_contents": [
        {
          "content": "http://cc.fp.ps.netease.com/file/6592448e0eeea9a08a835915udQBkbOe05",
          "type": "img",
          "pos": 0
        }
      ]
    }
  ],
  "effect_versions": {
    "0": [
      {
        "min": "11536"
      }
    ],
    "1001":
      {
        "min": "3.9.9"
      }
  },
  "_notify_type": "send_sub"
}

familyProtocol.reqEnterFamily(mGroupId, json)

```



## 三、实现

```kotlin
一、注解
@Target(ElementType.PARAMETER)  
@Retention(RetentionPolicy.RUNTIME)  
public @interface FiledJson {  
  
}

二、解析注解
private fun parseParameterAnnotation(type: Type, annotation: Annotation) =  
    when (annotation) {  
        ... 
        is FiledJson -> {  
            val rawParameterType = EasyTcpUtil.getRawType(type)  
            if (!JSONObject::class.java.isAssignableFrom(rawParameterType)) {  
                throw IllegalArgumentException("FiledJson 必须注解的类型为 JSONObject类型")  
            }  
            ParameterHandler.FiledJson()  
        }  
        else -> null  
    }

三、JSONObject转化成表单对象
/**  
 * JSONObject虽然有很多层，但每层都是JSONObject，不用处理嵌套对象的情况
 */
class FiledJson : ParameterHandler<JSONObject>() {  
    override fun append(builder: RealTcpRequest.Builder, values: JSONObject?) {  
        if (values != null) {  
            for (key in values.keys()) {  
                val value = values.get(key as String)  
                if (StringUtil.isNotNullOrEmpty(key)) {  
                    builder.addForm(key, value)  
                }  
            }  
        }  
    }  
}
```


## 四、作废

实际上和 Map 没什么差别，不如把 JSONObject 转成 Map 更实用

