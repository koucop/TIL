# CancellingCoroutineExecution

long-running application(장기 실행 애플리케이션)에서는 백그라운드 코루틴에 대한 세밀한 제어가 필요할 수 있다.  
예를 들어 사용자가 코루틴을 시작한 페이지를 닫았을 수 있으며 이제 그 결과가 더 이상 필요하지 않아 작업을 취소해야 할 수 있다.  
`launch` 는 실행 중인 코루틴을 취소하는 데 사용할 수 있는 `Job` 을 반환한다.

```kotlin
val job = launch {
    repeat(1000) { i ->
        println("job: I'm sleeping $i ...")
        delay(500L)
    }
}
delay(1300L) // delay a bit
println("main: I'm tired of waiting!")
job.cancel() // cancels the job
job.join() // waits for job's completion 
println("main: Now I can quit.")
```

위의 예시를 보면, main 에서 `job.cancel()` 호출하게 되면 취소되기 때문에 더 이상 print 가 되지 않는 것을 확인할 수 있다.  
여기서 `cancel()` 과 `join()` 은 extension function 인 
[cancelAndJoin()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel-and-join.html) 
으로 대체할 수 있다.
