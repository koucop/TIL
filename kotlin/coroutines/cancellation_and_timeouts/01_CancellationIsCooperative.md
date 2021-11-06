# CancellationIsCooperative

코루틴 cancellation(취소)은 cooperative(협조적)이다.  
kotlinx.coroutines의 모든 suspend function 은 *cancellable* 하고, 만약 코루틴이 취소 되었다면 호출시에 `CancellationException` 을 던진다.  
그러나 코루틴이 computation(계산)에서 작동하고 취소를 확인하고 있지 않다면 다음 예제와 같이 취소할 수 없다.

```kotlin
val startTime = System.currentTimeMillis()
val job = launch(Dispatchers.Default) {
    var nextPrintTime = startTime
    var i = 0
    while (i < 5) { // computation loop, just wastes CPU
        // print a message twice a second
        if (System.currentTimeMillis() >= nextPrintTime) {
            println("job: I'm sleeping ${i++} ...")
            nextPrintTime += 500L
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
// job: I'm sleeping 3 ...
// job: I'm sleeping 4 ...
// main: Now I can quit.
```
