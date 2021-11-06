# Run non cancellable block

이전 예제의 finally block에서 suspend functinon 을 사용하려고 하면 이 코드를 실행하는 코루틴이 취소되기 때문에 `CancellationException`이 발생한다.  
정상적으로 작동하는 모든 closing operations(닫기 작업)(e.g. 파일 닫기, 작업 취소 또는 모든 종류의 통신 채널 닫기)은 일반적으로 non-blocking 이고, suspend function 을 포함하지 않기 때문에 일반적으로 문제가 되지 않는다.  
그러나 드물게 취소된 코루틴에서 suspend 해야 하는 경우 다음 예제와 같이 withContext 함수와 NonCancellable context 를 사용하여 `withContext(NonCancellable) {...}`에 해당 코드를 래핑할 수 있다.

```kotlin
val job = launch {
    try {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    } finally {
        withContext(NonCancellable) {
            println("job: I'm running finally")
            delay(1000L)
            println("job: And I've just delayed for 1 sec because I'm non-cancellable")
        }
    }
}
delay(1300L) // delay a bit
println("main: I'm tired of waiting!")
job.cancelAndJoin() // cancels the job and waits for its completion
println("main: Now I can quit.")

// output : 
// job: I'm sleeping 0 ...
// job: I'm sleeping 1 ...
// job: I'm sleeping 2 ...
// main: I'm tired of waiting!
// job: I'm running finally
// job: And I've just delayed for 1 sec because I'm non-cancellable
// main: Now I can quit.
```
