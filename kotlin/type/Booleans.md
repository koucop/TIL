# Booleans

Boolean 타입에는 기본적으로 true, false 가 존재하고, nullable 로도 만들 수 있다.  
그리고 Boolean 에는 다음과 같은 operations 가 있다.  

- || – disjunction (logical OR)
- && – conjunction (logical AND)
- !- negation (logical NOT)

```kotlin
val myTrue: Boolean = true
val myFalse: Boolean = false
val boolNull: Boolean? = null

println(myTrue || myFalse) // true
println(myTrue && myFalse) // false
println(!myTrue) // false
```
