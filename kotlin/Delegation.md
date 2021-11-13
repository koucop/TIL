# Delegation

Delegate Pattern 은 implementation inheritance 에 대한 좋은 대안으로 입증되었으며, 

Kotlin 에서는 별도의 boiler plate 없이 기본적으로 지원해 준다.

아래의 Delegate Pattern 을 적용시킨 예시에서 Derived 부분을 보면 constructor 에 붙은 `b` 로 인해 Derived 객체에 내부적으로 저장되게 만들 수 있고

`Base by b` 를 통해 컴파일러가 Base 의 모든 메서드들을 `b` 로 생성해 준다.

``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```

Overriding a member of an interface implemented by delegation

이렇게 Delegate Pattern 을 사용하더라도 Overrides 는 예상대로 작동한다.  
컴파일러는 `delegate object` 대신 `override` 한 구현을 사용한다.  
`override fun printMessage() { print("abc") }` 를 Derived에 추가하는 경우 printMessage가 호출될 때 10 대신 abc가 프린트 된다.

```kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    println()
    Derived(b).printMessageLine()
}

// output :
// abc
// 10
```

이 때, `Derived` 에서 override 된 member(field) 는 인터페이스 멤버의 자체 구현에만 액세스할 수 있는 `delegate object` 의 member(field) 에서는 호출되지 않는다.

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}

// output : 
// BaseImpl: x = 10
// Message of Derived
```
