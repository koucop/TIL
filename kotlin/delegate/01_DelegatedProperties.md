# DelegatedProperties

일반적으로 `properties` 을 사용할 때면 필요할 때마다 수동으로 구현하게 끔 만들 수 있지만, 이 때, 한 번 구현하고 라이브러리에 추가시킨 후 나중에 다시 사용하도록 만드는 것이 더 유용할 수 있다.  

- Lazy properties: 값은 첫 번째 액세스에서만 계산시키도록 하는 방법
- Observable properties: `listeners` 를 통해 속성의 변경 사항에 대해 알림을 받는 방법
- 각 속성에 대해 별도의 필드 대신 맵에 속성을 저장하는 방법

이러한 경우를 다루기 위해 Kotlin은 `delegated properties`을 지원한다.

```kotlin
class Example {
    var p: String by Delegate()
}
```

`val/var <property name>: <type> by <expression>`. `property` 에 해당하는 `get()(및 set())`이 해당 `getValue()` 및 `setValue()` 메서드에 위임되기 때문에 `by` 뒤의 `expression` 은 `delegate` 이다.  
Property delegates 는 인터페이스를 구현할 필요가 없지만 `getValue()` 함수(및 var에 대한 `setValue()`)를 제공해야 한다.

```kotlin
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}  
```

Delegate의 인스턴스에 위임하는 p 를 읽을 때 Delegate 의 getValue() 함수가 호출된다.  
첫 번째 매개변수는 p 를 읽은 객체이고 두 번째 매개변수는 p 자체에 대한 설명을 담고 있다(예: 이름을 사용할 수 있음).
마찬가지로 p 에 할당하면 setValue() 함수가 호출된다. 처음 두 매개변수는 동일하고 세 번째 매개변수는 할당된 값을 보유한다.

```kotlin
val e = Example()
println(e.p)

e.p = "NEW"
```

다음은 각각 아래의 예시를 프린트한다.  

"$thisRef, thank you for delegating 'p' to me!"
"$value has been assigned to 'p' in $thisRef."
