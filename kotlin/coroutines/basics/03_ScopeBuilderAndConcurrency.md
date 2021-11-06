# ScopeBuilderAndConcurrency

`coroutineScope` builder 는 suspend function 내에서 여러 동시 작업을 수행하는 데 사용할 수 있다.  
아래 예시처럼 `doWorld` suspend function 내에서 두 개의 동시 코루틴을 실행해 보면,

```kotlin
// Sequentially executes doWorld followed by "Done"
fun main() = runBlocking {
    doWorld()
    println("Done")
}

// Concurrently executes both sections
suspend fun doWorld() = coroutineScope { // this: CoroutineScope
    launch {
        delay(2000L)
        println("World 2")
    }
    launch {
        delay(1000L)
        println("World 1")
    }
    println("Hello")
}
// output :
// Hello
// World 1
// World 2
// Done
```

`launch { ... }` 블록 안의 두 코드는 동시에 실행되며 시작 후 1초 후에 World 1이 먼저 print 되고 시작 후 2초 후에 World 2가 다음에 print 된다.  
doWorld의 `coroutineScope`는 둘 다 완료된 후에만 완료되므로 doWorld가 반환되고 그 후에만 Done 이 print 되도록 허용한다.
