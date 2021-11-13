# StoringPropertiesInMap

delegate 에서 또 다른 일반적인 사용 사례는 `values of properties`을 `map` 에 저장하는 것이다.  
이것은 JSON 구문 분석 또는 기타 동적 작업 수행과 같은 애플리케이션에서 자주 사용될 수 있다.  
이 경우 `map` 인스턴스 자체를 `delegated property` 의 `delegate` 로 사용할 수 있다.

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

val user = User(
    mapOf(
        "name" to "John Doe",
        "age"  to 25
    )
)

println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25

class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int by map
}
```

`delegate property` 는 속성들의 이름과 연결된 문자열 키를 통해 이 `map`에서 값을 가져온다.  
그리고, 맨 아래에 있는 예시처럼 `val` 뿐만 아니라 `var`에서도 사용 가능하다.

