# Gson

## 1.0 基本使用

```kotlin

package com.baima.jetpack

import android.app.Activity
import android.content.Intent
import android.content.Intent.CATEGORY_DEFAULT
import android.os.Build
import android.os.Bundle
import android.util.JsonReader
import android.util.JsonToken
import android.util.Log
import androidx.annotation.RequiresApi
import com.google.gson.GsonBuilder
import com.google.gson.reflect.TypeToken
import kotlinx.android.synthetic.main.activity_main.*
import java.io.StringReader

class MainActivity : Activity() {

    @RequiresApi(Build.VERSION_CODES.M)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button1.setOnClickListener {
            val pushIntent = Intent()
            pushIntent.addCategory(CATEGORY_DEFAULT)
            pushIntent.setAction("com.baima.service")
            startActivity(pushIntent)
        }

        //Object 2 Json
        val employee = Employee(1,"xia","rupeng","xrpcdut@163")
        val gson = GsonBuilder().setPrettyPrinting().create()
        val jsonString = gson.toJson(employee)
        Log.e("xia",jsonString)

        //Json 2 Object
        val jsonEmployee = "{\"email\":\"xrpcdut@163\",\"firstName\":\"xia\",\"id\":1,\"last\":\"rupeng\"}"
        val employee1 = gson.fromJson(jsonEmployee,Employee::class.java)
        Log.e("xia",employee1.toString())

        //Json Array 2 Object[]
        val jsonArray = "[{'name': 'Alex','id': 1}, {'name': 'Brian','id':2}, {'name': 'Charles','id': 3}]"
        val userArray = gson.fromJson(jsonArray,Array<User>::class.java)
        for (user in userArray) {
            Log.e("xia",user.toString())
        }

        //Json Array 2 ArrayList<Object>
        val userListType = object : TypeToken<ArrayList<User>>() {}.type
        val list = gson.fromJson<ArrayList<User>>(jsonArray,userListType)
        for (user in list) {
            Log.e("xia",user.toString())
        }

        val departmentJson = ("{'id' : 1, "
                + "'name': 'HR',"
                + "'users' : ["
                + "{'name': 'Alex','id': 1}, "
                + "{'name': 'Brian','id':2}, "
                + "{'name': 'Charles','id': 3}]}")
        val deparment = gson.fromJson(departmentJson,Deparment::class.java)
        Log.e("xia",deparment.toString())

	}
}

data class Employee(val id: Int, val firstName: String, val last: String, val email: String) {}

data class User(val id: Long, val name: String)

data class Deparment(val id:Long,val name:String,val users:List<User>)
```

### 1.1 JsonReader

- JsonReader是流式JSON解析器，也是pull parser的示例。pull parser解析JSON令牌并将其推送到事件处理程序中。
- 它有助于读取JSON（RFC 7159）编码值作为令牌流。
- 它读取字面值（字符串，数字，布尔值和null）以及对象和数组的开始和结束定界符。
- 令牌以深度优先顺序遍历，与JSON文档中出现的顺序相同。

### 1.1.1 示例

```kotlin

        val json = "{'id': 1001,'firstName': 'Lokesh','lastName': 'Gupta','email': null}"
        val jsonReader = JsonReader(StringReader(json))
        jsonReader.isLenient = true
        while (jsonReader.hasNext()){
            val nextToken = jsonReader.peek()
            if(JsonToken.BEGIN_OBJECT.equals(nextToken)){
                jsonReader.beginObject()
            }else if(JsonToken.NAME.equals(nextToken)){
                val name = jsonReader.nextName()
                Log.e("xia","Token Key >>>>  $name")
            }else if(JsonToken.STRING.equals(nextToken)){
                val str = jsonReader.nextString()
                Log.e("xia","Token Value >>>>  $str")
            }else if(JsonToken.NUMBER.equals(nextToken)){
                val number = jsonReader.nextLong()
                Log.e("xia","Token Value >>>>  $number")
            }else if(JsonToken.NULL.equals(nextToken)){
                jsonReader.nextNull()
                Log.e("xia","Token Value >>>>  null")
            }else if(JsonToken.END_OBJECT.equals(nextToken)){
                jsonReader.endObject()
            }
        }

```

