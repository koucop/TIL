# Arrays

Kotlin 에서 Array(배열) 은 Array 클래스로 표현된다.  
Arrays 에서는 operator overloading conventions 를 통해 get, set 을 `[]` 로 나타낼 수 있고, `size` 와 같은 property 를 가지고 있다.  

``` kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

Array 를 만들기 위해서 arrayOf() 를 사용할 수 있다.  
arrayOf(1, 2, 3) 을 해주면, `[1, 2, 3]` 에 상응하는 배열을 만들 수 있다.  
또한 부가적으로 arrayOfNulls() 를 해주면 각 요소들이 null 로 채워져 있는 배열로도 만들 수 있다.  
그리고 아래와 같이 각 index 에 상응하는 값을 넣어주도록 만들 수도 있다.

``` kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { println(it) }
```

### Primitive type arrays

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]

val arr0 = IntArray(5) // [0, 0, 0, 0, 0]
val arr1 = IntArray(5) { 42 } // [42, 42, 42, 42, 42]
var arr2 = IntArray(5) { it * 1 } // [0, 1, 2, 3, 4]
```


Arrays in Kotlin are represented by the Array class. It has get and set functions that turn into [] by operator overloading conventions, and the size property, along with other useful member functions:
