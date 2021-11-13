# Delegating to another property

속성은 getter 및 setter를 다른 속성에 위임할 수 있다.  
이러한 delegates 는 최상위 및 클래스 속성(멤버 및 확장) 모두에 사용할 수 있다.

- A top-level property
- A member or an extension property of the same class
- A member or an extension property of another class

속성을 다른 속성에 위임하려면 대리자 이름에 :: 한정자를 사용합니다(예: this::delegate 또는 MyClass::delegate).

```kotlin
var topLevelInt: Int = 0
class ClassWithDelegate(val anotherClassInt: Int)

class MyClass(var memberInt: Int, val anotherClassInstance: ClassWithDelegate) {
    var delegatedToMember: Int by this::memberInt
    var delegatedToTopLevel: Int by ::topLevelInt

    val delegatedToAnotherClass: Int by anotherClassInstance::anotherClassInt
}
var MyClass.extDelegated: Int by ::topLevelInt
```

이것은 예를 들어 이전 버전과 호환되는 방식으로 속성의 이름을 변경하려는 경우에 유용할 수 있습니다. 새 속성을 도입하고 이전 속성에 @Deprecated 주석을 추가하고 구현을 위임합니다.

```kotlin
class MyClass {
   var newName: Int = 0
   @Deprecated("Use 'newName' instead", ReplaceWith("newName"))
   var oldName: Int by this::newName
}
fun main() {
   val myClass = MyClass()
   // Notification : 'oldName: Int' is deprecated.
   // Use 'newName' instead
   myClass.oldName = 42
   println(myClass.newName) // 42
}
```
