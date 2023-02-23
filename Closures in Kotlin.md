## Closures in Kotlin

### 1. What is Closure
>  A closure is a scope of variables that can be accessed in the body of the function. (https://kotlinlang.org/docs/inline-functions.html)

closure 是一個 **scope** ，當我們說這個 variable 在這個 scope ， 就代表這個 variable 可以在這個 function body 被獲取到。
```kotlin
var sum = 0
fun myFun(){
    sum ++
    println(sum)
}
```
如此，我們可以說 **The variable, ``sum``, is captured in the closure**。

#### Reference
https://www.geeksforgeeks.org/closures-in-kotlin/