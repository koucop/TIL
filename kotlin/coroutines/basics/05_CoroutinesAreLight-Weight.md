# CoroutinesAreLight-Weight

```kotlin
import kotlinx.coroutines.*

//sampleStart
fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            print(".")
        }
    }
}
//sampleEnd
```

위의 샘플 코드를 실행했을 때, 100,000 개의 코루틴을 시작하고 5초 후에 각 코루틴들은 "." 을 print 한다.

이제 위의 코드에서 `runBlocking` 을 `Thread` 로 대체하고, `delay(5000L)` 을 `Thread.sleep` 으로 변경해보면 out-of-memory error 를 발생시키는 것을 확인할 수 있다.
