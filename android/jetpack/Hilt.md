# Hilt

이번 문서는 [Android-Hilt 공식문서](https://developer.android.com/training/dependency-injection/hilt-android) 기반으로 작성하였다.

## 개요

Hilt 란 DI(Dependency Injection, 의존성 주입) 를 수동으로 집어넣는 것이 아니라 자동으로 넣어지게끔 만들어 상용구를 줄일 수 있도록 도와주는 Android 전용 DI 라이브러리이다.  
스스로 의존성 주입을 하기 위해서는 모든 클래스와 모든 요소들을 수동으로 관리해줘야 하는데,  
Hilt 를 사용하면 모든 Android 요소들에 container를 제공하고 수명 주기를 자동으로 관리해주는 등 애플리케이션에서 DI를 사용하는 표준 방법을 제공해준다.  
(참고 : Hilt 는 [Dagger](https://developer.android.com/training/dependency-injection/dagger-basics?hl=ko) 가 제공하는 컴파일 시간 정확성, 런타임 성능, 확장성 및 Android 스튜디오 지원의 이점을 누리기 위해 Dagger 를 기반으로 빌드되었다.)

## 설치

먼저 hilt-android-gradle-plugin 플러그인을 프로젝트의 루트 build.gradle 파일에 추가한다.  
**build.gradle(:root)**  
```gradle
buildscript {
    // ...
    dependencies {
        // ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}

```

Hilt Gradle 플러그인을 적용하고 app/build.gradle 파일에 `kotlin-kapt`, `dagger.hilt.android.plugin` 등 dependencies 을 추가한다.  
Hilt 에서는 Java8 기능을 사용하기 때문에, JavaVersion.VERSION_1_8 도 반드시 추가해주도록 한다.
**build.gradle(:app)**  
```gradle
// ...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
  // ...
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
}
```

## @HiltAndroidApp

Hilt 를 사용하려면 반드시 Application class 에 @HiltAndroidApp annotation 을 추가해 주어야 한다.  
@HiltAndroidApp 을 추가해 줌으로, application 단계의 dependency container 역할을 하는 application 의 기본 클래스들을 포함하여, Hilt 의 코드 자동 생성을 시켜줄 수 있다.  

```kotlin
@HiltAndroidApp
class App : Application() { /*...*/ }
```

## @AndroidEntryPoint

Application 에 @HiltAndroidApp 를 설정한 후, 다른 Android 요소들에서도 Hilt를 사용하기 위해서는 Activity, Fragment, View 등 과 같은 Adnroid 요소들에 @AndroidEntryPoint 를 추가해야 한다.  
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() { /*...*/ }
```

@AndroidEntryPoint 로 Hilt 를 사용하게 만들 수 있는 Android 요소들 : 
- Activity
- Fragment
- View
- Service
- BroadcastReceiver

Android 하위 요소들에서 @AndroidEntryPoint 로 annotation 을 넣는 경우, 모든 상위 요소들에도 @AndroidEntryPoint annotation 을 넣어주어야 한다.  
그리고 하위 요소들에서는 상위 요소들에서 설정한 dependency 를 받을 수 있다.
(e.g. View 에서 Hilt 를 사용하고 있다면, 해당 뷰를 사용하고 있는 Fragment 또는 Activity 등에서도 @AndroidEntryPoint 를 넣어주어야 함.)

**주의 사항**
- AppCompatActivity 와 같은 ComponentActivity 를 상속받는 Activity 만 지원한다.  
- androidx.Fragment 를 상속받는 Fragment 만 가능하다.  
- Retained Fragment 는 지원하지 않는다.

다음과 같이 @Inject annotation을 사용하여 field injection을 할 수 있다.(단, Field injection 의 대상이 되는 field 는 private 이면 안 된다.)  
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

  @Inject lateinit var presenter: MainPresenter
  // ...
}
```

## @Inject constructor

Field injection 을 하는 경우에 Hilt 에서는 해당 Field 에 어떤 dependency 에 대한 인스턴스를 제공해야하는지 알 수 있어야 한다.  
해당 field 의 class 의 constructor 쪽에 @Inject annotation 을 사용하여 인스턴스를 제공하는 방법을 Hilt 에 알려줄 수 있다.

```kotlin
class MainPresenter @Inject constructor(
  private val service: MainService
) { /*...*/ }
```

@Inject annotation 이 지정된 class constructor 쪽의 parameter는 그 클래스의 dependency 이다. 이 예에서 MainPresenter 에는 MainService 가 dependency 로 있습니다. 따라서 Hilt는 MainService 의 인스턴스를 제공하는 방법도 알아야 한다.

## Hilt Module

때때로 constructor-injected 를 할 수 없는 경우가 있다. 예를 들어, interface 의 경우에는 constructor 를 통한 injection 을 할 수 없다. 또한, 외부 라이브러리와 같은 경우도 constructor-injected 를 하기 힘들다.  
이럴 때 사용하는게 @Module annotation 이고, @Module 을 통해 Hilt graph 에서 해당 인스턴스를 알도록 도와 줄 수 있다.  
이 때, @InstallIn annotation 을 통해 각 Module 을 사용할 Android 요소를 Hilt 에 알려줄 수 있다.

### @Binds 를 사용한 인터페이스 인스턴스 삽입

예를 들어, MainService 가 인터페이스라면 이 인터페이스를 constructor-injected 할 수 없다. 대신 Hilt Module 내에 @Binds annotation 붙은 abstract function 을 생성하여 Hilt 에 알려준다.  
@Binds annotation 은 인터페이스의 인스턴스를 제공해야 할 때 어떻게 구현되어야 하는지를 Hilt 에 알려준다.  

@Binds annotation 이 지정된 함수는 Hilt 에 다음 정보를 제공합니다.  
- 함수의 return type 은 함수가 어떤 인터페이스의 인스턴스를 제공하는지 Hilt 에 알려줍니다.
- 함수의 parameter 는 어떤 implementation 을 사용할지 결정합니다.

```kotlin
interface MainService {
  fun mainMethods()
}

// Constructor-injected, because Hilt needs to know how to
// provide instances of MainServiceImpl, too.
class MainServiceImpl @Inject constructor(
  // ...
) : MainService { /*...*/ }

@Module
@InstallIn(ActivityComponent::class)
abstract class MainModule {

  @Binds
  abstract fun bindMainService(
    mainServiceImpl: MainServiceImpl
  ): MainService
}
```

Hilt 가 MainModule 의 depency 를 MainActivity 에 삽입하려고 하기 때문에 MainModule 에 @InstallIn(ActivityComponent::class) annotation 을 넣어서,
MainModule 의 모든 dependency 를 앱의 모든 Activity 에서 사용할 수 있도록 해줍니다.

### @Provides 를 사용한 인스턴스 삽입

인터페이스만이 constructor-injected 를 할 수 없는 유일한 경우는 아니다. 예를 들어, Retrofit, OkHttpClient 또는 Room 데이터베이스 또는 빌더 패턴으로 인스턴스를 생성하는 경우에 constructor-injected 는 불가능하다.  
이 경우에는 @Provides annotation 을 넣어주어서 인스턴스를 제공하는 방법을 Hilt 에 알려줄 수 있다.

@Provides annotation 이 지정된 함수는 Hilt 에 다음 정보를 제공합니다.  
- 함수의 return type 은 함수가 어떤 인터페이스의 인스턴스를 제공하는지 Hilt 에 알려줍니다.
- 함수의 parameter 는 해당 유형의 dependency 를 Hilt 에 알려줍니다.
- 함수의 본문은 인스턴스를 제공하는 방법을 Hilt 에 알려줍니다. Hilt 는 해당 유형의 인스턴스를 제공해야 할 때마다 함수 본문을 실행합니다.

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object MainModule {

  @Provides
  fun provideMainService(
    // Potential dependencies of this type
  ): MainService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(MainService::class.java)
  }
}
```

### 동일한 유형에 대해 여러 결합 제공
동일한 유형의 다양한 구현을 제공하야 하는 경우에는 Hilt 에 여러 결합에 대한 내용을 제공해야 한다.  
이때는 @Qualifier annotation 을 사용하여 동일한 유형에 대해 다양한 구현을 정의할 수 있다.  
@Qualifier annotation 은 특정 유형에 대해 여러 구현이 정의되어 있을 때 그 유형의 특정 구현을 식별하는 데 사용한다.  
예를 들어, MainService 호출을 가로채야 할 때, 인터셉터와 함께 OkHttpClient 객체를 사용할 수 있다. OtherService 에서는 호출을 다른 방식으로 가로채야 할 수도 있다. 이 경우에는 서로 다른 두 가지 OkHttpClient 구현을 제공하는 방법을 Hilt에 알려야 한다.

1. 먼저 다음과 같이 @Qualifier를 정의한다.
``` kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient
```

2. @Binds 또는 @Provides 메서드에 정의한 @Qualifier 를 적용 한다.
```
@Module
@InstallIn(ApplicationComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
    authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }

  @OtherInterceptorOkHttpClient
  @Provides
  fun provideOtherInterceptorOkHttpClient(
    otherInterceptor: OtherInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(otherInterceptor)
               .build()
  }
}
```

3. 다음과 같이 field 또는 parameter 에 해당 @Qualifer 로 지정하여 필요한 특정 유형을 삽입할 수 있다.
```
// As a dependency of another class.
@Module
@InstallIn(ActivityComponent::class)
object MainModule {

  @Provides
  fun provideMainService(
    @AuthInterceptorOkHttpClient okHttpClient: OkHttpClient
  ): MainService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .client(okHttpClient)
               .build()
               .create(MainService::class.java)
  }
}

