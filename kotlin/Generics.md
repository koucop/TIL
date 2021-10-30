# Generics

이 문서는 [Kotlin-generics 공식문서](https://kotlinlang.org/docs/generics.html) 기반으로 작성하였다.

## 개요

Generic이란 Class 의 객체 생성시부터 어떤 자료형을 사용할 건지 정의(강제)하여서,  
해당 Class 에서 매개변수에 사용되는 자료형의 룰을 정하고 그로인해 안정성을 높이는 도구를 말한다.  

아래 예시처럼 Kotlin 에서의 클래스는 Java 에서 처럼 Type 을 가질 수 있고, 일반적으로 `<T>`로 표기한다.  

```kotlin
class Sample<T>(t: T) {
    var value = t
}
```

타입을 가지고 있는 Class 객체를 생성하고 확인하기 위해서는 아래처럼 해주면 된다.  
두번 째 줄에서 별도로 타입을 넣지 않아도 되는 이유는 "Sample" 을 통해, 타입 추론이 가능하기 때문이다.

```kotlin
val firstSample: Sample<String> = Sample<String>("Sample")
val secondSample = Sample("Sample")
```

Generic 을 사용하면, 아래의 이점을 챙길 수 있다.

1. Type Safety 를 확보 할 수 있다.
2. Runtime 에러가 아닌, Compile 타임에 에러를 발생시켜서 보다 안전한 코드작성이 가능하다.
3. 하나하나 TypeCasting 해주던 부분을 없앨 수 있다.

그리고 Generic 사용시에 타입이 어느 범위까지 영향을 미칠 것인지 정할 수 있는데, 이를 Type Bound 라고 한다.  

Type Bound 종류 에는 아래 3가지가 있고, 지금부터 각 내용에 대해 정리해보려고 한다.

- Invariant(불변, 무공변)
- covariant(공변)
- contravariant(반공변, 반변)

## Invariant(불변, 무공변)

일단 Variance(변화) 관련 내용에 들어가기 앞서, SubType 에 대한 개념을 알아야 한다.  
SubType이란 어떤 Class가 다른클래스를 상속받은것을 의미한다.  
다음과 같이 Line, Surface Class 가 있다고 했을 때, Surface 는 Line 을 상속받고 있으므로 Surface 는 Line 의 SubType 이라고 할 수 있다.  

```kotlin
open class Line() {
    val x = 0
}

class Surface : Line(){
    val y = 1
}
```

즉, Line 이 필요한 곳에 Surface 를 넣어도 문제가 없다면 Surface 는 Line 의 SubType 이라고 할 수 있다.  

``` kotlin
var line = Line()
var surface = Surface()
line = surface // OK!!

var line = Line()
var surface = Surface()
surface = line // ERROR!!
```

이제 본론으로 돌아와 Invariant 에 대해서 알아보면,  
Invariant 라는 건 상속 관계에 하나도 상관없이 딱 자기 자신의 타입만 허용하는 것을 뜻한다.  
**Kotlin** 뿐만 아니라 **Java** 에서도 Type 관계를 별도로 지정해주지 않은 경우, 모든 Generic Class 는 기본적으로 Invariant 로 된다.  

아래 예시를 보면, String 은 Any 의 SubType 이다.  
그래서 String 객체를 Any 객체에 넣어줄 수 있다.  

```kotlin
var str = "String"
var any = Any()
str = any // ERROR!!
any = str // OK!!
```

하지만, MutableList 의 경우를 보면 분명히 Generic 안에 들어가 있는 타입이 SubType 관계임에도 불구하고, 두번 째 줄 부터 error 가 뜨는 것을 확인할 수 있다.  
이는 읽기 및 쓰기가 가능한 MutableList 의 경우 무공변성을 가지기 때문인데, 만약 이를 허용해주게 되면, 4번째 줄에서 런타임 Exception 이 발생하게 된다.  
이를 피해주기 위해서 무공변성을 가지고 있고 이를 통해 개발자들이 좀 더 안전하게 컴파일단에서 에러를 잡을 수 있게끔 도와준다.

```kotlin
val strings: MutableList<String> = mutableListOf("1")
val anys: MutableList<Any> = strings // ERROR!!
anys.add(0)
val str1: String = anys[0] // class cast exception
```

## Covariant(공변)

Covariant 에는 다음과 같은 특징이 있다.

- 모든 읽기가 가능하고 모든 쓰기는 불가능하다. (read-only)
- 자기 자신 혹은 자식의 객체를 허용한다.
- Java 의 `<? extends T>`
- Kotlin 의 out

아래 공식 문서에 나와있는 예시를 보면, 
`interface Source<out T>` 로 만들어 줘서,  
Any 로 타입을 가질려고 하는 objects 에 SubType 인 String 을 타입으로 가지고 있는 strs 를 넣어줄 수 있는 것이다.

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // OK!!
    // ...
}
```

## Contravariant(반변)

Contravariant 에는 다음과 같은 특징이 있다.

- 모든 쓰기가 가능하고 모든 읽기는 불가능하다. (write-only)
- 자기 자신 혹은 부모의 객체를 허용한다.
- Java 의 `<? super T>`
- Kotlin 의 in

아래 공식 문서에 나와있는 예시를 보면,
`interface Comparable<in T>` 로 만들어 줘서,  
1.0 의 Type 인 Double 은 Number 의 SubType 이기 때문에, `x.compareTo(T)` 의 T 안에 넣어줄 수 있고,  
`y: Comparable<Double>` 에다가 `x` 를 넣어줄 수 있다.  

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0)
    val y: Comparable<Double> = x // OK!
}
Copied!

```

