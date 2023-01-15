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
