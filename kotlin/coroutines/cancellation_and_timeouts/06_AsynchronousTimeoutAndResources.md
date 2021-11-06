# Asynchronous timeout and resources

withTimeout의 타임아웃 이벤트는 해당 block에서 실행 중인 코드와 관련하여 비동기적이며 timeout block 내부에서 반환되기 직전에도 언제든지 발생할 수 있다.  
block 내부의 일부 리소스를 열거나 획득해서 block 외부에서 닫거나 해제해야 하는 경우 이 점을 염두해야 한다.  

아래 예시에서와 같이 `Resource` 클래스를 사용하여 instance 가 생성될 때 `acquired` 카운터를 증가시키고 `close` function 에서 이 카운터를 감소시켜 생성된 횟수를 단순히 추적하고 싶을 때,  
약간의 지연 후에 `withTimeout` 블록 내부에서 이 리소스를 획득하고 외부에서 해제하도록 만들고 이것을 small timeout(짧은 시간 초과)으로 많은 코루틴을 실행해 보면,  

```kotlin
var acquired = 0

class Resource {
    init { acquired++ } // Acquire the resource
    fun close() { acquired-- } // Release the resource
}

fun main() {
    runBlocking {
        repeat(100_000) { // Launch 100K coroutines
            launch { 
                val resource = withTimeout(60) { // Timeout of 60 ms
                    delay(50) // Delay for 50 ms
                    Resource() // Acquire a resource and return it from withTimeout block     
                }
                resource.close() // Release the resource
            }
        }
    }
    // Outside of runBlocking all coroutines have completed
    println(acquired) // Print the number of resources still acquired
}
```

컴퓨터의 타이밍에 따라 다를 수 있지만 항상 0을 print 하지 않는다는 것을 알 수 있다.  
이 문제를 해결하기 위해서는 `withTimeout` block 에서 리소스에 대한 참조를 반환하는 대신 변수에 리소스에 대한 참조를 저장할 수 있다.

```kotlin
runBlocking {
    repeat(100_000) { // Launch 100K coroutines
        launch { 
            var resource: Resource? = null // Not acquired yet
            try {
                withTimeout(60) { // Timeout of 60 ms
                    delay(50) // Delay for 50 ms
                    resource = Resource() // Store a resource to the variable if acquired      
                }
                // We can do something else with the resource here
            } finally {  
                resource?.close() // Release the resource if it was acquired
            }
        }
    }
}
// Outside of runBlocking all coroutines have completed
println(acquired) // Print the number of resources still acquired
```

