## Qualifirer in Quarkus
### 1. 定義
Qualifiers are annotations that help the container to **distinguish beans that implement the same type**. As we already said a bean is assignable to an injection point if it has all the required qualifiers. If you declare no qualifier at an injection point the @Default qualifier is assumed.

**@Qualifirer**: 用於使用者自定義 annotation (It is a user-defined annotation)，當有多個 class 同時實作一個 interface時，@Inject 會不知道要inject 哪個 class，因此需要添加自定義的 annotation 來指定要哪個 class，以下為步驟:
1. 針對每個 implement class 個別建立其自己的 annotation
2. 在對應的 implement class 加上剛建立的 annotation
3. 在 @Inject 的 data，再添加一個 annotation

### 2. 案例1: Student and Teacher 都實作 Person
- **@Retention()** : 編譯器如何處理annotation:
                - SOURCE: 編譯器處理完Annotation資訊後就沒事了;
                - CLASS: 編譯器將Annotation儲存於class檔中，預設;
                - RUNTIME: 編譯器將Annotation儲存於class檔中，可由VM讀入
- **@Target()**: annotation 可以修飾哪些 type

```kotlin
interface Person {
    fun speak(word : String)
}
```

```kotlin
@ApplicationScoped
@StudentAnno
class Student : Person{
    override fun speak(word: String) {
        print("學生說 $word")
    }
}

@Qualifier
@Retention(AnnotationRetention.RUNTIME) 
@Target(AnnotationTarget.FIELD, AnnotationTarget.TYPE, AnnotationTarget.FUNCTION, AnnotationTarget.CLASS)
annotation class StudentAnno()
```

```kotlin
@ApplicationScoped
@TeacherAnno
class Teacher : Person{
    override fun speak(word: String) {
        print("老師說: $word")
    }
}

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FIELD, AnnotationTarget.TYPE, AnnotationTarget.FUNCTION, AnnotationTarget.CLASS)
annotation class TeacherAnno()
```
接著，我在 school class 調用 Person 的實作類，我們可以使用 @StudentAnno 或 @TeacherAnno 來對 person 進行注入。

```kotlin
@Path("/")
class School {
    @Inject
    @StudentAnno // 相當於 person : Student = Student()
    lateinit var person : Person


    @GET
    @Path("/speak/{word}")
    fun speakWord(@PathParam("word") word : String){
        person.speak(word)
    }
}
```

### 3. 案例2: Event 註冊
1. ``Event<target class>`` :
    target class 指的是觸發事件的 class

2. addBookEvent.fire(book) :
    代表 book 觸發了 addBookEvent, book 相當於 js 的事件 target

3. 值得注意的是 event 處理流程:
    1. 當執行 addBookEvent.fire(book)時，這邊的程式碼會先 "暫停不往下執行"，優先尋找哪個 function 有觀察 (observe) Book class (例如: fun addBook(@Observes book : Book))
    2. 找到那個 function ， 並且執行
    3. 執行完畢後，再將暫停的程式碼繼續
   由此可知，CDI Event 的流程是 "同步" 的

4. 多個 function 在觀察 Book，例如: 
```kotlin
 fun addBook(@Observes  book : Book){ //  observes any event typed with Book. The annotated parameter is called the event parameter
        println("庫存加一本書 $book")
        inventory.add(book)
    }

    fun deleteBook(@Observes  book: Book){
        println("庫存刪掉書 $book")
        // do something
    }
```
， 則當執行 addBookEvent.fire(book)時，會變成所有 function 都執行，(有一定順序但規律我不確定)。如果說你希望 addBookEvent.fire(book) 只執行特定某個方法，那麼我們需要使用 @Qualifer

```kotlin
@ApplicationScoped
class BookService {

    @Inject
    @Added
    private lateinit var addBookEvent : Event<Book>

    @Inject
    @Deleted
    private lateinit var deleteBookEvent : Event<Book>

    @Inject
    @Updated
    private lateinit var updateBookEvent : Event<Book>


    fun createBook(title : String, price : Double, saleOut : Boolean) : Book{
        val book = Book(title, price, saleOut)
        println("事件準備觸發...")
        addBookEvent.fire(book)
        println("事件觸發完畢!")
        return book
    }

    fun deleteBook(title : String, price : Double, saleOut : Boolean) : Book{
        val book = Book(title, price, saleOut)
        println("事件準備觸發...")
        deleteBookEvent.fire(book)
        println("事件觸發完畢!")
        return book
    }
}

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.FUNCTION)
annotation class Added

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.FUNCTION)
annotation class Deleted

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FIELD, AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.FUNCTION)
annotation class Updated
```


```kotlin
@Singleton
class InventoryService {
    private val inventory : MutableList<Book> = ArrayList()


    fun addBook(@Observes @Added book : Book){ //  observes any event typed with Book. The annotated parameter is called the event parameter
        println("庫存加一本書 $book")
        inventory.add(book)
    }

    fun deleteBook(@Observes @Deleted book: Book){
        println("庫存刪掉書 $book")
        // do something
    }

    fun updateBook(@Observes @Updated  book: Book){
        println("更新書 $book")
        // do something
    }
}
```
測試事件是否成功
```kotlin
@Path("/")
class BookAddTest{
    @Inject
    lateinit var service : BookService
    @GET
    @Path("/addBook")
    fun createBookThenAdd() {
        service.createBook("book 很奇妙", 23.25, false)
    }

    @GET
    @Path("/deleteBook")
    fun deleteBook(){
        service.deleteBook("book 很奇妙", 1100.25, false)
    }
}
```