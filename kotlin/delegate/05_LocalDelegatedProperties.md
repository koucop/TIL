# Local delegated properties

지역 변수를 `delegate property` 로 선언할 수 있다.  
아래 예시에서 확인할 수 있듯이, memoizedFoo 변수는 첫 번째 액세스에서만 계산되고, someCondition이 실패하면 변수가 전혀 계산되지 않는다.

```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```
