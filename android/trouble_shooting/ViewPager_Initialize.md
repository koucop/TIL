# ViewPager_Initialize

ViewPager 가 생성될 때, ViewPager 다 만들어지기도 전 타이밍에 `setCurrentItem(position)` 을 해주게 되면,  
해당 페이지로 이동은 하나 뷰가 그려지지 않는 다던지 size 업데이트가 안되는 이슈가 있다.  

이것을 해결하기 위해서는 변경시키려는 상위 레이아웃에 view 의 post API 를 적용시켜서 뷰가 다 그려지고 난 후 적용시키도록 만들면 된다.

```kotlin
view.post {
    // Runnable
    // doSomething
}
```
