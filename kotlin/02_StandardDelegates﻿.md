# Standard delegates

Kotlin 표준 라이브러리는 몇 가지 유용한 종류의 `delegates` 를 위한 `factory methods` 를 제공한다.

## Lazy properties

[lazy()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 
는 람다를 통해 `lazy property` 을 구현하기 위한 `delegates` 역할을 할 수 있는 `Lazy<T>` 의 인스턴스를 반환하는 함수이다.  
get()에 대한 첫 번째 호출은 lazy()에 전달된 람다를 실행하고 결과를 기억하게 되고,  
이후에 호출되는 get() 에서는 단순히 기억된 결과를 반환해준다.
  
```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}

// output
// computed!
// Hello
// Hello
```

기본적으로 `lazy properties` 는 `synchronized` 된다.(LazyThreadSafetyMode.SYNCHRONIZED)  
> 값은 하나의 스레드에서만 계산되지만 모든 스레드는 동일한 값을 보게 되기 때문에, 
> 여러 스레드가 동시에 실행할 수 있도록 `initialization delegate` 의 동기화가 필요하지 않은 경우 
> `LazyThreadSafetyMode.PUBLICATION`을 `lazy()`에 매개변수로 전달합니다.

초기화가 항상 `property` 를 사용하는 스레드와 동일한 스레드에서 발생한다고 확신하는 경우 LazyThreadSafetyMode.NONE을 사용할 수 있다.  
이렇게 하면 `thread-safety guarantees` 및 관련 `overhead` 발생하지 않습니다.

## Observable properties

Delegates.observable()은 초기 값과 수정 `handler` 두 가지 인수를 가진다.

`handler` 는 속성에 할당할 때마다 호출된다(할당이 수행된 후).  
여기에는 할당되는 속성, 이전 값 및 새 값의 세 가지 매개변수가 있다.

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        **prop, old, new** -> println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}

// output
// <no name> -> first
// first -> second
```

할당을 가로채 거부하려면 `observable()` 대신 `vetoable()`을 사용하면 된다.  
`vetoable`에 전달된 `handler` 는 새 속성 값을 할당하기 전에 호출됩니다.
