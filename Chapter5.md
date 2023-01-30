## 레시피 5.6 주어진 범위로 값 제한하기

값이 주어졌을 때, 주어진 값이 특정 범위 안에 속하면 해당 값을 리턴하고, 그렇지 않다면 범위의 최솟값 또는 최댓값을 리턴하는 법을 알아본다.

`kotlin.ranges`의 `coerceIn` 함수를 범위 인자 또는 최솟값, 최댓값과 함께 사용하면 된다.

### 예시 코드
- 범위 인자
    ```
    fun test(){
        val range = 3..8
        
        println(5.coerceIn(range)) // 5 출력
        println(1.coerceIn(range)) // 3 출력
        println(9.coerceIn(range)) // 8 출력
    }
    ```
- 최솟값, 최댓값
    ```
    fun test(){
        val min = 3
        val max = 8
        
        println(5.coerceIn(range)) // 5 출력
        println(1.coerceIn(range)) // 3 출력
        println(9.coerceIn(range)) // 8 출력
    }
    ```
  
---
## 레시피 5.7 컬렉션을 윈도우로 처리하기
값 컬렉션이 주어진 경우 컬렉션을 횡단하는 작은 윈도우를 이용해 컬렉션을 처리하는 법을 알아본다.

컬렉션을 같은 크기로 나누고 싶다면 `chunked` 함수를 사용하고, 정해진 간격으로 컬렉션을 따라 움직이는 블록을 원한다면 `windowed` 함수를 사용하면 된다.

`chunked` 함수의 시그니처는 다음과 같다.
```
fun <T> Iterable<T>.chunked(size: Int): List<List<T>>

fun <T, R> Iterable<T>.chunked(
    size: Int,
    transform: (List<T>) -> R
): List<R>
```

### 예시 코드
- 첫 번째 시그니처
    ```
    fun test(){
        val range = 0..10
    
        val chunked = range.chunked(3)
        println(chunked) // [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10]] 출력
    }
    ```
  컬렉션이 같은 크기로 나누어졌고, `List<List<T>>`를 반환하는 것을 출력으로 알 수 있다.


- 두 번째 시그니처
    ```
    fun test(){
        val range = 0..10

        val chunkedSum=range.chunked(3){ it.sum() }
        println(chunkedSum) // [3, 12, 21, 19] 출력
    }
    ```
  컬렉션이 같은 크기로 나눠진 후, `transform` 인자의 람다가 적용된 것을 출력으로 알 수 있다.


`chunked` 함수는 구현을 `windowed`에 위임한다.
### 구현 코드
```
public fun <T> Iterable<T>.chunked(size: Int): List<List<T>> {
    return windowed(size, size, partialWindows = true)
}
```
<br>

`windowed` 함수는 세 개의 인자를 받고 그중 두 개의 인자는 선택 사항이다.
- `size`: 각 윈도우에 포함될 원소 개수
- `step`: 각 단계마다 전진할 원소의 개수(기본 1개)
- `partialWindows`: 나뉘어 있는 마지막 부분이 윈도우에 필요한 만큼의 원소 개수를 갖지 못한 경우, 해당 부분을 그대로 유지할지 여부를 알려주는 불리언 값. 기본값은 `false`

### 예시 코드
```
fun test(){
    val range = 0..10

    println(range.windowed(3, 3)) // [[0, 1, 2], [3, 4, 5], [6, 7, 8]] 출력
    println(range.windowed(3, 3) {it.sum()}) // [3, 12, 21] 출력
    println(range.windowed(3, 1)) // [[0, 1, 2], [1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6], [5, 6, 7], [6, 7, 8], [7, 8, 9], [8, 9, 10]] 출력, 원소 한칸씩 전진
    println(range.windowed(3, 1) { it.sum()}) // [3, 6, 9, 12, 15, 18, 21, 24, 27]
}
```
위에서 설명한 인자말고 람다를 제공 받는 경우가 있길래 `windowed` 함수의 시그니처를 찾아보았다. `windowed` 함수의 시그니처는 다음과 같다.
```
fun <T> Iterable<T>.windowed(
    size: Int,
    step: Int = 1,
    partialWindows: Boolean = false
): List<List<T>>

fun <T, R> Iterable<T>.windowed(
    size: Int,
    step: Int = 1,
    partialWindows: Boolean = false,
    transform: (List<T>) -> R
): List<R>
```

---

## 레시피 5.8 리스트 구조 분해하기
리스트의 원소에 접근할 수 있게 구조 분해하는 방법을 알아본다.

최대 5개의 원소를 가진 그룹에 리스트를 할당한다. 구조 분해는 변수 묶음에 추출한 값을 할당해 객체에서 값을 추출하는 과정이다.
### 예시 코드
```
fun test(){
    val list = listOf("a", "b", "c", "d", "e", "f", "g", "h")
    val (a, b, c, d, e) = list
    println("$a, $b, $c, $d, $e") // a, b, c, d, e 출력
}
```
위 코드가 작동한 이유는 코틀린 표준 라이브러리의 `List` 클래스에 `N`이 1부터 5까지인 `componentN`이라는 이름의 확장 함수가 정의되어 있기 때문이다. 구조 분해는 `componentN`이라는 함수에 의존한다. 그래서 `List`에서는 최대 5개 원소까지 구조 분해를 할 수 있는 것이다. 6개를 시도하려 했지만 오류가 발생했다.

그러나 데이터 클래스는 정의돈 모든 속성 관련 `component` 메소드를 자동으로 추가한다.
### 예시 코드
```
data class Num(val num1: Int, val num2: Int, val num3: Int, val num4: Int, val num5: Int, val num6: Int)
fun test(){
    val numData = Num(1, 2, 3, 4, 5, 6)
    val (n1, n2, n3, n4, n5, n6) = numData
    println("$n1, $n2, $n3, $n4, $n5, $n6") // 1, 2, 3, 4, 5, 6 출력
}
```
`List`와는 다르게 제한 없이 구조 분해가 되는 것을 출력으로 알 수 있다.

데이터 클래스가 아닌 클래스를 정의할 경우, 필요한 `component` 메소드를 직접 정의해야 한다.
### 예시 코드
```
class Num(val num1: Int, val num2: Int, val num3: Int, val num4: Int, val num5: Int, val num6: Int)
operator fun Num.component1() = num1
operator fun Num.component2() = num2
operator fun Num.component3() = num3
operator fun Num.component4() = num4
operator fun Num.component5() = num5
operator fun Num.component6() = num6

fun test(){
  val numData = Num(1, 2, 3, 4, 5, 6)
  val (n1, n2, n3, n4, n5, n6) = numData
  println("$n1, $n2, $n3, $n4, $n5, $n6") // 1, 2, 3, 4, 5, 6 출력
}
```
