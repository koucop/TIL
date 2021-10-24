# ComposeBase

이번 문서는 [Android-Compose 공식문서](https://developer.android.com/jetpack/compose/mental-model) 기반으로 작성하였다.

## 개요

Jetpack Compose는 최근 안드로이드 진영에서 밀고 있는 선언형 UI Toolkit 이다.  
Compose는 프런트엔드 뷰를 명령형(imperatively)으로 변형하지 않고도 앱 UI를 렌더링할 수 있게 하는 선언형(declarative) API를 제공하여 앱 UI를 더 쉽게 작성하고 유지관리할 수 있도록 도와준다.  

## The declarative programming paradigm (선언형 프로그래밍 패러다임)

기본적으로 Android 에서는 사용자와의 상호작용 등을 이유로 인해 앱의 상태가 변경되면, 현재 데이터를 표시하기 위해 UI 계층 구조를 업데이트한다.  
여기서 업데이트를 하기 위한 뷰 계층 구조는 트리 형태로 표시할 수 있고, UI를 업데이트하는 가장 일반적인 방법은 findViewById()와 같은 함수를 사용하여 해당 트리를 탐색하고 button.setText(String), container.addChild(View) 또는 img.setImageBitmap(Bitmap)과 같은 메서드를 호출하여 노드를 변경하는 것이다. 이러한 메서드는 위젯의 내부 상태를 변경한다.  

즉, 뷰의 상태에 대한 업데이트는 아래와 같은 순서로 진행된다.  
1. 뷰 계층 구조 탐색
2. 위젯 내부의 메서드 호출
3. 상태 변경

이러한 방식은 아래와 같은 단점이 존재하는데,  
- 뷰를 수동으로 조작하면 오류가 발생하기 쉽다.
- 데이터를 여러 위치에서 렌더링한다면 데이터를 표시하는 뷰 중 하나를 업데이트하는 것을 잊기 쉽다.
- 여러 업데이트를 통해서 충돌이 발생한 경우, 상태에 대한 트랙킹이 어렵다. (예를 들어 업데이트가 UI에서 방금 삭제된 노드의 값을 설정하려고 할 수 있다)

즉, 일반적으로 업데이트가 필요한 뷰의 수가 많을수록 소프트웨어 유지관리 복잡성이 증가하게 된다.  

지난 몇 년에 걸쳐 업계 전반에서는 선언형 UI 모델로 전환하기 시작했으며, 이에 따라 사용자 인터페이스 빌드 및 업데이트와 관련된 엔지니어링이 크게 간소화되었다.  
이 기법은 처음부터 화면 전체를 개념적으로 재생성한 후 필요한 변경사항만 적용하는 방식으로 작동한다.  
이러한 접근 방식은 스테이트풀(Stateful) 뷰 계층 구조를 수동으로 업데이트할 때의 복잡성을 방지할 수 있다.

화면 전체를 재생성하는 데 있어 한 가지 문제는 시간, 컴퓨팅 성능 및 배터리 사용량 측면에서 잠재적으로 비용이 많이 든다는 것이다.  
이 비용을 줄이기 위해 Compose는 특정 시점에 UI의 어떤 부분을 다시 그려야 하는지를 지능적으로 선택한다.  

## A simple composable function (간단한 composable 함수)

Compose를 사용하면 데이터를 받아서 UI 요소를 내보내는 composable 함수 집합을 정의하여 사용자 인터페이스를 빌드할 수 있다.  
예를들어, 아래처럼 Greeting 위젯으로, String을 받아서 인사말 메시지를 표시하는 Text 위젯을 내보낼 수 있다.  
```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello $name")
}
```

이 예에 관해 몇 가지 주목할 만한 참고 사항:
- 함수는 @Composable annotation 으로 지정해주어야 한다. 모든 Compose 에 사용되는 함수에는 이 annotation 이 있어야 한다. 
- @Composable annotation 을 통해 해당 함수가 데이터를 UI로 변환하기 위한 함수라는 것을 Compose 컴파일러에 알릴 수 있다.
- 함수는 데이터를 받는다. 함수는 매개변수를 받을 수 있으며 이 매개변수를 통해 앱 로직이 UI를 형성할 수 있다. (위 예에서 Greeting 위젯은 String 을 받아서 이름을 표시하고 있다)
- 텍스트 UI 요소를 생성하는 Text() 함수를 호출해서 UI에 텍스트를 표시한다.
- 함수는 아무것도 반환하지 않는다. UI를 내보내는 Compose 함수는 UI 위젯을 구성하는 대신 원하는 화면 상태를 나타내주므로 아무것도 반환할 필요가 없다.
- 이 함수는 빠르고 [멱등원](https://ko.wikipedia.org/wiki/%EB%A9%B1%EB%93%B1%EC%9B%90)이며 부작용이 없다.
  - 함수는 동일한 인수로 여러 번 호출될 때 동일한 방식으로 작동하며, 전역 변수 또는 random() 호출과 같은 다른 값을 사용하지 않는다.
  - 함수는 속성 또는 전역 변수 수정과 같은 부작용(side-effects) 없이 UI를 형성합니다.


## The declarative paradigm shift (선언형 패러다임 전환)

많은 명령형 객체 지향 UI Toolkit를 사용하여 위젯의 트리를 인스턴스화함으로써 UI를 초기화한다.  
흔히 XML 레이아웃 파일을 통해, 각 위젯은 자체의 내부 상태를 유지하고 앱 로직이 위젯과 상호작용할 수 있도록 하는 getter 및 setter 메서드를 노출한다.

Compose의 선언형 접근 방식에서 위젯은 비교적 스테이트리스(Stateless) 상태이며 getter 및 setter 함수를 노출하지 않는다. (사실상 위젯은 객체로 노출되지 않는다)  
동일한 composable 함수를 다른 인수로 호출하여 UI를 업데이트할 수 있고, 이렇게 하면 앱 아키텍처 가이드에 설명된 대로 ViewModel과 같은 아키텍처 패턴에 상태를 쉽게 제공할 수 있다.  
이렇게 되면 composable 에서 식별 가능한 데이터가 업데이트될 때마다 현재 상태를 UI로 변환한다.

### data

상위 수준 객체부터 하위 요소까지 Compose UI의 데이터 흐름을 보여주는 그림  
앱 로직은 최상위의 composable 함수에 데이터를 제공한다. 그러면 함수는 데이터를 사용하여 다른 composable을 호출함으로써 UI를 형성하고 적절한 데이터를 해당 composable 및 계층 구조 아래로 전달한다.

![Image](/assets/mmodel-flow-data.png)

### event

사용자가 UI와 상호작용할 때 UI는 onClick과 같은 이벤트를 발생시킵니다. 이러한 이벤트를 앱 로직에 전달하여 앱의 상태를 변경해야 합니다. 상태가 변경되면 composable 함수는 새 데이터와 함께 다시 호출됩니다. 이렇게 하면 UI 요소가 다시 그려집니다. 이 프로세스를 재구성이라고 합니다.

앱 로직에 의해 처리되는 이벤트를 트리거하여 UI 요소가 상호작용에 어떻게 응답하는지 보여주는 그림  
사용자가 UI 요소와 상호작용하며 이에 따라 이벤트가 트리거된다. 앱 로직이 이벤트에 응답합니다. 그러면 composable 함수가 필요한 경우 새 매개변수를 사용하여 자동으로 다시 호출됩니다.

![Image](/assets/mmodel-flow-events.png)

## Dynamic Content (동적 콘텐츠)

composable 함수는 XML이 아닌 Kotlin으로 작성되기 때문에 다른 Kotlin 코드와 마찬가지로 동적일 수 있다.

```kotlin
@Composable
fun Greeting(names: List<String>) {
    for (name in names) {
        Text("Hello $name")
    }
}
```

위 예시에서 함수는 이름 목록을 받아서 각 사용자에 대한 인사말을 생성한다. composable 은 Kotlin 으로 작성되므로, if 문을 사용하여 특정 UI 요소를 표시할지 여부를 결정할 수 있고, 루프를 사용할 수도 있고, 다른 함수를 호출할 수도 있다.  
즉, 기본 언어의 유연성을 완전히 활용할 수 있다. 이러한 성능과 유연성은 Jetpack Compose의 주요 이점 중 하나이다.

## Recomposition (재구성)

기존의 명령형 UI 모델에서 위젯을 변경하려면 위젯에서 setter를 호출하여 내부 상태를 변경한다.  
Compose에서는 새 데이터를 사용하여 composable 함수를 다시 호출한다.  
이렇게 하면 함수가 재구성되며, 필요한 경우 함수에서 내보낸 위젯이 새 데이터로 다시 그려진다.  
Compose 프레임워크는 변경된 구성요소만 지능적으로 재구성할 수 있다.

예를 들어 다음과 같은 counter 버튼을 표시하는 composable 함수가 있다면,  
``` kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}
```

버튼이 클릭될 때마다 clicks 값을 업데이트한다.  
그리고, Compose는 Text 함수를 사용해 람다를 다시 호출하여 새 값을 표시하는데, 이 프로세스를 recomposition 이라고 하고, 값에 종속되지 않은 다른 함수는 recomposition 되지 않는다.  

앞서 얘기했듯이 전체 UI 트리를 recomposition 하는 작업은 컴퓨팅 성능 및 배터리 수명을 사용한다는 측면에서 컴퓨팅 비용이 많이 들 수 있다. Compose는 이 문제를 지능적으로 recomposition 하여 해결합니다.

recomposition 은 입력이 변경될 때 composable 함수를 다시 호출하는 프로세스이다.  
Compose는 새 입력을 기반으로 recomposition 할 때 변경되었을 수 있는 함수 또는 람다만 호출하고 나머지는 건너뛴다. 매개변수가 변경되지 않은 함수 또는 람다를 모두 건너뜀으로써 Compose의 recomposition 이 효율적으로 이루어질 수 있다.

함수의 recomposition 을 건너뛸 수 있으므로 composable 함수 실행의 부작용(side-effects)에 의존해서는 안 된다. 그렇게 하면 사용자가 앱에서 이상하고 예측할 수 없는 동작을 경험할 수 있다.  
예를 들어 다음 작업은 모두 위험한 부작용이다.

- 공유 객체의 속성에 쓰기 작업
- `ViewModel` 안에 있는 observable 업데이트
- `SharedPreferences` 업데이트

Composable 함수는 애니메이션이 렌더링될 때와 같이 모든 프레임에서와 같은 빈도로 recomposition 될 수 있다.  
애니메이션 도중 버벅거림을 방지하려면 composable 함수가 빨라야 한다.  
특히, 공유 환경설정에서 읽기와 같이 비용이 많이 드는 작업을 실행해야 하는 경우 백그라운드 코루틴에서 작업을 실행하고 값 결과를 composable 함수에 매개변수로 전달한다.

예를 들어 다음 코드는 composable 을 생성하여 `SharedPreferences`의 값을 업데이트 한다.  
즉, composable 자체에서 `SharedPreferences` 를 읽거나 쓰지 않아야 한다.  
대신 이 코드는 ViewModel 안에서 background 로 읽기 및 쓰기를 하게끔 만들 수 있다.  
앱 로직은 콜백과 함께 현재 값을 전달하여 업데이트를 트리거한다.
``` kotlin
@Composable
fun SharedPrefsToggle(
    text: String,
    value: Boolean,
    onValueChanged: (Boolean) -> Unit
) {
    Row {
        Text(text)
        Checkbox(checked = value, onCheckedChange = onValueChanged)
    }
}
```

여기서 부터는 Compose에서 프로그래밍할 때 반드시 알아야 할 몇가지 사항에 대해서 설명한다.  
이 때, 가장 중요한 것은 어떤 경우든 composable 함수를 빠르고 멱등원이며 부작용이 없는 상태로 유지하는 것이 좋다.

1. Composable 함수는 순서와 관계없이 실행할 수 있다.
2. Composable 함수는 동시에 실행할 수 있다.
3. Recomposition 은 최대한 많은 수의 composable 함수 및 람다를 건너뛴다.
4. Recomposition 은 낙관적이며 취소될 수 있다.
5. Composable 함수는 애니메이션의 모든 프레임에서와 같은 빈도로 매우 자주 실행될 수 있다.

### 1.Composable 함수는 순서와 관계없이 실행할 수 있음
Composable 함수의 코드를 살펴보면 코드가 표시된 순서대로 실행된다고 가정할 수 있다.  
그러나 코드가 반드시 표시된 순서대로 실행되는 것은 아니다.  
Composable 함수에 다른 Composable 함수 호출이 포함되어 있다면 그 함수는 순서와 관계없이 실행될 수 있다.  
Compose에는 일부 UI 요소가 다른 UI 요소보다 우선순위가 높다는 것을 인식하고 그 요소를 먼저 그리는 옵션이 있다.

예를 들어 탭 레이아웃에 세 개의 화면을 그리는 다음과 같은 코드가 있다고 가정해 볼 때,  
``` kotlin
@Composable
fun ButtonRow() {
    MyFancyNavigation {
        StartScreen()
        MiddleScreen()
        EndScreen()
    }
}
```

StartScreen, MiddleScreen 및 EndScreen 호출은 순서와 관계없이 발생할 수 있다.  
즉, 예를 들어 StartScreen()이 일부 전역 변수(부작용)를 설정하고 MiddleScreen()이 이러한 변경사항을 활용하도록 할 수 없음을 의미하고, 때문에 이러한 각 함수는 독립적이어야 한다.  

### 2.Composable 함수는 동시에 실행할 수 있음
Compose는 composable 함수를 동시에 실행하여 recomposition 을 최적화할 수 있다.  
이를 통해 Compose는 다중 코어를 활용하고 화면에 없는 composable 함수를 낮은 우선순위로 실행할 수 있다.

이 최적화는 composable 함수가 백그라운드 스레드 풀(a pool of background threads) 내에서 실행될 수 있음을 의미하고,  
composable 함수가 ViewModel에서 함수를 호출하면 Compose는 동시에 여러 스레드에서 해당 함수를 호출할 수 있다.

애플리케이션이 올바르게 작동하도록 하려면 모든 composable 함수에 부작용이 없어야 하는데, 대신 UI 스레드에서 항상 실행되는 onClick과 같은 콜백에서 부작용을 트리거한다.

Composable 함수가 호출될 때 호출자(caller)와 다른 스레드에서 호출이 발생할 수 있다. 즉, composable 람다의 변수를 수정하는 코드는 피해야 한다.  
이러한 코드는 스레드로부터 안전하지 않을 뿐만 아니라 composable 람다의 허용되지 않는 부작용이기 때문이다.

다음은 목록과 개수를 표시하는 composable 을 보여주는 예시 이다.
```kotlin
@Composable
fun ListComposable(myList: List<String>) {
    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
            }
        }
        Text("Count: ${myList.size}")
    }
}
```
이 코드는 부작용이 없으며 입력 목록을 UI로 변환한다. 이 코드는 작은 목록을 표시할 때 유용한 코드이다.  
이 때, 함수가 로컬 변수에 쓰는 경우 이 코드는 스레드로부터 안전하지 않거나 적절하지 않다.

``` kotlin
@Composable
@Deprecated("Example with bug")
fun ListWithBug(myList: List<String>) {
    var items = 0

    Row(horizontalArrangement = Arrangement.SpaceBetween) {
        Column {
            for (item in myList) {
                Text("Item: $item")
                items++ // Avoid! Side-effect of the column recomposing.
            }
        }
        Text("Count: $items")
    }
}
```
위 예에서 items는 모든 recomposition 을 통해 수정된다.  
수정은 애니메이션의 모든 프레임에서 또는 목록이 업데이트될 때 실행될 수 있다.  
어느 쪽이든 UI에 잘못된 개수가 표시된다.  
이 때문에 이와 같은 쓰기는 Compose에서 지원되지 않고, 이러한 쓰기를 금지함으로써 프레임워크가 composable 람다를 실행하도록 스레드를 변경할 수 있다.

### 3.Recomposition 은 가능한 한 많이 건너뜀

UI의 일부가 잘못된 경우 Compose는 업데이트해야 하는 부분만 재구성하기 위해 최선을 다한다.  
즉, UI 트리에서 위 또는 아래에 있는 컴포저블을 실행하지 않고 단일 버튼의 컴포저블을 다시 실행하는 것을 건너뛸 수 있다.

모든 composable 함수 및 람다는 자체적으로 recomposition 할 수 있다. 다음은 목록을 렌더링할 때 recomposition 이 일부 요소를 건너뛸 수 있는 방법을 보여주는 예시 이다.

``` kotlin
/**
 * Display a list of names the user can click with a header
 */
@Composable
fun NamePicker(
    header: String,
    names: List<String>,
    onNameClicked: (String) -> Unit
) {
    Column {
        // this will recompose when [header] changes, but not when [names] changes
        Text(header, style = MaterialTheme.typography.h5)
        Divider()

        // LazyColumn is the Compose version of a RecyclerView.
        // The lambda passed to items() is similar to a RecyclerView.ViewHolder.
        LazyColumn {
            items(names) { name ->
                // When an item's [name] updates, the adapter for that item
                // will recompose. This will not recompose when [header] changes
                NamePickerItem(name, onNameClicked)
            }
        }
    }
}

/**
 * Display a single name the user can click.
 */
@Composable
private fun NamePickerItem(name: String, onClicked: (String) -> Unit) {
    Text(name, Modifier.clickable(onClick = { onClicked(name) }))
}
```

누누이 말하지만, 모든 composable 함수 또는 람다를 실행하는 작업에는 부작용이 없어야 합니다. 부작용을 실행해야 할 때는 콜백에서 부작용을 트리거해야 한다.

### 4.Recomposition은 낙관적임

Compose가 composable 의 매개변수가 변경되었을 수 있다고 생각할 때마다 recomposition 이 시작되고, recomposition 은 낙관적이다.  
즉, Compose는 매개변수가 다시 변경되기 전에 재구성을 완료할 것으로 예상한다.  
Recomposition 이 완료되기 전에 매개변수가 변경되면 Compose는 recomposition 을 취소하고 새 매개변수를 사용하여 recomposition 을 다시 시작할 수 있다.

Recomposition 이 취소되면 Compose는 recomposition 에서 UI 트리를 삭제한다.  
표시되는 UI에 종속되는 부작용이 있다면 구성이 취소된 경우에도 부작용이 적용되고, 이로 인해 일관되지 않은 앱 상태가 발생할 수 있다.

낙관적 recomposition 을 처리할 수 있도록 모든 composable 함수 및 람다가 멱등원이고 부작용이 없는지 확인해야 한다.

### 5.Composable 함수는 매우 자주 실행될 수 있음

경우에 따라 composable 함수는 UI 애니메이션의 모든 프레임에서 실행될 수 있다.  
함수가 `LocalDB` 에서 읽기와 같이 비용이 많이 드는 작업을 실행하면 이 함수로 인해 UI 버벅거림이 발생할 수 있다.

예를 들어 위젯이 기기 설정을 읽으려고 하면 잠재적으로 이 설정을 초당 수백 번 읽을 수 있으며 이는 앱 성능에 치명적인 영향을 줄 수 있다.

Composable 함수에 데이터가 필요하다면 데이터의 매개변수를 정의해야 합니다.  
그런 다음, 비용이 많이 드는 작업을 구성 외부의 다른 스레드로 이동하고 mutableStateOf 또는 LiveData를 사용하여 Compose에 데이터를 전달하도록 만들 수 있다.

