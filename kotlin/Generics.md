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

Type Bound 종류 에는 아래 3가지가 있고, 이번 문서에서는 각 내용에 대해 정리해보려고 한다.

- Invariance (불변성)
- covariant (공변성)
- contravariant (반공변성)

## Invariance (불변성)



