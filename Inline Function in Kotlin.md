##  Inline Function in Kotlin
### 1. High Order Function
> In Kotlin, functions are first-class citizens or high order functions, so we can pass functions around or return them just like other normal types. However, the representation of these functions at runtime sometimes may cause a few limitations or performance complications.

簡單說，就是當 function 被稱為 **high order function**，則代表此 function 的參數與回傳值可以使用 function，就好像把 function 當成一般的變數一樣。而在 kotlin中，funciton type 的寫法是
```kotlin
(variable type) -> return type
例如:
(String) -> Int
```

```kotlin
/**
 * use function type as return
 * 定義一個 function, 其回傳的type 為 function
 * */
fun returnFun() : (String) -> Int {
    return fun(s:String) : Int = s.toInt()
}


/**
 * use function type as parameter
 * 定義一個 function, 其傳入的 parameter 為 function
 * */
fun parameterFun(myFun : (String) -> Int) : Int{
    return myFun("123")
}
```
然而這樣的function，卻會衍生出一些效能問題。

### 2. Trouble in Paradise
#### 2.1  The Overhead of Lambdas in Kotlin
> Every time we declare a higher-order function, at least one instance of those special Function types will be created.

請看以下的 example:
``` kotlin
fun <T> Collection<T>.filter(predicate: (T) -> Boolean): Collection<T> = // Omitted

sampleCollection.filter { it == 1 }
```
這個時候，kotlin 實質上執行的是:
```java
filter(sampleCollection, new Function1<Integer, Boolean>() {
  @Override
  public Boolean invoke(Integer param) {
    return param == 1;
  }
});
```
我們可以發現，當我們 call 一次 filter function，則必須要在 heap 中建立一次 Function1 instance。當然，如果使用 invokedynamic 可以避開此問題，可惜的是 kotlin 是僅與 java 6 相容，而 invokedynamic 是要到 java 7 才有。
> Why does Kotlin do this instead of, say, using invokedynamic like how Java 8 does with lambdas? Simply put, Kotlin goes for Java 6 compatibility, and invokedynamic isn’t available until Java 7.

除此之外，我們還發現，為了執行 filter function ， 還需要多 call invoke function，因此僅僅是 passing a lambda to a function, 就會有以下的狀況出現:
1. At least one instance of a special type is created and stored in the heap
2. An extra method call will always happen

#### 2.2 Closures
所謂的 closure function 指的是把上一層的 variable 捕捉到自己的 function body進行使用，而被捕捉的 variable 則稱為 captured variable，因此，帶有 captured variable 的 function 稱為 closure function。

而在 kotlin 的官方文件中，closure 本身的定義是:
>  A closure is a **scope** of variables that can be accessed in the body of the function. (https://kotlinlang.org/docs/inline-functions.html)

closure 是一個 **scope** ，當我們說這個 variable 在此 function 中的 scope 是 closure scope ， 就代表這個 variable 定義在上一層，而此 function 將其捕捉到自己的 function body 中，進行使用。

``` kotlin
/*
從此例子來看，innerFun 把 sum 這個變數拉到自己的 function body 使用 ，
innerFun 為  closure (function)
*/
fun MyFun(){
    var sum = 0
    fun innerFun(){
        println("sum is $sum")
    }
    return innerFun()
}
```
而在 kotlin 中，lambda expression 是可以捕捉到上一層的 variable，也就是 lambda expression function 是 closure function。
>  a lambda expression can access its closure, that is, variables declared in the outer scope.

然而這就是問題點! 我們已經知道每當我們 call 一次 lambda function ，就要建立一個 instance，如果這個 lambda function 不包含 captured variable，那麼JVM 則會採用 singleton 方式建立 instance，換言之，即使你 call 很多次 lambda function，從頭到尾都只會有一個 instance。

但是，如果 lambda function 有 captured variable，那麼就不是採用 singleton，而是你 call 幾次 lambda function 就建立幾個 instance。這就會導致 memory 過度被占用。 

> The extra memory allocations get even worse when a lambda captures a variable: The JVM creates a function type instance on every invocation. For non-capturing lambdas, there will be only one instance, a singleton, of those function types.

>This means that each captured variable will be passed as constructor arguments, thus generating a memory overhead.

```kotlin
fun <T> Collection<T>.each(block: (T) -> Unit) {
    for (e in this) block(e)
}

fun main() {
    val numbers = listOf(1, 2, 3, 4, 5)
    val random = random()

    numbers.each { println(random * it) } // capturing the random variable
}
```

#### 2.3. Type Erasure
When it comes to generics on the JVM, it’s never been a paradise, to begin with! Anyway, Kotlin erases the generic type information at runtime. That is, an instance of a generic class doesn’t preserve its type parameters at runtime.

For example, when declaring a few collections like List<Int> or List<String>, all we have at runtime are just raw Lists. This seems unrelated to the previous issues, as promised, but we’ll see how inline functions are the common solution for both problems.

### 3. Inline Functions
在 kotlin 裡面寫 lambda 是一件非常方便的事情，但每次要傳入一個 lambda function 其實都會創造一個物件 instance，久了其實會造成效能問題，因此才會有另一種解法: inline function。
inline function 簡單來說就是把 lambda function 整個程式碼加入，而不用建立 instance，雖然不會有大量instance的出現，但會大幅增加程式碼的長度。 

```kotlin
fun main() {
	runInlineLambda("Tim") { id: Int, name: String ->
        println("result is userName: $name, his/her id is $id")
        "userName: $name, his/her id is $id"
  }
}

fun runInlineLambda(userName: String, showUserStatus: (Int, String) -> String) {
    val id = 83666
    println(showUserStatus(id, userName))
}
```
像以上的程式碼，如果打開 byte code 來看的話，會發現有呼叫 runInlineLambda 而且傳入 lambda 的地方創立了一個 instance
<img src="https://ithelp.ithome.com.tw/upload/images/20200916/20129902KHxMJK75l1.png">

但如果改成 inline function 後
```kotlin
inline fun runInlineLambda(userName: String, showUserStatus: (Int, String) -> String) {
    val id = 83666
    println(showUserStatus(id, userName))
}
```
<img src="https://ithelp.ithome.com.tw/upload/images/20200916/20129902xbi4IqDryq.png">
會發現原本 main 裡面呼叫 runInlineLambda 的程式碼消失了，進一步的他把下方 static final void runInlineLambda 內的整段程式碼都複製一份到 main 裡面了！


如此就可以減少 lambda function 造成的新建物件 instance 的問題，減少記憶體的消耗，但要注意的是因為 inlining 會造成大量的程式碼，所以如果是很大的函數，不建議使用 inline 關鍵字。

#### reference
https://www.baeldung.com/kotlin/inline-functions

https://medium.com/citycoddee/python%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-4-lambda-function-%E8%88%87-closure-%E4%B9%8B%E8%AC%8E-7a385a35e1d8

https://tw.kotlin.tips/articles/day-7-lambda-closure-inline-tail-recursion-function-