// As a dependency of a constructor-injected class.
class ExampleServiceImpl @Inject constructor(
  @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
) : ExampleService

// At field injection.
@AndroidEntryPoint
class MainActivity: AppCompatActivity() {

  @AuthInterceptorOkHttpClient
  @Inject lateinit var okHttpClient: OkHttpClient
}
```

@Qualifier 를 추가해 주는 경우 해당 dependency 를 제공하는 모든 방법에 qualifier 를 추가하는 것이 좋다.
그렇지 않다면, 오류가 발생하기 쉽고, Hilt 에서 잘못된 종속 항목을 삽입할 수 있다.

### @ApplicationContext 및 @ActivityContext

Hilt 에서 dependency 를 넣어주는 경우에 Context 가 필요할 수 있으므로 Hilt 에서는 기본적으로 @ApplicationContext 및 @ActivityContext qualifiers 를 제공한다.  
Application 단계에서 사용할 때는 ApplicationContext 를 Activity 단계에서 사용할 때는 ActivityContext 를 사용하면 된다.  
```
class MainAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: MainService
) { /*...*/ }
```

## Android class 들을 위해 생성된 구성요소

위에서는 Hilt Module 에서 ActivityComponent 를 사용하는 방법에 대해서 알아보았다.  
Hilt는 그 외에도 다음 구성요소를 제공한다.

| Hilt component | Injector for |
| --- | --- |
| SingletonComponent | Application |
| ActivityRetainedComponent | N/A |
| ViewModelComponent | ViewModel |
| ActivityComponent | Activity |
| FragmentComponent | Fragment |
| ViewComponent | View |
| ViewWithFragmentComponent | View annotated with @WithFragmentBindings |
| ServiceComponent | Service |

Broadcast Receiver 의 경우, Hilt 는 ApplicationComponent 에서 삽입하므로 별도의 Broadcast Receiver 에 대한 요소를 생성하지 않는다.

## 구성요소 와 Android 클래스의 수명 주기 관계

Hilt는 해당 Android class 의 수명 주기에 따라 Hilt 에 알려준 구성요소들의 인스턴스를 자동으로 만들고 제거한다.  

| Generated component | Created at | Destroyed at |
| --- | --- | --- |
| ApplicationComponent | Application#onCreate() | Application#onDestroy() |
| ActivityRetainedComponent | Activity#onCreate() | Activity#onDestroy() |
| ActivityComponent | Activity#onCreate() | Activity#onDestroy() |
| FragmentComponent | Fragment#onAttach() | Fragment#onDestroy() |
| ViewComponent | View#super() | 제거된 뷰 |
| ViewWithFragmentComponent | View#super() | 제거된 뷰 |
| ServiceComponent | Service#onCreate() | Service#onDestroy() |

## 구성요소 범위

기본적으로 Hilt의 모든 결합은 scope 가 지정되지 않는다. 즉, 앱이 bind, provide 를 요청할 때마다 Hilt는 필요한 유형의 새 인스턴스를 생성한다.  
그러나 Hilt 는 결합을 특정 구성요소로 scope 를 지정할 수도 있다. 
Hilt 는 scope 가 지정된 구성요소의 인스턴스마다 한 번만 범위가 지정된 bind, provide 를 생성하며, 이에 관한 모든 요청은 동일한 인스턴스를 공유한다.

| Android class | Generated component | Scope |
| --- | --- | --- |
| Application | SingletonComponent | @Singleton |
| Activity | ActivityRetainedComponent | @ActivityRetainedScoped |
| ViewModel | ViewModelComponent | @ViewModelScoped |
| Activity | ActivityComponent | @ActivityScoped |
| Fragment | FragmentComponent | @FragmentScoped |
| View | ViewComponent | @ViewScoped |
| View annotated with @WithFragmentBindings| ViewWithFragmentComponent | @ViewScoped |
| Service | ServiceComponent | @ServiceScoped |

아래와 같이 MainAdapter 에 @ActivityScoped annotation 을 넣어주면 Hilt 는 해당 Acitivty 의 수명 주기 동안 동일한 MainAdapter 인스턴스를 제공한다.  
``` kotlin
@ActivityScoped
class MainAdapter @Inject constructor(
  private val service: MainService
) { ... }
```

## 구성요소 계층 구조

![Image](/assets/hilt_hierarchy.svg)

기본적으로 뷰에서 필드 삽입을 실행하면 ViewComponent는 ActivityComponent에 정의된 결합을 사용할 수 있다. FragmentComponent에 정의된 결합도 사용해야 하며 뷰가 프래그먼트의 일부라면 @AndroidEntryPoint와 함께 @WithFragmentBindings annotation 을 사용하면 된다.

## Hilt 에서 지원하지 않는 클래스에 대한 Dependency Injection

Hilt에는 가장 일반적인 Android 클래스에 대한 구성요소를 지원해 줍니다. 그러나 Hilt 가 지원하지 않는 클래스에 field injection 을 해야 할 수도 있습니다.

이러한 경우 @EntryPoint annotation 을 사용하여 직접 진입점을 만들 수 있다. 진입점은 Hilt 가 관리하는 코드와 그렇지 않은 코드 사이의 경계아다.  
즉, Hilt가 관리하는 객체의 그래프에 코드가 처음 들어가는 지점이다. 진입점을 통해 Hilt 는 Hilt 가 관리하지 않는 코드를 사용하여 graph 내에서 dependency 를 제공할 수 있다.  
예를 들어 Hilt는 [content providers](https://developer.android.com/guide/topics/providers/content-providers) 를 직접 지원하지 않지만,  
Hilt 를 사용하여 일부 dependency 를 가져오도록 하려면 원하는 결합 유형마다 @EntryPoint로 주석이 지정된 인터페이스를 정의하고 Qualifier 를 포함해야 한다.  
그리고 다음과 같이 @InstallIn을 추가하여 진입점을 설치할 구성요소를 지정한다.

``` kotlin
class ExampleContentProvider : ContentProvider() {
  @EntryPoint
  @InstallIn(ApplicationComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun mainService(): MainService
  }
  // ...
}
```

진입점에 접근하기 위해서는 EntryPointAccessors 의 알맞는 static 메서드를 사용하면 된다.  
파라미터는 구성 요소 인스턴스이거나 구성 요소 소유자 역할을 하는 @AndroidEntryPoint 객체여야 한다.  
파라미터로 전달하는 구성요소와 EntryPointAccessors static 메서드가 모두 @EntryPoint 인터페이스의 @InstallIn annotation 에 있는 Android 클래스와 일치하는지 확인해야 한다.

``` kotlin
class ExampleContentProvider: ContentProvider() {
  // ...
  override fun query(/*...*/): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint = **EntryPointAccessors.fromApplication(appContext, ExampleContentProviderEntryPoint::class.java)**
    val analyticsService = hiltEntryPoint.analyticsService()
    // ...
  }
}
```

위 예에서는 진입점이 ApplicationComponent 에 설치되어 있으므로 ApplicationContext를 사용하여 진입점을 검색해야 한다.  
만약 진입점이 ActivityComponent 에 있다면 ActivityContext 를 대신 사용하면 된다.

