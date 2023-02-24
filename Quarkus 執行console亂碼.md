## IntelliJ Quarkus console 中文亂碼
### 1. 問題描述
在 IntelliJ 使用 Quarkus Framework，如果 property value 存取中文字，那麼使用 ``println()``
console 會出現亂碼，但是直接使用 ``println("你好")`` console 不會有亂碼，中文字可以正確顯示。
```kotlin
data class Person(
    var name : String
)

@Path("/test")
class PersonResource{
    @Get
    @Path("people")
    fun getAllPeople() : String {
        val p = Person("大華")
        println(p.name) // p 會變成亂碼，而非顯示大華
    }
}
```
### 2. 解決方式
需要進行兩個步驟:
1. 在 Run/DeBug Configuration 中的 environment variable 添加: ``JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF-8``
<img src="https://i.stack.imgur.com/MGzSq.png">

2. 點選 IDEA 中 Help --> Edit Costom VM options，添加 ``-Dfile.encoding="UTF-8"``
<img src="https://img-blog.csdnimg.cn/2020010216495597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg0OTUxOQ==,size_16,color_FFFFFF,t_70">


#### reference
https://blog.csdn.net/weixin_43849519/article/details/103807157?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-103807157-blog-116660638.pc_relevant_aa2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-103807157-blog-116660638.pc_relevant_aa2&utm_relevant_index=6

https://stackoverflow.com/questions/70444142/chinese-characters-are-not-displayed-correctly-when-running-quarkus-directly-in