## 레시피 13.1 코루틴 빌더 선택하기
코루틴을 생성하는 올바른 함수를 선택해야 한다.

새 코루틴을 생성하려면 빌더 함수 `runBlocking`, `launch`, `async` 중 하나를 사용하면 된다. `runBlocking` 빌더 함수는 최상위 함수인 반면, `launch`와 `async`는 `CoroutineScope`의 확장 함수다.

`GlobalScope`에 정의된 `launch`와 `async`도 있음을 인지하고, 사용하지 말 것을 권장한다.

### runBlocking
`runBlocking`은 명령줄 검증 또는 테스트에 유용하다. `runBlocking`은 현제 스레드를 블록하고 모든 내부 코루틴이 종료될 때까지 블록한다. 시그니처는 다음과 같다.
```
fun <T> runBlocking(block: suspend CorutineScope.() -> T): T
```
함수 자체는 `suspend` 함수가 아니므로 보통 함수에서 호출할 수 있다.

### launch
독립된 프로세스를 실행하는 코루틴을 시작하고, 해당 코루틴에서 리턴값을 받을 필요가 없다면 `launch` 코루틴 빌더를 사용하면 된다. `launch` 함수는 `CoroutineScope`의 확장 함수이기 때문에 `CorutineScope`이 사용 가능한 경우에만 사용할 수 있다.`launch` 함수는 코루틴 취소가 필요하다면 사용할 수 있는 `Job` 인스턴스를 리턴한다. 시그니처는 다음과 같다.
```
fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job
```
`CoroutineContext`는 다른 코루틴과 상태를 공유하기 위해 사용한다. `CoroutineStart` 파라미터는 오직 `DEFAULT`, `LAZY`, `ATOMIC`, `UNDISPATCHED` 값만이 될 수 있다.

### async
값을 리턴해야 하는 경우에는 일반적으로 `async` 빌더를 사용한다. `async` 빌더도 `CoroutineScope`의 확장 함수이며 시그니처는 다음과 같다.
```
fun CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T>
```
파라미터로 제공한 `suspend` 함수는 값을 리턴하면 `aync` 함수가 `Deferred` 인스턴스로 해당 값을 감싼다. `Deferred` 에서 중요한 함수는 코루틴이 완료될 때까지 기다리는 `await`이다.

### coroutineScope 빌더
마지막으로 `coroutineScope` 함수는 종료 전에 포함된 모든 코루틴이 완료될 때가지 기다리는 `suspend` 함수다. `coroutineScope` 함수는 `runBlocking` 과 다르게 메인 스레드를 블록하지 않는다는 것이 장점이지만 반드시 `suspend` 함수의 일부로서 호출돼야 한다.

`coroutineScope`의 이점은 코루틴 완료 여부를 확인하기 위해 코루틴을 조사해야 할 필요가 없다는 것이다. 자동으로 모든 자식 코루틴이 완료될 때가지 기다린다. 만약 코루틴이 하나라도 실패하면 나머지 코루틴을 취소한다.

---
## 레시피 13.2 async/await를 withContext로 변경하기
`async`로 코루틴을 시작하고 바로 다음에 코루틴이 완료될 동안 기다리는 `await` 코드를 간소화하고 싶다. 그러면 `withContext` 함수를 사용하면 된다. 시그니처는 다음과 같다.
```
suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T
```
코틀린 공식 문서에서 `withContext`는 "주어진 코루틴 컨텍스트와 함께 명시한 일시정지 블록을 호출하고, 완료될 때까지 일시정지한 후에 그 결과를 리턴한다."고 나와 있다. 예시를 보면 바로 이해된다.
```
suspend fun retrieve1(url: String) = coroutineScope {
    // can replace async/await with withContext
    async(Dispatchers.IO + CoroutineName("async")) {
        println("Retrieving data on ${Thread.currentThread().name}")
        delay(100L)
        "asyncResults"
    }.await()
}

suspend fun retrieve2(url: String) =
    withContext(Dispatchers.IO + CoroutineName("withContext")) {
        println("Retrieving data on ${Thread.currentThread().name}")
        delay(100L)
        "withContextResults" 
}

fun main() = runBlocking {
    val result1 = retrieve1("www.mysite.com")
    val result2 = retrieve2("www.mysite.com")
    println("printing result on ${Thread.currentThread().name} $result1")
    println("printing result on ${Thread.currentThread().name} $result2")
}
```
---
## 레시피 13.3 디스패처 사용하기
`I/O` 또는 다른 작업을 위한 전용 스레드 풀을 사용해야 한다.
`Dispatchers` 클래스에서 적당한 디스패처를 사용한다.

코루틴은 `CoroutineContext` 타입의 컨텍스트 내애서 실행된다. 코루틴 컨텍스트에는 `CoroutineDispatcher` 클래스의 인스턴스에 해당하는 코루틴 디스패처가 포함돼 있다. 디스패처는 코루틴이 어떤 스레드 혹은 어떤 스레드 풀에서 코루틴을 실행할지 결정한다. `launch` 또는 `async` 같은 빌더를 사용할 때 `CoroutineContext` 선택 파라미터를 통해 사용하고 싶은 디스패처를 명시할 수 있다.

- `Dispatchers.Default`
- `Dispatchers.IO`
- `Dispatchers.Unconfined`

마지막 디스패처는 일반적으로 애플리케이션 코드에서 사용하면 안 된다. 코루틴이 대규모의 계산 리소스를 소모하는 경우에 기본 디스패처가 적합하다. 파일 `I/O` 또는 블록킹 네트워크 `I/O`는 `IO` 디스패처가 적합하다.
```
fun main() = runBlocking {
    launchWithIO()
    launchWithDefault()
}

suspend fun launchWithIO() {
    withContext(Dispatchers.IO) {
        delay(100L)
        println("Using Dispatchers.IO")
        println(Thread.currentThread().name)
    }
}

suspend fun launchWithDefault() {
    withContext(Dispatchers.Default) {
        delay(100L)
        println("Using Dispatchers.Default")
        println(Thread.currentThread().name)
    }
}
```

안드로이드 `API`에는 추가로 `Dispatchers.Main`이라는 디스패처가 있다. `Main`에서 `UI`를 갱신하는 모든 작업을 하길 바라는 일반적인 `UI` 툴킷이지만 모든 작업에 추가 시간이 필요하거나 `Main`을 지연시킨다. 안드로이드 `Main` 디스패처를 사용하려면 `kotlinx-coroutines-android` 의존성을 추가해야 한다.
