## Any, Nothing, Unit, Object in Kotlin 差異

### 1. Unit 
- Unit object : Unit is an object which means it is a single static instance， 請看 kotlin 官方文件如何定義 Unit class:

```kotlin
/**
 * The type with only one value: the `Unit` object. This type corresponds to the `void` type in Java.
 */
public object Unit {
    override fun toString() = "Kotlin.Unit"
}
```
基本上，Unit 就相當於 java 的 void type，但是! kotlin **實質上** 還是回傳了一個 Unit object instance，所以你調用 ``toString()`` 時，會得到 kotlin.Unit。

```kotlin
fun speak() : Unit {
    // do something
}

fun speak1() {
    // do something
}

val mySpeak = speak()
val mySpeak1 = speak1() 
mySpeak.toString() 
mySpeak1.toString() 

```
### 2. Nothing
相較於 Unit，Nothing class 是沒有任何 instance，且由於 constructor 是 private，因此也無法實例化。 Nothing 是**真的**代表沒有回傳東西，例如: 必然會 throw exception 的 method。
> It is used to define a function return type which is going to throw an exception every time. 

以下為 kotlin 官方文件，
```kotlin
/**
 * Nothing has no instances. You can use Nothing to represent "a value that never exists": for example,
 * if a function has the return type of Nothing, it means that it never returns (always throws an exception).
 */
public class Nothing private constructor()
```
使用 Nothing in method，
```kotlin
// 錯誤的寫法:
fun iWillAlwaysThrowException() =  throw Exception("Unnecessary Exception")

//The ebove function will show a compilation error 'Nothing' return type needs to be specified explicitely

// 正確的法寫:
fun iWillAlwaysThrowException() : Nothing =  throw Exception("Unnecessary Exception")
```
### 3. Any
Any class 是所有 Kotlin class 的 root，也就是所有class 的 superclass。
```kotlin
/**
 * The root of the Kotlin class hierarchy. Every Kotlin class has [Any] as a superclass.
 */
public open class Any {
    public open operator fun equals(other: Any?): Boolean
    public open fun hashCode(): Int
    public open fun toString(): String
}
```

#### reference
https://agrawalsuneet.github.io/blogs/difference-between-any-unit-and-nothing-kotlin/