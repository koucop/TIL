# AnExplicitJob (명시적 작업)

[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 
coroutine builder 는 실행된 코루틴을 처리하는 
[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) 
object 를 반환하고 explicitly(명시적으로) 완료를 기다리는 데 사용할 수 있다.  
예를 들어, children(자식) 코루틴이 완료될 때까지 기다린 다음 Done 문자열을 print 할 수 있습니다.

```kotlin
val job = launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello")
job.join() // wait until child coroutine completes
println("Done") 
// output :
// Hello
// World!
// Done
```