## 선언부 Variance

위에서 언급한 예시처럼 out 과 in 키워드를 선언부에서 사용하는 것을 선언부 Variance 라고 한다.  
이를 통해, 형식 인자로 정의한 T에 대해 T를 사용하는 클래스가 T의 producer 로만 쓰일 것인지 consumer로만 쓰일 것인지를 선언부에서 정의를 해줄 수 있다.

## 사용부 Variance

선언부 Variance 가 일반적으로 사용되는 방식이고 굉장히 유용하지만 다음과 같은 경우에는 사용하지 못한다.

```kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { ... }
    fun set(index: Int, value: T) { ... }
}
```
T라는 형식 인자가 함수의 인자로도 들어가고 `fun set(index: Int, value: T) { ... }`  
함수의 반환값으로도 들어가기 때문에 `fun get(index: Int): T { ... }`  
위와 같은 경우에는 covariant(공변), contravariant(반변) 둘 다 사용이 불가능하다.  
그렇다고 사용을 안한채로 invariant(불변) 인 상태로 남겨두면 다음과 같은 불편함이 있다.

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any)
//   ^ type is Array<Int> but Array<Any> was expected
```
copy는 ClassCastException을 유발하지는 않지만 명확히 그렇다는걸 컴파일러에게 전달해줄 수 있어야 한다.  
그렇지 않으면 위처럼 컴파일 에러가 발생하게 된다.  
이 때 사용하는 것이 사용부 variance 이다.  

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```
위와 같은 선언은 from 매개인자가 producer로써만 사용될 수 있다는 것을 명시해주기 때문에, 강제적으로 copy내에서는 consume하는 메소드(set)들의 사용이 제한된다.

## Star-projections

`*` 을 이용한 projection은 다음 예시와 같이 가능한 모든 타입을 사용할 수 있게 해준다.  

```kotlin
interface Function<in T, out U>
Function<*, String> = Function<in Nothing, String>
Function<Int, *> = Function<Int, out Any?>
Function<*, *> =Function<in Nothing, out Any?>
```

covariant 는 `*` 를 만나면 `out Any?` 로 변하고, contravariant는 `*`를 만나면 `in Nothing` 으로 변한다.  
즉, 타입을 모를 때 쓸 수는 있지만 우리가 projection 을 적용시켜놨기 때문에 out 에서는 set 같은 consume 메소드를 사용할 수 없게 되고, in 에서는 get 과 같은 produce 메소드를 사용할 수 없게 된다.

## Why in, out?

그렇다면 Kotlin 에서는 왜 Convariant 와 Contravariant 에 out과 in 이라는 키워드를 사용할까?  
일단, C#에서도 Kotlin과 같이 convariant 와 contravariant 를 out, in 키워드로 사용한는데,  
공식 홈페이지에서 아래와 같이 설명하고 있는 것을 보면 Kotlin 에서 이 개념을 그대로 가져온듯 하다.  

>The words in and out seem to be self-explanatory (as they’ve already been used successfully in C# for quite some time)

여기서 Joshua Bloch는 convariant(out) 와 contravariant(in) 의 역할에 대해서 다음과 같이 정리했다. 

>PECS (Producer Extends Consumer Super)
>Producers에서 읽고, Consumers에서 쓴다.  
>제네릭의 최대의 유용성을 위해, 인자를 producers와 consumers로 구분하여 사용하라.

즉, producer 와 consumer 의 관계는 아래와 같이 정리할 수 있다.
- Producer == out == 읽기
- Consumer == in == 쓰기



