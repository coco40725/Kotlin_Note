## Object and Class Declaration in Kotlin

### 1. class 
當我們在 kotlin 定義一個 class，如下:
```kotlin
class Person(var name : String, var age : Int){
    companion object {
        salary : Double = 0.0
        fun breath(){
            printlin("Person必然呼吸")
        }
    }


    fun speak(word : String){
        printlin("Person 說: $word")
    }
}
```
基本上定義在 companion object 中的 member ，相當於是 static memeber，
> the members of the companion object can be accessed as static members of the class.

因此，我們可以調用:
```kotlin
// 可以直接用 class 調用
println(Person.salary)
Person.breath()

// 須透過 instance 調用
val p  = Person()
p.speak("早安")
```

### 2. object
> an object instead represents a single static instance, and can never have any more or any less than this one instance. (因此無法再實例化)

透過 object 定義的 function 與 property 都是 static ， 換言之類似於定義一個 singleton class 物件，但所有 function 與 property 都是寫在``companion object`` 裡面。

```kotlin
object Person(var name : String, var age : Int){
    salary : Double = 0.0
    fun breath(){
            printlin("Person必然呼吸")
    }
}
```
因此，我們可以直接調用:
```kotlin
// 可以直接用 class 調用
println(Person.salary)
Person.breath()

// 無法實例化
val p = Person() // error
```


#### reference
https://www.baeldung.com/kotlin/objects