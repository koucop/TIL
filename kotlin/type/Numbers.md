# [Numbers](https://kotlinlang.org/docs/basic-types.html#numbers)

## Integer types (정수형 타입들)

### types

| Type | Size(bits) | Min value | Max value |
| --- | --- | --- | --- |
| Byte | 8 | -128 | 127 |
| Short | 16 | -32768 | 32767 |
| Int | 32 | -2,147,483,648 (-2^31) | 2,147,483,647 (2^31 - 1) |
| Long | 64 | -9,223,372,036,854,775,808 (-2^63) | 9,223,372,036,854,775,807 (2^63- 1) |

### usages

```kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1
```

## Floating-point types (소수형 타입들)

### types

| Type | Size (bits) | Significant bits | Exponent bits | Decimal digits |
| --- | --- | --- | --- | --- |
| Float | 32 | 24 | 8 | 6-7 |
| Double | 64 | 53 | 11 | 15-16 |

### usages

```kotlin
val pi = 3.14 // Double
// val one: Double = 1 // Error: type mismatch
val oneDouble = 1.0 // Double

val e = 2.7182818284 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817

fun main() {
    fun printDouble(d: Double) { print(d) }

    val i = 1
    val d = 1.0
    val f = 1.0f

    printDouble(d)
//    printDouble(i) // Error: Type mismatch
//    printDouble(f) // Error: Type mismatch
}
```

## Literal constants (리터럴 상수)

Kotlin 에서는 정수형 값들에 대해서, 다음과 같이 리터럴 상수를 지원합니다.

- Decimals: 123
  - Longs are tagged by a capital L: 123L
- Hexadecimals: 0x0F
- Binaries: 0b00001011
>`0` prefix 를 붙여서 사용하는 octal literal 은 지원하지 않습니다.

Kotlin 에서는 소수형 값들에 대해서, 기존의 표기법(conventional notation)을 지원합니다.

- Doubles by default: 123.5, 123.5e10
- Floats are tagged by f or F: 123.5f

`UnderScore(_)` 를 사용해서 readability 를 늘릴 수 있습니다.

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

## Numbers representation on the JVM (JVM 에서의 Numbers)

JVM 플랫폼에서, numbers 는 int, double 과 같은 primitive types 로 저장된다.  
여기서 중요한 포인트는 nullable 객체는 같은 number 일지라도 다른 객체로 판단될 수 있다는 점이다.  
아래 예시와 같이 결과가 나오는 이유는, a 의 경우 -128 과 127 사이의 값이므로 JVM 에서 동일한 객체로 인식할 수 있기 때문이다.  
하지만 b 의 경우에는 해당사항이 없기 때문에 다른 객체로 인식된다.  

```kotlin
val a: Int = 100
val boxedA: Int? = a
val anotherBoxedA: Int? = a

val b: Int = 10000
val boxedB: Int? = b
val anotherBoxedB: Int? = b

println(boxedA === anotherBoxedA) // true
println(boxedB === anotherBoxedB) // false
```

객체가 다를 뿐, 두 값은 같다는 것을 알 수 있다.
```kotlin
val b: Int = 10000
println(b == b) // Prints 'true'
val boxedB: Int? = b
val anotherBoxedB: Int? = b
println(boxedB == anotherBoxedB) // Prints 'true'
```

## Explicit conversions (명시적 변환)

작은 타입들이 큰 타입들의 SubType 이라고 볼 수는 없다. 즉, Int 보다 Long 이 더 넓은 개념이라고 해서, Int 가 Long 의 SubType 은 아니다.

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(b == a) // Surprise! This prints "false" as Long's equals() checks whether the other is Long as well
```

이는 Byte 와 Int 의 관계에서도 동일하다.

```kotlin
val b: Byte = 1 // OK, literals are checked statically
// val i: Int = b // ERROR
val i1: Int = b.toInt()
```

모든 Number 타입들에서는 다른 타입으로의 전환이 가능하다.

- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

아래와 같이 타입 추론이 가능한 경우에는 별도로 전환을 적용시켜줄 필요가 없다.

```kotlin
val l = 1L + 3 // Long + Int => Long
```

## Operations

Kotlin Number 에서는 다음의 Operator 를 제공한다.  

- `+` 
- `-`
- `*`
- `/`
- `%`

### usages

```kotlin
println(1 + 2)
println(2_500_000_000L - 1L)
println(3.14 * 2.71)
println(10.0 / 3)
```

### Division of integers

- 정수끼리 나누면 타입에 맞게 정수 값이 나온다.

```kotlin
val x = 5 / 2
//println(x == 2.5) // ERROR: Operator '==' cannot be applied to 'Int' and 'Double'
println(x == 2)

val x = 5L / 2
println(x == 2L)
```

- 소수형을 하나라도 넣어주게 되면 소수값이 나온다.

```kotlin
val x = 5 / 2.toDouble()
println(x == 2.5)
```

### Bitwise operations

Kotlin 에서는 정수형에 한해서 *Bitwise 연산자*를 제공한다.

- shl(bits) – signed shift left
- shr(bits) – signed shift right
- ushr(bits) – unsigned shift right
- and(bits) – bitwise and
- or(bits) – bitwise or
- xor(bits) – bitwise xor
- inv() – bitwise inversion

### Floating-point numbers comparison

소수형에는 다음과 같은 방법으로 두 값을 비교할 수 있다.

- Equality checks: `a == b` and `a != b`
- Comparison operators: `a < b`, `a > b`, `a <= b`, `a >= b`
- Range instantiation and range checks: `a..b`, `x in a..b`, `x !in a..b`

여기서 반드시 알아두어야 할 부분은 

- `NaN` 자기 자신과 같을 수 잇다.
- `NaN` 는 `POSITIVE_INFINITY` 를 포함한 어떤 다른 요소들보다 크다.
- `-0.0` 은 `0.0` 보다 작다.

