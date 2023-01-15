## 레시피 3.8 싱글톤 생성하기

싱글톤 디자인 패턴을 사용하고 싶을 때 코틀린에서는 어떻게 하는가 알아본다. 일단 싱글톤을 정의하는 방법은 다음과 같다.
1. 클래스의 모든 생성자를 `private`로 정의한다.
2. 필요하다면 해당 클래스를 인스턴스화하고 그 인스턴스 참조를 리턴하는 정적 팩토리 메소드를 제공한다.

자바 표준 라이브러리에서 싱글톤 예는 `Runtime` 클래스가 있다.
```
public class Runtime {
  private static final Runtime currentRuntime = new Runtime();
  
  public static Runtime getRuntime(){
    return currentRuntime;
  }
  
  private Runtime(){}
} 
```

코틀린에서 싱글톤 구현은 `object` 키워드를 사용하기만 하면 된다.
```
object MySingleton{
  val myProperty = 3
  
  fun myFunction() = "Hello"
}
```
생성된 바이트코드를 디컴파일하면 결과는 다음과 같다.
```
public final class MySingleton {
  private static final int myProperty = 3;
  public static final MySingleton INSTANCE;
  
  private mySingleton(){}
  
  public final void myFunction(){
    return "Hello";
  }
  
  static {
    MySingleton var0 = new MySingleton();
    INSTANCE = var0;
    myProperty = 3;
  }
}
```

싱글톤 멤버에 다음과 같이 접근하면 된다.
```
MySingleton.myFunction()
MySingleton.myProperty
```

자바에서 코틀린 싱글톤에 접근할 때는 코틀린이 생성한 `INSTANCE` 속성을 사용한다.
```
MySingleton.INSTANCE.myFunction();
MySingleton.INSTACNE.getMyProperty;
```

코틀린은 자바와 달리 편리하게 싱글톤을 생성할 수 있다. 하지만 단점도 있다. 인자와 함게 싱글톤을 인스턴스화 하려면 문제가 발생한다. 코틀린 `object`는 생성자를 가질 수 없기 때문에 쉽게 인자를 전달할 수 있는 방법이 없다.

---

## 레시피 3.9 Nothing에 관한 야단법석
`Nothing` 클래스를 사용법에 맞게 적절하게 사용하는 법을 알아본다.

코틀린에는 `Nothing`이라는 클래스가 있는데, 클래스의 전체 구현은 다음과 같다.
```
package kotlin

public class Nothing private constructor()
```

`private` 생성자는 클래스 밖에서 인스턴스화할 수 없다는 것을 의미하고, `Nothing` 클래스 안에서도 인스턴스화를 하지 않는다. 따라서 `Nohting`의 인스턴스는 존재하지 않는다. 코틀린 공식 문서에서는 **"결코 존재할 수 없는 값을 나타내기 위해 `Nothing`을 사용할 수 있다."** 고 명시되어 있다.

`Nothing` 클래스의 사용은 2가지 경우가 있다.
1. 함수 몸체가 전적으로 예외를 던지는 코드로 구성된 상황
  ```
  fun doNothing(): Nothing = throw Exception("Nothing at all")
  ```
  리턴 타입을 반드시 구체적으로 명시해야 하는데 해당 메소드는 결코 리턴하지 않으므로 리턴 타입은 `Nothing`이다. 이 부분이 자바 개발자들에게 많은 혼동을 준다. 자바에서는 어떤 타입의 예외를 던지든 메소드의 리턴 타입이 변경되지 않기 때문이다.
  
 2. 변수에 널을 할당할 때 구체적인 타입을 명시하지 않은 경우
    ```
    var x = null
    ```
    `x`는 널 할당이 가능한 타입이고 컴파일러는 `x`에 다한 다른 정보가 없기 때문에 추론된 `x`의 타입은 `Nothing?`이다. 
    
    코틀린에서 `Nothing` 클래스는 다른 모든 타입의 하위 타입이다. 어떤 수를 3으로 나눌 때 나머지는 반드시 0, 1, 2 중 한다.따라서 다음 예제의 `when` 문에는 `else`절이 필요 없지만 컴파일러는 알 수가 없다.
    ```
    for (n in 1..10){
      val x = when (n % 3) {
        0 -> "n % 3 = 0"
        1 -> "n % 3 = 1"
        2 -> "n % 3 = 2"
        else -> throw Exception("Houston, we have a problem...")
      }
      asswerTrue(x is string)
    }
    ```
    예외를 던지는 경우는 리턴하는 것이 아니므로 반환되는 리턴 타입은 `Nothing`이다. 하지만 예외가 발생하지 않으면 리턴 타입은 `String`이다. 컴파일러가 `x`의 타입을 추론할 때, `String`으로 추론하고, 이를 위해 `Nothing` 클래스는 다른 모든 타입의 하위 타입어이야 한다고 이해했다.
