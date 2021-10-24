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
> mutableStateOf는 관찰 가능한 `MutableState<T>`를 생성하고, 이는 런타임 시 Compose에 통합되는 관찰 가능 유형이다.

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

## Other supported types of state(지원되는 기타 상태 유형)

Jetpack Compose 에서는 `State`를 보존하기 위해서 반드시 `MutableState<T>` 를 사용할 필요가 없다.  
Jetpack Compose 는 관찰 가능한 다른 유형을 지원하는데, 이 때 Compose 에서 관찰 가능한 다른 유형을 읽으려면 상태를 `State<T>`로 변환해야 한다.  
그래야 상태가 변할 때 Jetpack Compose가 자동으로 재구성된다.

Compose에는 다음의 Android 앱에 사용되는 관찰 가능한 일반 유형에 대해 `State<T>` 를 만들 수 있는 함수가 기본적으로 내장되어 있다.

- LiveData
- Flow
- RxJava2

앱에 관찰 가능한 커스텀 클래스(observable custom class)를 사용하는 경우 관찰 가능한 다른 유형을 읽어오기 위한 Compose용 확장 함수(Extension function)를 빌드할 수 있다.  
모든 변경사항을 수신하도록 Compose 에 허용하는 모든 객체를 `State<T>` 로 변환하고 composable 을 통해 읽어올 수 있다.

### 요점
> Compose는 `State<T>` 객체를 읽어오면서 자동으로 재구성된다.  
> Compose에서 `LiveData` 같은 관찰 가능한 또 다른 유형을 사용할 경우, composable 에서 `LiveData<T>.observeAsState()` 같은 composable 확장 함수(Extension function)를 사용하여 그 유형을  `State<T>` 로 변환한 후에 읽어올 수 있다.

### 주의
> Compose에서 `ArrayList<T>` 또는 `mutableListOf()` 같은 변경 가능 객체를 상태로 사용하면 앱에 잘못되거나 오래된 데이터가 표시될 수 있다.  
> 변경 가능한 객체 중 `ArrayList<T>` 또는 변경 가능한 데이터 클래스 같은 관찰 불가능한 객체는 그 객체가 변경될 때 composition 을 트리거하도록 Compose 에서 관찰할 수 없다.  
> 관찰 불가능하면서 변경 가능한 객체를 사용하는 대신 `State<List<T>>` 및 변경 불가능한 listOf() 같은 관찰 가능 데이터 홀더를 사용하는 것이 좋다.

## Stateful and stateless(스테이트풀과 스테이트리스)

`remember` 를 사용하여 객체를 저장하는 composable 은 내부 상태를 생성하여 composable 을 스테이트풀(Stateful)로 만든다.  
HelloContent는 내부적으로 name 상태를 보존하고 수정하므로 스테이트풀(Stateful) 컴포저블의 한 예가 된다.  
이는 호출자(caller)가 상태를 제어할 필요가 없고 상태를 직접 관리하지 않아도 상태를 사용할 수 있는 경우에 유용하다.  
그러나 내부 상태를 갖는 컴포저블은 재사용 가능성이 적고 테스트하기가 더 어려운 경향이 있다.

스테이트리스(Stateless) composable 은 상태를 갖지 않는 친구이다. 스테이트리스(Stateless)를 달성하는 한 가지 쉬운 방법은 `State hoisting` 을 사용하는 것이다.

재사용 가능한 composable 을 개발할 때는 동일한 컴포저블의 스테이트풀(Stateful) 버전과 스테이트리스(Stateless) 버전을 모두 노출해야 하는 경우가 있다.  
스테이트풀(Stateful) 버전은 상태를 염두에 두지 않는 호출자(caller)에 편리하며, 스테이트리스(Stateless) 버전은 상태를 제어하거나 끌어올려야 하는 호출자(caller)의 경우에 필요하다.

## 상태 호이스팅

Compose에서 state hoisting 은 composable 을 스테이트리스(Stateless)로 만들기 위해 state 를 composable 의 호출자(caller)로 옮기는 패턴이다.  
Compose에서 state hoisting 을 위한 일반적 패턴은 상태 변수를 다음 두 개의 매개변수로 바꾸는 것이다.

- `value`: T: 표시할 현재 값
- `onValueChange`: (T) -> Unit: T가 제안된 새 값인 경우 값을 변경하도록 요청하는 이벤트

하지만 이는 `onValueChange` 로만 제한되지 않는다.  
컴포저블에 더 특정한 이벤트가 어울리는 경우, 예를 들어, ExpandingCard가 onExpand와 onCollapse를 정의할 때와 같이 람다를 사용하여 그 이벤트를 정의할 수 있다.

