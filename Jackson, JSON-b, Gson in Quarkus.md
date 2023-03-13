## 序列化 (Serialize) 與反序列化 (Deserialize) 

### 1. 定義
- JSON: JavaScript Object Notation (JSON) is a standard text-based format for representing structured data based on JavaScript object syntax. JSON 是一種以 **text** 為基底，以來儲存或傳遞資料的格式。
```js
{
  "firstName": "John",
  "lastName": "Smith",
  "isAlive": true,
  "age": 27,
  "address": {
    "streetAddress": "21 2nd Street",
    "city": "New York",
    "state": "NY",
    "postalCode": "10021-3100"
  },
}
```
- Serialize 序列化 與 Deserialize 反序列化: 序列化一般指的是將一個資料結構或一個 object 轉換成可以**傳遞或儲存**的格式，例如把object 轉換成 bytes，而在 JSON Serialize，這裡指的是把一個資料結構或 object 轉換成 JSON 格式，流程上是先把 object 轉成 string 再將 string 轉成 byte[]，然而這樣的轉換方式卻會大幅增加儲存空間，以下簡單舉個例子:
```java
class PP { 

long userId = 102333320132133L; // long 佔 8 bytes

int passportNumber = 123456; // int 佔 4 bytes

}
```
要存取 PP instance，基本上要 12 bytes，但是如果我們把上述資料轉成 JSON 格式 (string) :
(以 ASCII 編碼，一個 character 會使用 1 bytes)
```js
{ // 佔 1 bytes

"userId":102333320132133, // 佔 25 bytes

"passportNumber":123456 // 佔 23 bytes

} // 佔 1 bytes
```
可以發現直接從 12 bytes 變成 50 bytes (or 50 bytes + 1 null terminator in C/C+)，需要的空間直接放大4倍以上，不過這邊不針對這個細談，詳請可見 <a href="https://zhuanlan.zhihu.com/p/48045115"> 參考</a>; 反序列化則是指把 JSON 格式轉回資料結構或object。

<img src="https://www.scientecheasy.com/wp-content/uploads/2021/07/java-serialization-deserialization-process.png">


### 2. 於 Kotlin 使用 Serialize  與 Deserialize
關於 Kotlin 相關的 Serialize  與 Deserialize 有非常多的插件可以使用，包含:
GSON、Jackson、JSON-b 與 Kotlinx Serialization 等等，有文章也針對這幾個提出效能比較，可看 <a href="https://apiumhub.com/tech-blog-barcelona/json-in-kotlin-comparing-options/#Kotlinx_Serialization"> 效能比較 </a> ，另外，也有人針對功能多寡進行比較，可看 <a href="https://itsallbinary.com/jackson-vs-gson-vs-json-b-vs-json-p-vs-org-json-vs-jsonpath-java-json-libraries-features-comparison/"> 功能比較 </a>。
這裡我們針對 GSON、Jackson 與 JSON-b 泛用性較高的進行簡單的操作說明。

#### 2.1 Gson
- **2.1.1 Serialize**: Object to JSON 
```kotlin
    val builder = GsonBuilder().serializeNulls().create()
     builder.toJson(paintings)  // paintings is the instance of House class
```
- **2.1.2 Deserialize**: JSON to Object
```kotlin
    val builder = GsonBuilder().serializeNulls().create()
     builder.fromJson(JsonString)  // JsonString is the String instance with json format
```

- **2.1.3 Modifying properties when Serializing / Deserializing**
```kotlin
class Paintings(
    @SerializedName("畫作名稱")
    var title : String,
    var price : Double,
    @Transient
    var desc : String
) {
}
```


#### 2.2 JSON-b
- **2.2.1 Serialize**: Object to JSON 
```kotlin
val builder = JsonbBuilder.create()
builder.toJson(houses) // house is the instance of House class
```
- **2.2.2 Deserialize**: JSON to Object
```kotlin
val builder = JsonbBuilder.create()
builder.fromJson(JsonString) // JsonString is the String instance with json format
```

- **2.2.3 Modifying properties when Serializing / Deserializing**
```kotlin
class House (
    // 注意! @JsonbProperty 在這裡是無效的，因為getter / setter 並沒有加上 @JsonbProperty
    // 所以當你換成 json 調用 getter 基本上是沒有用的!
    // 你要寫成  @get:Jsonbproperty
    @JsonbProperty(value = "myAddress")
    var address : String,
    var price : Double,
    @JsonbTransient
    var desc : String
){
    // 這裡的JsonbProperty就是直接加在 getter / setter上
    @JsonbProperty(value = "Desc_OKAY")
    var anotherDesc : String? = null
}
```

#### 2.3 Jackson
- **2.3.1 Serialize**: Object to JSON 
```kotlin
private val mapper = ObjectMapper()
mapper.writeValueAsString(school) // school is the instance of School class
```
- **2.3.2 Deserialize**: JSON to Object
```kotlin
private val mapper = ObjectMapper()
mapper.readValue(JsonString, School::class.java) // JsonString is the String instance with json format
```
- **2.3.3 Modifying properties when Serializing / Deserializing**
先針對寫在 **primaey constructor 的 properties**
    1. ``@JsonProperty(Myname)`` : 寫在 getter 代表當 class 要轉成 JSON 時，這個 property 在 JSON 的 key 值會是 Myname; 寫在 setter 代表當 JSON 要轉成 class 時，property 的值會是從 JSON key = Myname 取得。簡單說，這個是用來更改 property name 的 annotation，當資料庫的column 名稱與 class property 名稱不同時可以使用。

    2. ``@get:JsonInclude(JsonInclude.Include.NON_NULL)`` : 寫在 getter 代表當 class 要轉成 JSON 時，如果這個 property 是 Null，則不顯示在 JSON。

    3. ``@JsonDeserialize(using = ObjectIdDeserializer::class)``: 使用字定義的 Deserializer
