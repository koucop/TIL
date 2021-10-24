# ComposeState

이번 문서는 [Android-Compose 공식문서](https://developer.android.com/jetpack/compose/state) 기반으로 작성하였다.

## 개요

상태는 시간이 지남에 따라 변할 수 있는 값이다.  
이는 매우 광범위한 정의로서 Room 데이터베이스부터 클래스안에 있는 변수까지 모든 항목이 포함된다.

모든 Android 앱에서는 사용자에게 상태가 표시되는데, 다음은 Android 앱 상태의 몇 가지 예시이다.

- 네트워크 연결을 설정할 수 없을 때 표시되는 스낵바
- 블로그 게시물 및 관련 댓글
- 사용자가 클릭하면 버튼에서 재생되는 ripple animations (물결 애니메이션)
- 사용자가 이미지 위에 그릴 수 있는 스티커

Jetpack Compose를 사용하면 Android 앱에서 상태를 저장하고 사용하는 위치와 방법을 명시적으로 나타낼 수 있는데,  
이 문서에서는 State 와 Composable 간의 관계에 관해 그리고 보다 손쉬운 상태 처리를 위해 Jetpack Compose에서 제공되는 API에 관해 집중적으로 설명하려고 한다.

## State and Composition (상태 및 컴포지션)

Compose는 선언적이므로 Compose를 업데이트하는 유일한 방법은 새 arguments(인수)로 동일한 composable 을 호출하는 것이다.  
이러한 인수는 UI 의 상태를 표현하는데, 해당 상태가 업데이트될 때마다 recomposition(재구성) 이 실행된다.  
따라서 TextField와 같은 항목은 명령형 XML 기반 뷰에서처럼 자동으로 업데이트되지 않고, composable 이 새 상태에 따라 업데이트되려면 새 상태를 명시적으로 알려줘야 한다.

``` kotlin
@Composable
fun HelloContent() {
   Column(modifier = Modifier.padding(16.dp)) {
       Text(
           text = "Hello!",
           modifier = Modifier.padding(bottom = 8.dp),
           style = MaterialTheme.typography.h5
       )
       OutlinedTextField(
           value = "",
           onValueChange = { },
           label = { Text("Name") }
       )
   }
}
```

위 코드를 실행하면 아무 일도 일어나지 않는다.  
TextField가 자체적으로 업데이트되지 않기 때문이고, value 매개변수가 변경될 때 업데이트된다.  
이는 Compose에서의 composition 및 recomposition 작동 방식 때문이다.

### 핵심용어
> Composition(컴포지션): Jetpack Compose가 composable 을 실행할 때 빌드한 UI에 관한 설명  
> Initial Composition(초기 컴포지션): 처음 composable 을 실행하여 composition을 만든다.  
> Recomposition(리컴포지션): 데이터가 변경될 때 composition 을 업데이트하기 위해 composable 을 다시 실행하는 것을 말한다.

## State in composables(컴포저블의 상태)

Composable 함수는 `remeber` composable 을 사용하여 메모리에 단일 객체를 저장할 수 있다.  
`remember` 에 의해 계산된 값은 `Initial Composition` 에 저장되고 저장된 값은 `Recomposition` 중에 반환된다.  
`remember` 는 변경 가능한 객체뿐만 아니라 변경할 수 없는 객체를 저장하는데에도 사용할 수 있다.

### 참고
> `remember` 는 객체를 composition 에 저장하고, `remember`를 호출한 composable 이 composition 에서 삭제되면 그 객체 또한 잊어버린다.  
> mutableStateOf는 관찰 가능한 MutableState<T>를 생성하고, 이는 런타임 시 Compose에 통합되는 관찰 가능 유형이다.

``` kotlin
interface MutableState<T> : State<T> {
    override var value: T
}
```

`value`가 변경되면 `value`를 읽는 composable 함수의 recomposition 이 예약(schedulbe)된다.  
예를 들어, ExpandingCard의 경우 expanded가 변경될 때마다 ExpandingCard가 재구성된다.

Composable 에서 MutableState 객체를 선언하는 데는 세 가지 방법이 있다.

``` kotlin
val mutableState = remember { mutableStateOf(default) }
var value by remember { mutableStateOf(default) }
val (value, setValue) = remember { mutableStateOf(default) }
```

이러한 선언은 동일한 것이며 서로 다른 용도의 상태를 사용하기 위해 제공되고,  
작성 중인 composable 에서 가장 읽기 쉬운 코드를 생성하는 선언을 선택해야 한다.
여기서, by 위임 구문(delegate syntax) 에는 다음의 imports 가 필요하다.

``` kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
```

`remember` 된 값을 다른 composable 의 매개변수로 사용하거나 로직으로 사용하여 표시할 composable 을 변경할 수 있다.  
예를 들어 이름이 비어 있는 경우 인사말을 표시하지 않으려면 if 문에 상태를 사용하면 된다.

``` kotlin
@Composable
fun HelloContent() {
   Column(modifier = Modifier.padding(16.dp)) {
       var name by remember { mutableStateOf("") }
       if (name.isNotEmpty()) {
           Text(
               text = "Hello, $name!",
               modifier = Modifier.padding(bottom = 8.dp),
               style = MaterialTheme.typography.h5
           )
       }
       OutlinedTextField(
           value = name,
           onValueChange = { name = it },
           label = { Text("Name") }
       )
   }
}
```

`remember` 가 recomposition 과정 전체에서 상태를 유지하는 데 도움은 되지만 composable 전반에서는 상태가 유지되지 않는다.  
이 경우에는 rememberSaveable을 사용해야 하는데, rememberSaveable은 Bundle에 저장할 수 있는 모든 값을 자동으로 저장한다.  
다른 값들의 경우에는 Custom Saver 객체를 전달할 수 있다.

