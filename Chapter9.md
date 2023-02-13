## 레시피 9.1 테스트 클래스 수명주기 설정하기
`Junit5`의 테스트 수명주기를 기본값인 테스트 함수당 한 번 대신 클래스 인스턴스당 한 번씩 인스턴스화하는 것을 알아본다.
`@TestInstance`를 사용하거나 `junit-platform.properties` 파일의 `lifecycle.default` 속성을 설정하면 된다.

기본적으로 `Junit4`는 각 테스트 메소드마다 테스트 클래스의 새 인스턴스를 생성한다. 테스트 자체로 독립적이지만 초기화 코드가 각 테스트마다 반복 실행된다는 단점이 있다.

자바에서는 이러한 단점을 피하기 위해 클래스의 모든 속성을 `static`으로 표시해 모든 초기화 코드를 딱 한 번만 실행하는 `@BeforeClass`가 달린 `static`메소드 안에 배치할 수 있다.

그러나 코틀린에서 자바와 동일하게 구현하려고 할 때 코틀린에는 `static` 키워드가 없다는 문제에 직면한다. 그럴 때는 동반 객체를 사용해야 한다.
```
class JUnit4ListTests{
    companion object{
        @JvmStatic
        private val strings = listOf("a", "b", "c")
        
        @BeforeClass
        @JvmStatic
        fun runBefore(){
            println("@strings")
        }
    }
    
    privatel val modifiable = ArrayList<Int>()
    
    @Before
    fun initialize(){
        println($modifiable)
        modifiable.add(3)
    }
    
    @Test
    public void test1(){}
}
```
동반 객체는 `strings` 컬렉션이 클래스 전체에서 한 번만 인스턴스화되고 원소가 채워지게 하려고 사용됐다.
`@Before`을 꼭 설정해야 하는 경우라도 `var` 사용은 코틀린 다운 구현에서 멀어지게 만든다. 다행히 `JUnit5`는 더 간단한 방법을 제공한다.
```
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class JUnit5ListTests{
    private val strings = listOf("a", "b", "c")
    privatel lateinit var modifiable : MutableList<Int>
    
    @BeforeEach
    fun setUp(){
        modifiable = mutableListOf(1, 3)
        println($modifiable)
    }
    
    @Test
    public void test1(){}
}
```
테스트 인스턴스의 수명주기를 `PER_CLASS`로 설정하면 테스트 함수의 양과 상관없이 테스트 인스턴스가 딱 하나만 생성된다.

`junit-platform.properties`파일을 생성해서 테스트 수명주기를 설정할 수 있다.
```
junit.jupiter.testinstance.lifecycle.default=per_class
```

---

## 레시피 9.2 테스트에 데이터 클래스 사용하기
코드를 부풀리지 않고 객체의 여러 속성을 체크하는 것을 알아본다. 

원하는 모든 속성을 캡슐화하는 데이터 클래스를 생성한다.

코틀린의 데이터 클래스는 `equals`, `toString`, `hashCode`, `copy`와 구조 분해를 위한 `component` 메소드를 자동으로 포함한다. 데이터 클래스의 이런 특성은 테스트를 위한 속성을 래핑하기에 적합하다.

```
data class Book(
    val isbn: String,
    val title: String,
    val author: String,
    val published: LocalDate
)
```
데이터 클래스의 모든 속성을 수동으로 테스트할 수 있다.
```
@Test
internal fun `test book the hard way`(){
    val book=service.findBookbyId("123")
    assertThat(book.isbn, `is`("123")
    assertThat(book.title, `is`("title")
    assertThat(book.author, `is`("author")
    assertThat(book.published, `is`(LocalDate.of(2013, Month.SEPTEMBER, 30)))
}
```
테스트는 작 동작하지만 모든 속성의 단언을 직접 작성해야 하고, 첫 번째 단언이 실패하면 모든 테스트가 실패하기 때문에 일부 속성은 전혀 확인되지 않는 문제가 발생할 수 있다.

다행히 `Junit5`에는 `Executable` 인스턴스의 가변 리스트를 받는 `assertAll` 메소드가 추가됐다. 이 메소드는 `Executable` 인스턴스의 단언이 1개 이상 실패하더라도 모든 인스턴스를 실행한다는 것이다.
```
@Test
internal fun `assertAll`(){
    val book=service.findBookbyId("123")
    assertAll("check all",
        { assertThat(book.isbn, `is`("123") },
        { assertThat(book.title, `is`("title") },
        { assertThat(book.author, `is`("author") },
        { assertThat(book.published, `is`(LocalDate.of(2013, Month.SEPTEMBER, 30))) })
}
```
`Executable` 인스턴스 대신 코틀린 람다가 사용됐는데, 여기서 사용한 람다는 인자가 없고 `assertThat` 함수는 `void`를 리턴하기 때문에 `Executable` 인터페이스에 적합하다.

그럼에도 개별 테스트를 작성해야 하는데, 이는 귀찮다. 코틀린 `data` 클래스는 이미 `equals` 메소드가 올바르게 구현돼 있으므로 다음과 같인 간소화할 수 있다.
```
@Test
internal fun `use data class`(){
    val book=service.findBookbyId("123")
    val expected = Book( isbn = "123",
        title = "title",
        author = "author",
        published = LocalDate.of(2013, Month.SEPTEMBER, 30))
    
    assertThat(book, `is`(expected))
}
```
컬렉션 인스턴스는 `Hamcrest matcher`가 제공하는 메소드를 이용해 컬렉션의 모든 원소를 확인할 수 있다. ex) `arrayContainingInAnyOrder`