이러한 방식으로 끌어올린(hoisting) 상태에는 중요한 속성이 몇 가지 있다.

- Single source of truth(단일 소스 저장소): 상태를 복제하는 대신 옮겼기 때문에 소스 저장소가 하나만 있다. 버그 방지에 도움이 된다.
- Encapsulated(캡슐화됨): 스테이트풀(Stateful) composable 만 상태를 수정할 수 있다. 철저히 내부적 속성이다.
- Shareable(공유 가능함): 호이스팅한 상태를 여러 composable 과 공유할 수 있다. 다른 composable에서 name을 사용하려는 경우 호이스팅을 통해 그렇게 할 수 있다.
- Interceptable(가로채기 가능함): 스테이트리스(Stateless) composable의 호출자(caller)는 상태를 변경하기 전에 이벤트를 무시할지 수정할지 결정할 수 있다.
- Decoupled(분리됨): 스테이트리스(Stateless) ExpandingCard의 상태는 어디에나 저장할 수 있다. 예를 들어 name을 ViewModel로 옮길 수 있다.

아래의 예시 에서는 `HelloContent`에서 `name`과 `onValueChange`를 추출한 다음,  
이러한 항목을 트리 상단을 거쳐 `HelloContent`를 호출하는 `HelloScreen` composable 로 옮긴다.

``` kotlin
@Composable
fun HelloScreen() {
    var name by rememberSaveable { mutableStateOf("") }

    HelloContent(name = name, onNameChange = { name = it })
}

@Composable
fun HelloContent(name: String, onNameChange: (String) -> Unit) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello, $name",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.h5
        )
        OutlinedTextField(
            value = name,
            onValueChange = onNameChange,
            label = { Text("Name") }
        )
    }
}
```

HelloContent에서 상태를 끌어올리면(hoisting) 더 쉽게 composable 을 추론하고 여러 상황에서 재사용하며 테스트할 수 있다.  
HelloContent는 상태의 저장 방식과 디커플링(분리)된다.  
디커플링 된다는 것은 HelloScreen을 수정하거나 교체할 경우 HelloContent의 구현 방식을 변경할 필요가 없다는 의미이다.

![Image](/assets/udf-hello-screen.png)

상태가 내려가고 이벤트가 올라가는 패턴을 *단방향 데이터 흐름*이라고 한다.  
이 경우 상태는 HelloScreen에서 HelloContent로 내려가고 이벤트는 HelloContent에서 HelloScreen으로 올라간다.  
단방향 데이터 흐름을 따르면 UI에 상태를 표시하는 composable 과 상태를 저장하고 변경하는 앱 부분을 서로 분리할 수 있다.

### 핵심 사항
> 상태를 끌어올릴 때 상태의 이동 위치를 쉽게 파악할 수 있는 세 가지 규칙이 있다.  
> 
> 1. 상태는 적어도 그 상태를 사용하는 모든 컴포저블의 가장 낮은 공통 상위 요소로 끌어올려야 한다(읽기).
> 2. 상태는 최소한 변경될 수 있는 가장 높은 수준으로 끌어올려야 한다(쓰기).
> 3. 동일한 이벤트에 대한 응답으로 두 상태가 변경되는 경우 두 상태를 함께 끌어올려야 한다.
> 
> 이러한 규칙에서 요구하는 것보다 상태를 더 높은 수준으로 끌어올릴 수 있다.  
> 하지만 상태를 끌어내리면 단방향 데이터 흐름을 따르기가 어렵거나 불가능할 수 있다.

## Restoring state in Compose(Compose 에서 state 복원)

`remeberSaveable` 을 사용해서 액티비티 또는 프로세스가 재생성됐을 때, UI 상태를 복원할 수 있다.  
`remeberSaveable` 는 recomposition 될 때에 상태를 유지한다.  
더해서, `rememberSaveable` 은 액티비티와 프로세스가 재생성될 때 또한 상태를 유지한다.

### 상태를 저장하는 방법

`Bundle` 에 추가된 모든 data 타입들은 자동적으로 저장된다.  
만약 `Bundle`에 추가되기 어려운 무언가를 저장하기 원한다면, 다음과 같은 옵션이 있다.  

1. Parcelize
2. MapSaver
3. ListSaver

**1.Parcelize**

객체에 `@Parcelize` annotation 을 추가하면 가장 쉽게 상태를 저장할 수 있다.  
이를 통해 객체는 parcelable 이 되고, bundled 될 수 있다.  
예를 들어, 아래 코드는 City 데이터 타입을 parcelable 로 만들고 이를 상태에 저장시킨다.

