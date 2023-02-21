## Tail Call Function

### 1. Tail Call Function 定義
當一個 function 被稱為 tail call function，指的是這個function 最後一步是呼叫其他function，例如:
```kotlin
fun firstFun(){
    // do something
    return secondFun()
}
```

以下為 **非** Tail Call Function:
```kotlin
fun firstFun(){
    // do something
    return secondFun() + 1 * 2
}
```


### 2. 為何 Tail Call Function 可以避免 stack overflow
假設你有三個 function: fun1, fun2, fun3，然後是 nested 的方式呼叫，也就是
fun1 呼叫 fun2， 而 fun2 呼叫 fun3。
一般情況下，你的 memory stack frame 的結構會是:
[fun3]
[fun2]
[fun1]

先出現的 function 會先擺放，然後接著往上放。而處理 stack frame 則是由上往下處理，然而這樣的設計，卻會因為過多的 function call 而導致 stack overflow，因此才會有 tail call。

如果是 tail call，則會由於 fun2 在 fun1 整個做完後才調用，則電腦會認為沒必要存取 fun1 的 stack frame，以 func1 -> func2 -> func3 為例，
當 call func2 的動作在 func1 的對底部執行，那 func1 的 stack frame 就不需要了，
也叫表示記憶體可以釋出（因為 func1 做完了），以此類推：
```kotlin
[fun1] --> [fun2]   -->      [fun3]
           [fun1] remove     [fun2] remove
                             [fun1] remove
```
因此較不會有 stack 過度堆疊的問題

圖片可參考:

<img src="https://i.imgur.com/ybO5suT.jpg">

#### reference
1. https://ithelp.ithome.com.tw/articles/10197230
