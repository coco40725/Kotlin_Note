## MongoDB CRUD in Quarkus
### 1. application.properties
- 同源網域: 對 DOM 的同源政策來說，只要 Scheme、Domain 跟 Port 一致的資源，就會被視為同源。
<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*afZO2ypaTM7LtEbQ6YJh8g.png">

本次的前後端是非同源網域，前端用 port 8080，後端用 port 8081 (甚至之後用 microservices 也是如此)，因此會有跨網域求資源的需求，也就是 CORS (Cross-Origin Resource Sharing)。而在 Quarkus 我們可以設定哪些來源的、Header 甚至是 method 允許資源共享。

```properties
quarkus.mongodb.connection-string = mongodb://localhost:27017
quarkus.mongodb.database = quarkusEduDB
quarkus.http.port=8081
quarkus.http.cors=true
quarkus.http.cors.headers=accept, authorization, content-type, x-requested-with
quarkus.http.cors.origins=http://127.0.0.1:8080
quarkus.http.cors.methods=GET,PUT,PATCH,POST,DELETE
```

### 2. Dependency

### 3. MondoDB CRUD 基本架構
#### 3.1 Entity
```kotlin
import com.edu.utils.ObjectIdDeserializer
import com.edu.utils.ObjectIdSerializer
import com.fasterxml.jackson.annotation.JsonInclude
import com.fasterxml.jackson.annotation.JsonProperty
import com.fasterxml.jackson.databind.annotation.JsonDeserialize
import com.fasterxml.jackson.databind.annotation.JsonSerialize


import org.bson.types.ObjectId
/**
@author Yu-Jing
@create 2023-03-09-上午 11:14
 */
class School (
    @JsonProperty("_id")
    @get:JsonInclude(JsonInclude.Include.NON_NULL) // 如果property是 NULL，則 class 轉 json 時該 property 不顯示在 JSON
    @JsonDeserialize(using = ObjectIdDeserializer::class)
    @JsonSerialize(using = ObjectIdSerializer::class)
    var id : ObjectId? = null,

    @JsonProperty("name")
    var name : String? = null,

    @JsonProperty("score")
    var score : String? = null,
    ){
    override fun toString(): String {
        return "School(id=$id, name=$name, score=$score)"
    }
}
```
#### 3.2 Repository 
```kotlin
import com.edu.entities.School

/**
@author Yu-Jing
@create 2023-03-09-上午 11:32
 */
interface SchoolRepositories {
    fun addSchool(school : School) : School
    fun deleteSchoolById(id : String)
    fun updateSchoolById(id : String, school : School) : School
    fun findAllSchools() : List<School>?
    fun findSchoolById(id : String) : School?
}
```

這裡要注意一個點，要 inject MongoClient 時， **不要** 寫成
```kotlin
class SchoolRepositoriesImp() : SchoolRepositories {
@Inject 
 private lateinit var mongoClient : MongoClient 
}
```
這樣會出現 MongoClient 尚未初始化這個錯誤，可能是因為 SchoolRepositoriesImp 實例建好了 但  MongoClient  卻還沒注入，所導致失敗，因此還是建議寫@Inject constructor 寫法，已保證當實例建立起來時，inject 也一併完成。
<a href="https://github.com/quarkusio/quarkus/issues/6264"> 開發者也是寫 constructor 的寫法 </a>


