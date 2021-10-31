# Extract function refactoring

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}
```

`launch { ... }` 내부의 코드 블록을 별도의 함수로 꺼내기 위해,  
위 코드에 대해 리팩토링을 수행하면 suspend modifier(수정자)가 있는 새 function 을 만들게 된다.  
`suspend function` 는 일반 함수와 마찬가지로 코루틴 내부에서 사용할 수 있지만 `suspend function` 을 사용했을 때 다른 점은,  
다른 `suspend function` (e.g. `delay`) 을 사용하여 코루틴 실행을 일시 중단할 수 있다는 것이다.

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```
