# Timeout

코루틴의 실행을 취소하는 가장 명백한 실제 이유는 실행 시간이 일부 제한 시간을 초과했기 때문이다.  
`Job` 에 대한 참조를 수동으로 추적하고 지연 후 추적된 것을 취소하기 위해 별도의 코루틴을 시작하게끔 만들 수 있고, `withTimeout` 을 통해 앞선 동작을 달성할 수 있다.  

```kotlin
withTimeout(1300L) {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
}

// output : 
// I'm sleeping 0 ...
// I'm sleeping 1 ...
// I'm sleeping 2 ...
// Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms
// Copied!
```

`withTimeout`에 의해 throw되는 `TimeoutCancellationException`은 `CancellationException`의 하위 클래스이다.  
cancellation 은 exception 일 뿐이므로 모든 리소스는 일반적인 방식으로 닫힌다.  
특정 종류의 시간 초과에 대해 추가 작업을 수행하기 위해서는 `try {...} catch(e: TimeoutCancellationException) {...}` block 을 사용하거나 거나 `withTimeoutOrNull` 함수를 사용해야 한다.  
withTimeoutOrNull 은 withTimeout이지만 예외를 던지는 대신 시간 초과 시 null을 반환한다.

```kotlin
val result = withTimeoutOrNull(1300L) {
    repeat(1000) { i ->
        println("I'm sleeping $i ...")
        delay(500L)
    }
    "Done" // will get cancelled before it produces this result
}
println("Result is $result")

// output :
// I'm sleeping 0 ...
// I'm sleeping 1 ...
// I'm sleeping 2 ...
// Result is null
```
