# 🚣‍♀️ Coroutines

## 개요

Kotlin 에서는 Coroutine 을 통해 이러한 문제점을 Kotlin 언어 레벨에서 비동기 처리나 non-blocking 프로그래밍을 유연하게 처리할 수 있도록 만들었다.  
좀 더 부연설명 하자면, 코루틴은 스레드와 기능적으로 같지만, 스레드에 비교하면 좀더 가볍고 유연하며 한단계 더 진화된 병렬 프로그래밍을 위한 기술이다.  
하나의 스레드 내에서 여러개의 코루틴이 실행되는 개념인데, 아래의 코드는 동일한 기능을 스레드와 코루틴으로 각각 구현한 코드의 예시이다.

```kotlin
Thread(Runnable {
    for(i in 1..10) {
        Thread.sleep(1000L)
        print("I'm working in Thread.")
    }
}).start()
GlobalScope.launch() {
    repeat(10) {
        delay(1000L)
        print("I'm working in Coroutine.")
    }
}
```
