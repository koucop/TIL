# [📩 Serialization](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/serialization-guide.md)

*serialization* 은 application 에서 사용하는 데이터를 네트워크를 통해 전송하거나 데이터베이스 또는 파일에 저장할 수 있는 형식으로 변환하는 process 이다.  
*deserialization* 는 외부 소스에서 데이터를 읽고 이를 런타임 개체로 변환하는 반대 process 이다.  
*serialization* 과 *deserialization* 은 데이터를 교환하는 거의 모든 application 에서 필수적인 부분입니다.

[JSON](https://www.json.org/json-en.html) 및 [protocol buffers](https://developers.google.com/protocol-buffers)와 같은 data serialization formats 은 특히 common 하게 사용되어지고 있다.   
부가설명 해보자면 위의 serialization formats 는 언어 중립적이고 플랫폼 중립적이기 때문에 현대 언어로 작성된 시스템 간에 데이터 교환이 가능하다.

Kotlin에서의 데이터 직렬화 도구는 별도의 구성요소인 kotlinx.serialization 으로 사용할 수 있으며,  
Gradle 플러그인인 org.jetbrains.kotlin.plugin.serialization 과 runtime libraries 두 가지 주요 부분으로 구성된다.

## Libraries

kotlinx.serialization은 지원되는 모든 플랫폼(JVM, JavaScript, Native)과 다양한 직렬화 형식(JSON, CBOR, 프로토콜 버퍼 등)에 대한 sets of libraries(라이브러리 세트)를 제공한다.  
아래 Formats 섹션에서 지원되는 직렬화 형식의 전체 목록을 찾을 수 있다.

모든 Kotlin 직렬화 라이브러리는 org.jetbrains.kotlinx: 그룹에 속하며, 이름은 kotlinx-serialization-으로 시작하고 직렬화 형식을 반영하는 접미사가 있다.
- org.jetbrains.kotlinx:kotlinx-serialization-json은 Kotlin project에 대한 JSON 직렬화를 제공한다.
- org.jetbrains.kotlinx:kotlinx-serialization-cbor는 CBOR 직렬화를 제공한다.

플랫폼별 아티팩트는 자동으로 처리되기 때문에, 수동으로 추가할 필요가 없고, JVM, JS, Native 및 다중 플랫폼 프로젝트에서 동일한 종속성을 사용하면 된다.

kotlinx.serialization 라이브러리는 Kotlin의 버전 관리와 일치하지 않는 자체 버전 관리 구조를 사용하며, 최신 버전을 찾으려면 [GitHub의 release](https://github.com/Kotlin/kotlinx.serialization/releases)를 확인하면 된다.

## Formats

kotlinx.serialization에는 다양한 직렬화 형식에 대한 라이브러리가 포함되어 있다.

- JSON: kotlinx-serialization-json
- protocol buffers: kotlinx-serialization-protobuf
- CBOR: kotlinx-serialization-cbor
- properties: kotlinx-serialization-properties
- HOCON: kotlinx-serialization-hocon(only on JVM)

JSON 직렬화(kotlinx-serialization-core)를 제외한 모든 라이브러리는 [Experimental](https://kotlinlang.org/docs/components-stability.html)이므로 예고 없이 API가 변경될 수 있다.

YAML 또는 Apache Avro와 같이 더 많은 serialization formats 를 지원하는 community-maintained libraries 도 있다.  
사용 가능한 직렬화 형식에 대한 자세한 내용은 [kotlinx.serialization 문서](https://github.com/Kotlin/kotlinx.serialization/blob/master/formats/README.md)를 참조하면 된다.

## Example: JSON serialization

*serialization* 을 사용해서 Kotlin 객체를 JSON으로 직렬화하는 방법

시작하기 전에 프로젝트에서 Kotlin serialization tools 을 사용할 수 있도록 빌드 스크립트를 구성해야 한다.

1. Kotlin 직렬화 Gradle 플러그인 org.jetbrains.kotlin.plugin.serialization 을 적용한다

```gradle
plugins {
    id 'org.jetbrains.kotlin.jvm' version '1.5.31'
    id 'org.jetbrains.kotlin.plugin.serialization' version '1.5.31'
}
```

2. JSON 직렬화 라이브러리 종속성 추가: org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0

```gradle
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-serialization-json:1.3.0'
}
```

이제 코드에서 직렬화 API를 사용할 준비가 되었고, API는 kotlinx.serialization 패키지와 kotlinx.serialization.json과 같은 형식별 하위 패키지에 있다.

3. @Serializable 주석을 달아 클래스를 직렬화 가능하게 만든다
```kotlin
import kotlinx.serialization.Serializable

@Serializable
data class Data(val a: Int, val b: String)
```

4. Json.encodeToString()을 호출하여 이 클래스의 인스턴스를 직렬화한다
```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import kotlinx.serialization.encodeToString

@Serializable
data class Data(val a: Int, val b: String)

fun main() {
   val json = Json.encodeToString(Data(42, "str"))
}
```

5. JSON 형식으로 json 객체의 상태를 포함하는 문자열을 얻을 수 있다
`{"a": 42, "b": "str"}`

6. 한 번의 호출로 list 와 같은 collections 를 직렬화할 수 있다
```kotlin
val dataList = listOf(Data(42, "str"), Data(12, "test"))
val jsonList = Json.encodeToString(dataList)
```

7, JSON에서 객체를 역직렬화하려면 decodeFromString() 함수를 사용하면 된다
```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import kotlinx.serialization.decodeFromString

@Serializable
data class Data(val a: Int, val b: String)

fun main() {
   val obj = Json.decodeFromString<Data>("""{"a":42, "b": "str"}""")
}
```
