## 레시피 7.1 `apply`로 객체 생성 후에 초기화하기
객체를 사용하기 전에 생성자 인자로만 할 수 없는 초기화 작업이 필요하다. `apply` 함수를 알아본다.

코틀린에는 객체에 적용할 수 있는 영역 함수가 몇 개 있다. `apply` 함수는 `this`를 인자로 전달하고 `this`를 리턴하는 확장 함수다.

### `apply`함수의 정의
```
inline fun <T> T.apply(block: T.() -> Unit): T
```

`apply` 함수는 모든 제네릭 타입 `T`에 존재하는 확장 함수다. `apply` 함수는 명시된 블록을 수신자인 `this`와 함께 호출하고 해당 블록인 완료되면 `this`를 리턴한다.

```
@Repository
class JdbcOfficerDAO(private val jdbcTemplate: JdbcTemplate) {
    private val insertOfficer = SimpleJdbcInsert(jdbcTemplate)
        .withTableName("OFFICERS")
        .usingGeneratedKeyColumns("id")
        
    fun save(officer: Officer) =
        officer.apply {
            id = insertOfficer.executeAndReturnKey(
                mapOf("rank" to rank,
                      "first_name" to first,
                      "last_name" to last))
        }
}
```
위 예시처럼 `Officer` 엔티티가 데이터베이스에 저장되고 해당 엔티티의 기본 키를 반환 받아서 초기화해야 할 때 `apply`를 사용하여 초기화 하면 된다. `Officer` 인스턴스는 `this`로서 `apply` 블록에 전달되기 때문에 블록 안에서 객체의 속성에 접근할 수 있다. `apply` 블록은 이미 인스턴스화된 객체의 추가 설정을 위해 사용하는 가장 일반적인 방법이다.

---

## 레시피 7.2 부수 효과를 위해 `also` 사용하기
코드 흐름을 방해하지 않고 메시지를 출력하거나 다른 부수 효과를 생성하고 싶다. `also` 함수를 사용해 부수 효과를 생성하는 동작을 수행해본다.

### `also` 확장 함수
```
public inline fun <T> T.also(
    block: (T) -> Unit
): T
```
`also`는 모든 제네릭 타입 `T`에 추가되고 `blcok` 인자를 실행시킨 후에 자신을 리턴한다. 일반적으로 객체에 함수 호출을 연쇄시키기 위해 사용한다.

```
val book = create(Book)
    .also { println(it) }
    .also { Logger.getAnonymousLogger().info(it.toString)) }
```