``` kotlin
@Parcelize
data class City(val name: String, val country: String) : Parcelable

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

**2.MapSaver**

가끔 `@Parcelize` 가 안어울리는 경우에는, mapSaver 를 사용해서 객체를 converting 하는 custom 룰을 만들어서 시스템으로 하여금 bundle 에 저장시킬 수 있도록 만들 수 있다.

``` kotlin
data class City(val name: String, val country: String)

val CitySaver = run {
    val nameKey = "Name"
    val countryKey = "Country"
    mapSaver(
        save = { mapOf(nameKey to it.name, countryKey to it.country) },
        restore = { City(it[nameKey] as String, it[countryKey] as String) }
    )
}

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

**3.ListSaver**

map 으로 만들어서 반드시 key 를 만드는게 싫다면, listSaver 를 사용할 수 있다.

``` kotlin
data class City(val name: String, val country: String)

val CitySaver = listSaver<City, Any>(
    save = { listOf(it.name, it.country) },
    restore = { City(it[0] as String, it[1] as String) }
)

@Composable
fun CityScreen() {
    var selectedCity = rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

## Managing state in Compose(Compose 에서 상태 관리)

간단한 상태 호이스팅은 composable 함수 자체에서 관리할 수 있다.  
그러나 추적할 상태의 양이 증가하거나 composable 기능에서 수행할 로직이 생기는 경우 로직 및 상태에 대한 책임을 다른 클래스인 state holders 에게 위임하는 것이 좋다.

### 주요 용어
> State holders: composable 의 로직 및 상태를 관리합니다.
> 다른 곳에서는 state holders 를 *hoisted state objects*라고도 부릅니다.

지금부터 Compose에서 다양한 방식으로 상태를 관리하는 방법에 대해 알아보자.  
Composable 의 복잡성에 따라 다음과 같이 상태관리를 하는게 좋다.

- 간단한 UI 요소 상태 관리를 위한 composable.
- 복잡한 UI 요소 상태 관리를 위한 state holders. State holders 는 UI 요소의 상태와 UI 로직을 소유합니다.
- [Architecture Components ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel) 는 비즈니스 로직과 화면 또는 UI 상태에 대한 액세스를 제공하는 특별한 유형의 state holders 이다.

State holders 는 하단 앱 바와 같은 단일 위젯에서 전체 화면에 이르기까지 관리하는 해당 UI 요소의 범위에 따라 다양한 크기로 제공된다.  
State holders 는 복합 가능하다. 즉, 상태 보유자는 특히 상태를 집계할 때 다른 state holders 에 통합될 수 있다.

다음 다이어그램은 Compose 상태 관리와 관련된 entities 간의 관계에 대한 요약을 보여준다.  

- Composable 은 복잡성에 따라 0개 이상의 state holder(일반 객체, ViewModel 또는 둘 다일 수 있음)에 종속될 수 있다.
- 일반 state holders 는 비즈니스 로직 또는 화면 상태에 대한 접근이 필요한 경우 ViewModel에 의존할 수 있다.
- ViewModel은 비즈니스 또는 데이터 계층에 따라 다르다.

![Image](/assets/state-dependencies.svg)

### Types of state and logic(상태와 로직의 유형)

Android 앱에서는 다양한 유형의 상태를 고려해야 한다.

- **UI element state** 는 UI 요소의 호이스트 상태이다. 예를 들어 ScaffoldState는 Scaffold composable 상태를 처리한다.
- **Screen or UI state** 는 화면에 표시되어야 하는 상태이다. 예를 들어 CartUiState 클래스에서는 장바구니 항목, 사용자에게 표시할 메시지 또는 로드 플래그를 포함한다. 이 상태는 애플리케이션 데이터를 포함하기 때문에 일반적으로 계층의 다른 계층과 연결된다.

또한 다양한 유형의 로직이 있다.

- UI behavior logic or UI logic 은 화면에 상태 변경 사항을 표시하는 방법과 관련이 있다. 예를 들어 navigation logic 은 다음에 표시할 화면을 결정하거나 스낵바 또는 토스트를 사용할 수 있는 화면에 사용자 메시지를 표시하는 방법을 결정하는 UI 로직이다. UI behavior logic 은 항상 Composition 에 있어야 합니다.
- Business logic 은 상태 변경을 처리하는 것이다. 예를 들어 결제를 하거나 사용자 기본 설정을 저장한다. 이 로직은 일반적으로 UI 계층이 아닌 비즈니스 또는 데이터 계층에 배치된다.

### Composables as source of truth

UI 로직과 UI 요소 상태를 composable 에 두는 것은 상태와 로직이 단순한 경우 좋은 접근 방식이다.  
예를 들어 다음은 ScaffoldState 및 CoroutineScope를 처리하는 MyApp composable 이다.

```kotlin
@Composable
fun MyApp() {
    MyTheme {
        val scaffoldState = rememberScaffoldState()
        val coroutineScope = rememberCoroutineScope()

        Scaffold(scaffoldState = scaffoldState) {
            MyContent(
                showSnackbar = { message ->
                    coroutineScope.launch {
                        scaffoldState.snackbarHostState.showSnackbar(message)
                    }
                }
            )
        }
    }
}
```

ScaffoldState에는 변경 가능한 속성이 포함되어 있기 때문에 모든 상호 작용은 MyApp 구성 가능에서 발생해야 한다.  
그렇지 않고 다른 composable 에 전달하면 single source of truth principle 을 준수하지 않고 버그 추적을 더 어렵게 만드는 상태로 변경되 수 있다.

### State holders as source of truth

Composable 에 여러 UI 요소의 상태를 포함하는 복잡한 UI 로직이 포함된 경우, 해당 책임을 state holders 에게 위임해야 한다.  
이렇게 하면 이 로직을 격리된 상태에서 더 쉽게 테스트할 수 있고 composable 의 복잡성을 줄일 수 있다.  
이 접근 방식은 관심 분리 원칙을 선호하고, Composable 은 UI 요소 방출을 담당하고 state holder 는 UI 로직과 UI 요소의 상태를 포함한다.

State holders 는 composition에서 생성되고 기억되는 일반 클래스이고, 때문에 Composable 의 수명 주기를 따르기 때문에 Compose 종속성을 사용할 수 있다.

Composables as the source of truth 섹션의 MyApp 컴포저블에 대한 책임이 커지면 MyAppState 상태 홀더를 만들어 복잡성을 관리할 수 있다.

``` kotlin
// Plain class that manages App's UI logic and UI elements' state
class MyAppState(
    val scaffoldState: ScaffoldState,
    val navController: NavHostController,
    private val resources: Resources,
    /* ... */
) {
    val bottomBarTabs = /* State */

    // Logic to decide when to show the bottom bar
    val shouldShowBottomBar: Boolean
        @Composable get() = /* ... */

    // Navigation logic, which is a type of UI logic
    fun navigateToBottomBarRoute(route: String) { /* ... */ }

    // Show snackbar using Resources
    fun showSnackbar(message: String) { /* ... */ }
}