```kotlin
import com.edu.entities.School
import com.edu.repositories.SchoolRepositories
import com.fasterxml.jackson.databind.ObjectMapper
import com.mongodb.client.MongoClient
import com.mongodb.client.MongoCollection
import com.mongodb.client.model.Filters
import com.mongodb.client.model.Updates
import org.bson.Document
import org.bson.conversions.Bson
import org.bson.types.ObjectId
import javax.enterprise.context.ApplicationScoped
import javax.inject.Inject



/**
@author Yu-Jing
@create 2023-03-09-上午 11:41
 */
@ApplicationScoped
class SchoolRepositoriesImp @Inject constructor(
    // 1. 得到 MongoClient
    private  val  mongoClient : MongoClient
) : SchoolRepositories {

    // (optional) 建立 JsonbBuilder，快速轉換 document 與 class
    private val mapper = ObjectMapper()

    // 2. 使用MongoClient --> 得到 database --> 得到 collection
    private val mongoCollection : MongoCollection<Document> = mongoClient.getDatabase("quarkusEduDB").getCollection("schools")
    override fun addSchool(school: School): School {
        // 方法一: 使用反射將 school 的資料一個一個map 到  document
        // 透過反射得到當前 class其所有的 property
        // p.value: 取得 property 的名字
        // p.getter.call(school): school 去 call p 的 getter method
        // val document  = Document()
//        school::class.declaredMemberProperties.forEach{p ->
//            document.append(p.name, p.getter.call(school))
//        }

        // 方法二: 直接用 jackson 將 school 變成 json 再變成 document
        println(school)
        mongoCollection.insertOne(Document.parse(mapper.writeValueAsString(school)))
        return school
    }

    override fun deleteSchoolById(id : String) {
        val filter = Filters.eq("_id", ObjectId(id))
        mongoCollection.deleteOne(filter)
    }

    override fun updateSchoolById(id : String, school: School): School {
        val filter = Filters.eq("_id", ObjectId(id))
        val updates = mutableListOf<Bson>()
        Document.parse(mapper.writeValueAsString(school)).forEach{(key, value) ->
            if (value != null) updates.add(Updates.set(key, value))
        }
        mongoCollection.findOneAndUpdate(filter, Updates.combine(updates))
        return school
    }

    override fun findAllSchools(): List<School>? {
        val schools = mutableListOf<School>()
        mongoCollection.find().forEach { document ->
            schools.add(mapper.readValue(document.toJson(), School::class.java))
        }
        return schools
    }

    override fun findSchoolById(id: String): School? {
        val filter = Filters.eq("_id", ObjectId(id))
        val document : Document? = mongoCollection.find(filter).first()
        println(document)
        if (document != null) {
            mapper.readValue<School>(document.toJson(), School::class.java)
            return mapper.readValue<School>(document.toJson(), School::class.java)
        }
         return null
    }
}
```
#### 3.3 Services
```kotlin
import com.edu.entities.School
import com.edu.repositories.SchoolRepositories
import javax.enterprise.context.ApplicationScoped
import javax.inject.Inject

/**
@author Yu-Jing
@create 2023-03-09-下午 01:08
 */
@ApplicationScoped
class SchoolServices @Inject constructor(
   private val schoolRepositories : SchoolRepositories
) {
    fun addSchool(school : School) = schoolRepositories.addSchool(school)
    fun deleteSchoolById(id : String) = schoolRepositories.deleteSchoolById(id)
    fun updateSchoolById(id : String, school : School) = schoolRepositories.updateSchoolById(id, school)
    fun findAllSchools() = schoolRepositories.findAllSchools()
    fun findSchoolById(id : String) = schoolRepositories.findSchoolById(id)
}
```
#### 3.4 Controller
```kotlin

import com.edu.entities.School
import com.edu.services.SchoolServices
import javax.inject.Inject
import javax.ws.rs.Consumes
import javax.ws.rs.DELETE
import javax.ws.rs.GET
import javax.ws.rs.PATCH
import javax.ws.rs.POST
import javax.ws.rs.Path
import javax.ws.rs.PathParam
import javax.ws.rs.Produces
import javax.ws.rs.core.MediaType

/**
@author Yu-Jing
@create 2023-03-09-下午 01:13
 */
@Path("/schools")
class SchoolControllers @Inject constructor(
    private val schoolServices : SchoolServices
) {
    @POST
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    fun addSchool(school : School) = schoolServices.addSchool(school)

    @DELETE
    @Path("/{id}")
    fun deleteSchool(@PathParam("id") id : String) = schoolServices.deleteSchoolById(id)

    @PATCH
    @Path("/{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    fun updateSchool(@PathParam("id") id : String, school: School) = schoolServices.updateSchoolById(id, school)

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    fun findSchools() = schoolServices.findAllSchools()

    @GET
    @Path("/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    fun findSchool(@PathParam("id") id : String) = schoolServices.findSchoolById(id)

}
```

#### Reference
https://www.knowledgefactory.net/2021/10/quarkus-vuejs-mongodb-crud-example.html
https://medium.com/%E7%A8%8B%E5%BC%8F%E7%8C%BF%E5%90%83%E9%A6%99%E8%95%89/same-origin-policy-%E5%90%8C%E6%BA%90%E6%94%BF%E7%AD%96-%E4%B8%80%E5%88%87%E5%AE%89%E5%85%A8%E7%9A%84%E5%9F%BA%E7%A4%8E-36432565a226
https://medium.com/starbugs/%E5%BC%84%E6%87%82%E5%90%8C%E6%BA%90%E6%94%BF%E7%AD%96-same-origin-policy-%E8%88%87%E8%B7%A8%E7%B6%B2%E5%9F%9F-cors-e2e5c1a53a19