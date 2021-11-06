# Making computation code cancellable

computation code(계산 코드)를 cancellabel(취소 가능)하게 만드는 방법에는 두 가지가 있다.  
1. 취소를 확인하는 suspend function 을 주기적으로 호출하는 것 (e.g. [yield](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html) function) 
2. 취소 상태를 명시적으로 확인하는 것 (e.g. `isActive`)

이전 예제의 while(i < 5)을 while(isActive)로 바꾸고 다시 실행하면, 취소 되어지는 것을 확인 할 수 있다.

```kotlin
val startTime = System.currentTimeMillis()
val job = launch(Dispatchers.Default) {
    var nextPrintTime = startTime
    var i = 0
    while (isActive) { // cancellable computation loop
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
// main: Now I can quit.
```

[isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html)
는 `CoroutineScope` object 를 통해 코루틴 내부에서 사용할 수 있는 확장 속성이다.
