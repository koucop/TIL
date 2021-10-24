# FrameLayout

안드로이드에서 기본적으로 제공하는 레이아웃중에 기본중의 기본인 레이아웃
하나의 박스라고 생각하면 이해하기 쉽다.

Frame 은 틀, 액자라는 뜻이고,  
FrameLayout 은 '틀' 이라는 뜻 그대로 화면에 틀을 둔다고 생각할 수 있다.  

안드로이드 공식문서에서는 FrameLayout 관련해서 아래와 같이 설명하고 있는데,
[Android Developer 프레임레이아웃 설명](https://developer.android.com/reference/android/widget/FrameLayout)
FrameLayout should be used to hold a single child view.

여기서 알 수 있듯이, FrameLayout 이 필요한 상황은 심플하게 자식으로 한가지의 뷰 만을 가지고 싶을 때로 정리할 수 있다.

### android:layout_gravity

다음과 같이 layout_gravity attribute 를 child 뷰 안에 설정하여 포지션을 제한할 수 있다.

``` xml
<FrameLayout>
    <View
        ...
        android:layout_gravity="start|top"
        ... />

    <View
        ...
        android:layout_gravity="end|bottom"
        ... />
</FrameLayout>
```

### views stack

추가된 뷰들은 FrameLayout 내부에서 stack 처럼 쌓이게 된다.

``` xml
<FrameLayout>
    <!-- 최하단 -->
    <View
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_gravity="center"
        android:background="@color/red600" />

    <View
        android:id="@+id/viewFrame2"
        android:layout_width="250dp"
        android:layout_height="250dp"
        android:layout_gravity="center"
        android:background="@color/blue600" />

    <View
        android:id="@+id/viewFrame3"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center"
        android:background="@color/purple600" />

    <!-- 최상단 -->
    <View
        android:id="@+id/viewFrame4"
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_gravity="center"
        android:background="@color/green600" />
</FrameLayout>
```