@Composable
fun rememberMyAppState(
    scaffoldState: ScaffoldState = rememberScaffoldState(),
    navController: NavHostController = rememberNavController(),
    resources: Resources = LocalContext.current.resources,
    /* ... */
) = remember(scaffoldState, navController, resources, /* ... */) {
    MyAppState(scaffoldState, navController, resources, /* ... */)
}
```

MyAppState는 종속성을 사용하므로 컴포지션에서 MyAppState의 인스턴스를 기억하는 메서드를 제공하는 것이 좋다. 이 경우에는 RememberMyAppState 함수이다.

이제 MyApp은 UI 요소를 내보내는 데 중점을 두고 있으며 모든 UI 논리 및 UI 요소의 상태를 MyAppState에 위임한다.

```
@Composable
fun MyApp() {
    MyTheme {
        val myAppState = rememberMyAppState()
        Scaffold(
            scaffoldState = myAppState.scaffoldState,
            bottomBar = {
                if (myAppState.shouldShowBottomBar) {
                    BottomBar(
                        tabs = myAppState.bottomBarTabs,
                        navigateToRoute = {
                            myAppState.navigateToBottomBarRoute(it)
                        }
                    )
                }
            }
        ) {
            NavHost(navController = myAppState.navController, "initial") { /* ... */ }
        }
    }
}
```

위와 같이, composable 의 책임을 늘리면 state holders 가 필요하다.=

### 참고

> State holders 가 액티비티 또는 프로세스를 다시 만든 후 보존하려는 상태를 포함하는 경우, `rememberSaveable`를 사용하고 이에 대한 custom Saver를 된다.

### ViewModels as source of truth

일반적인 state holders 클래스가 UI 로직 및 UI 요소의 상태를 담당하는 경우 ViewModel은 다음을 담당하는 특수한 유형의 state holders 이다.
- 일반적으로 비즈니스 및 데이터 계층과 같은 계층의 다른 계층에 배치되는 애플리케이션의 비즈니스 로직에 대한 액세스를 제공하고,
- 화면 또는 UI 상태가 되는 특정 화면에 표시하기 위해 애플리케이션 데이터를 준비한다.

ViewModel은 configuration 변경 후에도 유지되기 때문에 Composition 보다 수명이 더 길다.  
이들은 Compose 콘텐츠 호스트의 수명 주기(즉, activities or fragments) 또는 대상의 수명 주기 또는 navigation 라이브러리를 사용하는 경우 navigation-graph를 따를 수 있다.  
수명이 더 길기 때문에 ViewModel은 composition의 수명에 바인딩된 상태에 대한 수명이 긴 참조를 보유하지 않아야 한다. 혹시, 그럴 경우 메모리 누수가 발생할 수 있다.

비즈니스 로직에 대한 액세스를 제공하고 UI 상태에 대한 정보 소스가 되기 위해 ViewModel을 사용하려면 화면 수준 composable 을 사용하는 것이 좋다.  
다음은 화면 수준 composable 에서 사용되는 ViewModel의 예시이다.

``` kotlin
data class ExampleUiState(
    dataToDisplayOnScreen: List<Example> = emptyList(),
    userMessages: List<Message> = emptyList(),
    loading: Boolean = false
)

