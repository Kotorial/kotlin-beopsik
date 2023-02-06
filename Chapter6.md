## 레시피 6.4 시퀀스에서 yield하기

구간을 지정해 시퀀스 값을 생성하고 싶다. `yiled` 중단 함수와 함께 `sequence` 함수를 사용하는 것을 알아본다.

### `sequence` 함수의 시그니처
```
fun <T> sequence(
    block: suspend SequenceScope<T>.() -> Unit
): Sequence<T>
```
`sequence` 함수는 주어진 블록에서 평가되는 시퀀스를 생성한다.

### 일반적으로 시퀀스를 생성하는 방법
- 기존 데이터에서 `sequenceOf` -> `Sequence<T>` 반환
- 컬렉션을 `asSequence` -> `Sequence<T>` 반환
- 그 외엔 `genrateSequence` -> `Sequence<T>` 반환

생성한 시퀀스에 최종 연산(`first()`, `toList()` 등)을 해서 데이터를 처리

### `sequence` 함수
`sequence` 함수의 경우에는 필요한 때 `yield`해 시퀀스를 생성하는 **람다**를 제공해야 한다. 피보나치 함수를 예시로 들겠다.
```
fun fibonacciSequence() = sequence {
    var terms = Pair(0, 1)

    while (true) {
        yield(terms.first)
        terms = terms.second to terms.first + terms.second
    }
}
fun main() {
    println(fibonacciSequence().take(10).toList())
}
```

`yield` 함수는 이터레이터에 값을 제공하고 다음 값을 요청할 때까지 값 생성을 중단한다. 코틀린 런타임은 코루틴에 값을 제공한 후에 다음 값을 요청할 때까지 해당 코루틴을 중단시킬 수 있다.
`take` 연산에 의해 `yield`가 호출될 때마다 무한 루프는 값을 하나씩 제공한다.