- JsonReader 深度优先遍历 与Json出现顺序相同
- 使用beginArray（）和endArray（）方法检查数组的左括号和右括号“ [”和“]”。使用beginObject（）和endObject（）方法检查对象的左括号和右括号“ {”和“}”。
- 当遇到未知名称时，严格的解析器应该失败，并带有异常。宽大的解析器应调用skipValue（）以递归地跳过该值的嵌套令牌，否则可能会发生冲突。


### 1.2 JsonParser

#### 1.2.1 转化Json

JsonParser类提供3种方法来提供JSON作为源并将其解析为JsonElements树。

- JsonElement parse（JsonReader json）–使用JsonReader读取JSON作为令牌流，并从JSON流中返回下一个值作为分析树。
- JsonElement parse（java.io.Reader json）–使用指定的阅读器读取JSON并将JSON字符串解析为解析树。
- JsonElement parse（java.lang.String json）–将指定的JSON字符串解析为解析树。

如果指定的文本不是有效的JSON，则这三个方法都将抛出JsonParseException和JsonSyntaxException。

#### 1.2.2  JsonElement, JsonObject 和JsonArray
在JsonElement树中解析了JSON字符串后，我们就可以使用它的各种方法来访问JSON数据元素。

例如，使用一种类型检查方法找出它代表什么类型的JSON元素：

```kotlin
jsonElement.isJsonObject();
jsonElement.isJsonArray();
jsonElement.isJsonNull();
jsonElement.isJsonPrimitive();
```

我们可以使用相应的方法将JsonElement转换为JsonObject和JsonArray：

```kotlin
JsonObject jsonObject = jsonElement.getAsJsonObject();
JsonArray jsonArray = jsonElement.getAsJsonArray();
```
一旦有了JsonObject或JsonArray实例，就可以使用其get()方法从中提取字段。

#### 1.2.3 使用示例

```kotlin 
        val jsonString = ("{'id': 1001, "
                + "'firstName': 'Lokesh',"
                + "'lastName': 'Gupta',"
                + "'email': 'howtodoinjava@gmail.com'}")
        
        val jsonElement = JsonParser.parseString(json)
        val jsonObject = jsonElement.asJsonObject

        println(jsonObject.get("id"))
        println(jsonObject["firstName"])
        println(jsonObject["lastName"])
        println(jsonObject["email"])

```

### 1.3 自定义序列化、反序列化

#### 1.3.1 自定义序列化

```kotlin
class BooleanSerializer :JsonSerializer<Boolean>{
    override fun serialize(
        src: Boolean?,
        typeOfSrc: Type?,
        context: JsonSerializationContext?
    ): JsonElement {
        src?.let {
            return if (src) {
                JsonPrimitive(1)
            } else {
                JsonPrimitive(0)
            }
        }
        return JsonPrimitive(0)
    }
}

...
  val user1 = User1(100L, "json", true)
        val gson1 = GsonBuilder()
            .registerTypeAdapter(object : TypeToken<Boolean>() {}.type, BooleanSerializer())
            .setPrettyPrinting()
            .create()
        Log.e("xia", gson1.toJson(user1))

...

	{
      "active": 1,
      "id": 100,
      "name": "json"
    }

```

#### 1.3.1 自定义反列化

```kotlin
data class Employee1(val id: Int, val firstName: String, val lastName: String, val email: String,val localDa:LocalDate)

class EmployeeDeserializer :JsonDeserializer<Employee1>{
    @RequiresApi(Build.VERSION_CODES.O)
    override fun deserialize(
        json: JsonElement?,
        typeOfT: Type?,
        context: JsonDeserializationContext?
    ): Employee1 {
        val jsonObject = json!!.asJsonObject
        val localDate = LocalDate.of(
            jsonObject.get("year").asInt,
            jsonObject.get("month").asInt,
            jsonObject.get("day").asInt
        )
        return Employee1(
            jsonObject.get("id").asInt,
            jsonObject.get("firstName").asString,
            jsonObject.get("lastName").asString,
            jsonObject.get("email").asString,
            localDate
        )
    }
}

...

 val json2 = ("{'id': 1001,"
                + "'firstName': 'Lokesh',"
                + "'lastName': 'Gupta',"
                + "'email': 'howtodoinjava@gmail.com', "
                + "'day': 11, "
                + "'month': 8, "
                + "'year': 2019}")

        val gson2 = GsonBuilder()
            .registerTypeAdapter(Employee1::class.java, EmployeeDeserializer())
            .create()
        val employee2 = gson2.fromJson(json2, Employee1::class.java)
        Log.e("xia",employee2.toString())

```