```kotlin
class School (
    @JsonProperty("_id")
    @get:JsonInclude(JsonInclude.Include.NON_NULL)
    @JsonDeserialize(using = ObjectIdDeserializer::class)
    var id : ObjectId? = null,

    @JsonProperty("name")
    var name : String? = null,

    @JsonProperty("score")
    var score : String? = null,
    ){
}
```
值得注意的是先針對寫在 **class {}** 裡面的 ``@JsonInclude`` 不需要寫 get
```kotlin
class School (
    ){
    @JsonProperty("_id")
    @JsonInclude(JsonInclude.Include.NON_NULL)
    @JsonDeserialize(using = ObjectIdDeserializer::class)
    var id : ObjectId? = null

    @JsonProperty("name")
    var name : String? = null

    @JsonProperty("score")
    var score : String? = null
}
```




- **2.3.4 Customer Deserializing**
<a href="https://stackoverflow.com/questions/14012890/jackson-json-deserialization-of-mongodb-objectid" >如何對 mongoDB 的 ObjectId 進行 Deserialize?</a> 也就是把 
```js
{   "_id": {"$oid": "5489f420c8306b6ac8d33897"},
    "name": "小名"
}
```
轉成
```kotlin
class Person(
    var _id : ObjectId? = null,
    var name: String? = null
){}
```
問題出在 Jackson 無法自動 Deserialize ObjectId，即Jackson 不知道要把 ``$oid``的值取出來使用，因此這部分需要我們自訂 Deserializer。

```kotlin
import com.fasterxml.jackson.core.JsonParser
import com.fasterxml.jackson.databind.DeserializationContext
import com.fasterxml.jackson.databind.JsonDeserializer
import com.fasterxml.jackson.databind.JsonNode
import org.bson.types.ObjectId

/**
@author Yu-Jing
@create 2023-03-10-下午 03:39
 */
class ObjectIdDeserializer : JsonDeserializer<ObjectId>() {
    override fun deserialize(p: JsonParser?, ctxt: DeserializationContext?): ObjectId {
        // 把 p 轉成
        val oid: JsonNode = (p!!.readValueAsTree() as JsonNode).get("\$oid")
        return ObjectId(oid.asText())
    }
}
// ObjectId =  {"$oid": "5489f420c8306b6ac8d33897"} 
// 換成  
// ObjectId =  "5489f420c8306b6ac8d33897" 
```
最後再補上 annotation
```kotlin
@JsonDeserialize(using = ObjectIdDeserializer::class)
    var id : ObjectId? = null
```

- **2.3.5 Customer Serializing** : 同樣的問題，Jackson 不會自動轉換 ObjectId 成 Json，這樣會導致你在前台看到:
```js
{ "timestamp": 1678331132, "date": "2023-03-09T03:05:32.000+00:00" }
```
但請注意，這個時間是**當下**的時間，而**不是**把資料庫的 objectId 轉換成 timestamp!!! 換言之，Jackson 根本沒有把資料庫的 objectId 進行處理，只是把當前時間扔進來罷了。因此，我們需要自定義 Serializer，手法類似上面，不再贅述。

``` kotlin
// 自行將 ObjectId 轉換成你想要的 String 樣式，例如: ObjectId: "5489f420c8306b6ac8d33897@@@@@"，
// 然後把改好的String 寫進 j.writeString(...)
import com.fasterxml.jackson.core.JsonGenerator
import com.fasterxml.jackson.databind.JsonSerializer
import com.fasterxml.jackson.databind.SerializerProvider
import org.bson.types.ObjectId

/**
@author Yu-Jing
@create 2023-03-13-下午 02:06
 */
class ObjectIdSerializer : JsonSerializer<ObjectId>() {
    override fun serialize(value: ObjectId?, gen: JsonGenerator?, serializers: SerializerProvider?) {
        if (value == null){
            gen?.writeNull()
        }else{
            gen?.writeString(value.toString())
        }
    }
}
```
```kotlin
@JsonSerialize(using = ObjectIdSerializer::class)
var id : ObjectId? = null,
```
另外，如果對 Jackson 的原理有興趣可參考 <a href="https://www.baeldung.com/jackson-object-mapper-tutorial" >Jackson JsonNode 相關介紹 </a>
簡單說，Jackson 會把 json 以 JsonNode 形式存取，因此你可以
```java
String json = "{ \"color\" : \"Black\", \"type\" : \"FIAT\" }";
JsonNode jsonNode = objectMapper.readTree(json);
String color = jsonNode.get("color").asText(); 
// Output: color -> Black
```

### Reference
https://stackoverflow.com/questions/3316762/what-is-deserialize-and-serialize-in-json
https://zhuanlan.zhihu.com/p/48045115
https://www.quora.com/Are-Java-strings-null-terminated
https://mkyong.com/java/how-to-convert-java-object-to-from-json-jackson/
https://proandroiddev.com/parsing-optional-values-with-jackson-and-kotlin-36f6f63868ef
