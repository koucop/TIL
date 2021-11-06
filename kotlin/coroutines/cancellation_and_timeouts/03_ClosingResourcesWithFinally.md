# ClosingResourcesWithFinally

cancellable(취소 가능)한 suspend functino 은 취소 시 일반적인 방법으로 처리할 수 있는 `CancellationException`을 발생시킨다.  
예를 들어 `try {...} finally {...}` expression(표현식) 과 Kotlin `use` function 은 코루틴이 취소될 때 정상적으로 finalization actions(종료 작업)을 실행한다.

```kotlin
val job = launch {
    try {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    } finally {
        println("job: I'm running finally")
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
// main: Now I can quit.
```
