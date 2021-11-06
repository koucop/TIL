# ScopeBuilder

launch, runBlocking 과 같은 builder 에서 제공하는 CoroutineScope 외에도 coroutineScope 빌더를 사용하여 고유한 scope 를 선언할 수 있다.  
CoroutineScope 를 만들고 실행된 모든 children(자식)이 완료될 때까지 완료되지 않는다.  
runBlocking 및 coroutineScope 빌더는 모두 자신과 모든 자식이 완료될 때까지 기다리기 때문에 비슷해 보일 수 있다.  
주요한 차이점은 runBlocking 메서드는 대기를 위해 현재 스레드를 block(차단)하는 반면 coroutineScope는 일시 중단되어 다른 용도를 위해 기본 스레드를 underlying(해제)한다는 것이다.  
그 차이 때문에 runBlocking은 regular function 이고 coroutineScope는 일시 suspend function 이다.  
모든 suspend function 에서 coroutineScope를 사용할 수 있는데,  
아래 예시에서 `Hello`와 `World`의 동시 인쇄를 `suspend fun doWorld()` function 으로 이동할 수 있다.  

```kotlin
fun main() = runBlocking {
    doWorld()
}

suspend fun doWorld() = coroutineScope {  // this: CoroutineScope
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```
