# Characters

Characters 는 Char 로 표현된다.  
Character 리터럴(literals) 은 작은 따옴표로 나타낼 수 있다. (e.g. '1')

특히 Kotlin 의 Char 에서는 다음과 같은 이스케이프 시퀀스(escape sequence) 를 지원한다.  

- \t
- \b
- \n
- \r
- \'
- \"
- \\
- \$

```kotlin
val aChar: Char = 'a'

println(aChar)
println('\n') //prints an extra newline character
```