class ExampleViewModel(
    private val repository: MyRepository,
    private val savedState: SavedStateHandle
) : ViewModel() {

    var uiState by mutableStateOf<ExampleUiState>(...)
        private set

    // Business logic
    fun somethingRelatedToBusinessLogic() { ... }
}

@Composable
fun ExampleScreen(viewModel: ExampleViewModel = viewModel()) {

    val uiState = viewModel.uiState
    ...

    Button(onClick = { viewModel.somethingRelatedToBusinessLogic() }) {
        Text("Do something")
    }
}
```

> 참고 : ViewModels에 프로세스 재생성 후 보존하려는 상태가 포함되어 있으면 SavedStateHandle을 사용하여 유지한다.

### ViewModel and state holders

Android 개발에서 ViewModels의 이점은 비즈니스 로직에 대한 액세스를 제공하고 화면에 표시할 애플리케이션 데이터를 준비하는 데 적합하다.  
즉, 이점은 다음과 같이 풀어서 쓸 수 있는데,

- ViewModel에 의해 트리거된 작업은 구성 변경 후에도 유지된다.
- Navigation 과의 통합:
   - Navigation 은 화면이 백 스택에 있는 동안 ViewModel을 캐시한다. 목적지로 돌아갈 때 이전에 로드한 데이터를 즉시 사용할 수 있도록 하는 것이 중요하다. 이것은 composable 화면의 수명 주기를 따르는 state holders 로 수행하기 더 어려운 작업이다.
   - 대상이 백 스택에서 팝되면 ViewModel도 지워지므로 상태가 자동으로 정리된다. 이는 구성 변경 등으로 인해 새 화면으로 이동하는 등 여러 가지 이유로 발생할 수 있는 composable 폐기를 바라보고 있는 것과 다르다.
- Hilt와 같은 다른 Jetpack 라이브러리와의 통합.

> 참고: ViewModel 이점이 프로젝트에 맞지 않거나 다른 방식으로 작업을 수행하는 경우 ViewModel의 책임을 state holders 로 이동시킬 수 있다.

State holders 는 복합 가능하고 ViewModel과 일반 state holders 는 책임이 다르기 때문에 화면 수준 composable 에는 비즈니스 논리에 대한 액세스를 제공하는 ViewModel과 UI 로직 및 UI 요소의 상태를 관리하는 state holders 가 모두 있을 수 있다.  
ViewModel은 state holders 보다 수명이 더 길기 때문에 state holders는 필요한 경우 ViewModel을 종속성으로 사용할 수 있다.

다음 코드는 ExampleScreen에서 함께 작동하는 ViewModel 및 일반 state holders 이다.

``` kotlin
private class ExampleState(
    val lazyListState: LazyListState,
    private val resources: Resources,
    private val expandedItems: List<Item> = emptyList()
) { ... }

@Composable
private fun rememberExampleState(...) { ... }

@Composable
fun ExampleScreen(viewModel: ExampleViewModel = viewModel()) {

    val uiState = viewModel.uiState
    val exampleState = rememberExampleState()

    LazyColumn(state = exampleState.lazyListState) {
        items(uiState.dataToDisplayOnScreen) { item ->
            if (exampleState.isExpandedItem(item) {
                ...
            }
            ...
        }
    }
}
```
