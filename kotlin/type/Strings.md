# Strings

Kotlin 에서 String 타입은 따움표 안에 있는 값으로 표현된다.  

```kotlin
val str = "abcd 123"
```

String 의 요소들은 Characters 의 집합이라고 할 수 있다. 때문에 `s[i]` 와 같이 접근할 수 있다.  

```kotlin
for (c in str) {
    println(c)
}
```

String 은 불변이다. 즉, string 을 한 번 초기화 시키면, 다른 값으로 변경할 수 없다.  

```kotlin
val str = "abcd"
println(str.uppercase()) // 새 String 객체를 만든 후 print 함
println(str) // str 객체가 그대로 같은 값으로 남아있음
```

String 값을 엮기 위해서는 `+` operator 를 사용하면 된다.  
만약 처음 요소가 String 타입이었다면, 하나의 String 타입과 다른 타입을 사슬처럼 엮을 수 있다.  

``` kotlin
val s = "abc" + 1
println(s + "def") // abc1def
```

### String literals

Kotlin 에는 다음과 같은 두 개의 String 리터럴이 있다.  

- escaped characters 를 포함한 escaped strings

```kotlin
val s = "Hello, world!\n" // escaped string
```

- newlines 과 임의의 텍스트를 포함한 문자열

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

만약 leading whitespace 를 없애고 싶다면 `trimMargin()` 을 사용하면 되고, prefix 를 부팅고 싶다면 `trimMargin(">")` 과 같이 문자열 값을 파라미터 안에 넣어주면 된다.  

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

### String templates

``` kotlin
val i = 10
println("i = $i") // prints "i = 10"

val s = "abc"
println("$s.length is ${s.length}") // prints "abc.length is 3"

val price = """
${'$'}_9.99
"""
```
