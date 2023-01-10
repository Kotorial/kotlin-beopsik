## 레시피 2.8 비트 불리언 연산자 사용

비트 값에 마스크를 적용하고 싶다.

코틀린에서 비트 마스킹 연산자는 `and`, `or`, `xor`, 그리고 `not` 대신 `inv`이다. 비트마스킹은 알고리즘 문제에도 활용되니까 알고 있으면 좋다.

`inv`는 숫자에 `2의 보수` 연산을 한다. `4`의 바이너리 값은 `0b00000100`이다. 여기에 `2의 보수` 연산을 하면 다음과 같다.
```
0b0000_0100.inv() -> 0b1111_1011 -> -(0b0000_0100 + 1) -> -0b0000_0101 -> -5
```

`and`,  `or`, `xor` 비트 연산자는 익숙하니 예시로 설명하겠다.
```
n1 = 0b0000_1100
n2 = 0b0001_1001

n1 and n2 -> 0b0000_1000
n1 or n2 -> 0b0001_1101
n1 xor n2 -> 0b0001_0101
```
<br>
<br>
비트 시프트 연산자와 비트 마스크 연산자를 사용해서 특정 위치의 값을 추출할 수 있다. 자바의 java.awt.Color 클래스로 예시를 들겠다.

Color 클래스의 RGB는 다음과 같다.
```
알파(4bit)_빨강(4bit)_초록(4bit)_파랑(4bit)
```

```
fun intsFromColor(color: Color): List<Int> {
    val rgb = color.rgb
    val alpha = rgb shr 24 and 0xff
    val red = rgb shr 16 and 0xff
    val green = rgb shr 8 and 0xff
    val blue = rgb and 0xff

    return listOf(alpha, red, green, blue)
}
```
알파, 빨강, 초록, 파랑의 올바른 값을 추출하기 위해 비트 시프트 연산자로 해당 `4bit`가 제일 오른쪽에 위치히게 비트 시프트 연산자를 사용한다. 그 후 `and` 비트 마스크 연산자를 이용해서 제일 오른쪽 `4bit` 값을 제외하고는 모두 `0`으로 만든다. 그러면 색상의 값을 추출할 수 있다.

<br>

이와 반대로 알파, 빨강, 초록, 파랑의 값을 합쳐서 RGB 값을 만들 수 있다.
```
fun colorFromInts(alpha: Int, red: Int, green: Int, blue: Int) = 
    (alpha and 0xff shl 24) or
    (red and 0xff shl 16) or
    (green and 0xff shl 8) or
    (blue and 0xff)
```
알파, 빨강, 초록, 파랑의 값을 비트 시프트 연산자로 RGB에서 해당 색상의 위치로 비트 시프트한다. 그 후 `or` 연산자를 통해 모든 색상의 값을 합친다.

비트 마스크 연산자를 잘 활용하면, 탐색 알고리즘에서 방문한 곳과 방문하지 않은 곳 체크 시 시간 복잡도를 줄일 수 있었다. 많이 쓸 일은 없지만 알고 있으면 가끔 쓸 곳이 잇을 것이다.



