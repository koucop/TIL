# Your first coroutine

코루틴은 일시 중단 가능한 계산의 인스턴스이다.  
코드의 나머지 부분과 동시에 작동하는 코드 블록을 실행 한다는 점에서 개념적으로 스레드와 유사하다.  
그러나 코루틴은 특정 스레드에 바인딩되지 않고, 한 스레드에서 실행을 일시 중지하고 다른 스레드에서 다시 시작할 수 있다.

코루틴은 경량 스레드라고 생각할 수 있지만 실제 사용을 보면 기존 스레드와는 다른 여러 차이점들을 확인할 수 있다.  

```kotlin
fun main() = runBlocking { // 여기에서 this 는 CoroutineScope 임
    launch { // 새로운 코루틴을 launch 함
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // print before delay
}

rsult : 
Hello
World!
```

`launch` 는 독립적으로 계속 작동하는 나머지 코드와 동시에 새 코루틴을 시작하는 코루틴 빌더이다.  
그래서 Hello가 먼저 인쇄된 것을 확인 할 수 있다.

`delay` 는 특별한 *suspending function* 이고, 특정 시간 동안 코루틴을 일시 중단할 수 있다.  
코루틴을 일시 중단하면 밑에 있는 스레드가 차단되지 않기 때문에 다른 코루틴이 실행하고 사용되는 코드에 밑에 있는 스레드를 사용할 수 있다.

`runBlocking` 은 `fun main()` 의 코루틴이 아닌 세상과 `runBlocking { ... }` 중괄호 내부의 코루틴 세상을 연결하는 다리 역할을 하는 코루틴 빌더이다.  

위 코드에서 `runBlocking`을 제거하거나 잊어버리면 `launch` 가 `CoroutineScope` 에서만 선언될 수 있기 때문에 다음과 같은 오류가 발생한다.

```
Unresolved reference: launch
```

`runBlocking` 이라는 이름은 `runBlocking { ... }` 내부의 모든 코루틴이 실행을 완료할 때까지 이를 실행하는 스레드(위 예시의 경우 메인 스레드)가 호출 기간 동안 차단된다는 것을 의미한다.  

### Structured concurrency

코루틴은 **structured concurrency** 의 원칙을 따른다.  
즉, 새로운 코루틴은 코루틴의 수명을 제한하는 특정 `CoroutineScope` 에서만 시작될 수 있다.  
`runBlocking`이 해당하는 `scope` 를 설정하기 때문에 위의 예시에서 1초의 지연 후 `World!` 찍히고 나서야 종료된다.

실제 Application 에서는 많은 코루틴을 `launch` 하게 되는데, 이 때 **structured concurrency** 는 손실되지 않고 누수되지 않는 것을 보장한다.  
그리고 모든 하위 코루틴이 완료될 때까지 바깥 `scope` 를 완료할 수 없다.  
또한 **structured concurrency** 은 코드의 모든 오류가 적절하게 보고되고 손실되지 않도록 보장한다.

