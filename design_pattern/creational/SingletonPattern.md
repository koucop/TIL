# 1️⃣ Singleton pattern

## 개요

싱글톤은 객체지향 design pattern 에서 가장 유명한 패턴이라고 할 수 있을 정도로, design pattern 을 따로 공부하지 않은 사람이라도 한번쯤은 봤을 법한 패턴이다.  
많이 사용되어지는 만큼 예제들도 쉽게 접할 수 있고 간단하게 적용할 수 있는 패턴이다.  
그래서 그런지 왜 사용해야하는지 어떻게 사용하는게 맞는 것인지 고려하지 않고 예시들과 동일하게 적용하는 경우가 많다.  
이번 문서에서는 singleton pattern 에는 어떤 종류가 있고 각각 어떤 상황에 사용하는지 알아보려고 한다.  

## Singleton pattern 이란?

싱글톤 패턴은 어떤 한 클래스의 인스턴스가 메모리 상에 단 하나임을 보장하며, 단 하나만 있는 인스턴스에 대해 어디에서나 접근할 수 있도록 전역적인 접촉점을 제공하는 패턴이다.  
싱글톤 패턴을 적용하는 경우는 일반적으로 여러 객체를 관리하는 역할의 객체같이 프로그램내에서 단 하나의 인스턴스를 갖는 것이 유리한 경우 혹은 반드시 그래야만하는 경우에 사용된다.  

싱글톤 패턴에서는 기본적으로 생성자를 private하게 만들어 클래스 외부에서는 instance를 생성하지 못하게 차단하고,  
내부에서 단 하나의 instance를 생성하여 외부에는 해당 instance에 대한 접근 방법만을 제공한다.

## Singleton pattern 종류

1. Eager Initialization

Eager Initialization 은 Singleton pattern 중에서 가장 간단한 형태의 방식이다. 

application 을 생성할 때 해당 instance 를 사용하지 않은 경우라도 무조건 instance 를 생성하기 때문에 메모리 낭비를 야기할 수 있다.

```java
public class EagerInitialization {
    // 외부에서 해당 EagerInitialization 을 constructor 로 생성하지 못하도록 private 으로 만들어줌
    private EagerInitialization() {}

    private static final EagerInitialization INSTANCE = new EagerInitialization();
 
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

EagerInitialization 를 사용하는 경우는 보통 class 가 적은 리소스를 다룰 때여야 하고, 이 방식은 Exception 에 대한 Handling 도 제공하지 않기 때문에 보통 권장되진 않는다.

2. Static Block Initialization

Static Block Initialization 은 이름 그대로 static block 을 사용해서 Exception 에 대한 Handling 을 제공한다.

```java
public class StaticBlockInitialization {
  	private StaticBlockInitialization() {}

  	private static StaticBlockInitialization INSTANCE;

  	// static block 을 사용해서 Exception 에 대한 Handling
  	static {
        try {
            INSTANCE = new StaticBlockInitialization();
        } catch(Exception e) {
            throw new RuntimeException("Failed create StaticBlockInitialization instance");
        }
    }

    public static StaticBlockInitialization getInstance() {
        return INSTANCE;
    }
}
```



위의 예시에서 확인할 수 있듯이 나머지는 Eager Initialization 과 상당히 유사하지만 static block 을 통해 Exception 에 대한 Handling 을 지원하고 있다. 하지만 Eager Initialization과 동일하게 instance 를 생성하는 것을 application 생성시에 하기 때문에(정확하게는 class 로딩할 때) class 가 적은 리소스를 다룰 때 사용하여야 한다.

3. Lazy Initialization

Lazy Initialization 은 Eager Initialization 과 Static Block Initialization 에서 문제과 되었던 instance 의 생성을 class 로딩시가 아니라 getInstance() 함수가 불릴 때로 이후로 미루는 방식이다.

```java
public class LazyInitialization {
    private LazyInitialization() {}

    private static LazyInitialization INSTANCE;

    public static LazyInitialization getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new LazyInitialization();
        }
        return INSTANCE;
    }
}
```


위의 예시와 같이 getInstance()  가 호출됐을 때, instance 를 생성하게 되면 사용하지도 않은 instance 가 남아있게 되는 문제를 해결할 수 있지만, instance 가 생성되지 않은 시점에 여러 thread 에서 동시에 getInstance() 를 호출하게되어 여러 instance 를 생성시키는 문제를 일으킬 수 있다.

위와 같이 multi-thread 환경에서 문제가 발생할 수 있다는 것은 단 하나의 instance 를 제공하기 위한 singleton pattern 자체를 위반하는 문제이기 때문에 single-thread 환경이 보장되지 않는다면 절대 사용해선 안된다.

4. Thread Safe Singleton

Thread Safe Singleton 은 Lazy Initialization 의 multi-thread 환경에서의 문제를 해결하기 위한 방법으로, Critical Section(임계 영역)을 형성해 해당 영역에 오직 하나의 쓰레드만 접근 가능하게 해주는 synchronized 키워드를 getInstance() 메소드에 걸어두는 방식이다.

```java
public class ThreadSafeSingleton {
    private ThreadSafeSingleton() {}

    private static ThreadSafeSingleton INSTANCE;

    public static synchronized ThreadSafeSingleton getInstance() {
        if(INSTANCE == null){
            INSTANCE = new ThreadSafeSingleton();
        }
        return INSTANCE;
    }
}
```


synchronized 로 인해서 getInstance() 메소드 내에 진입하는 thread 가 하나라는 것을 보장받기 때문에 multi-thread 환경에서도 정상 동작할 수 있다. 그러나 synchronized 키워드는 비용이 많이 들기 때문에 getInstance() 에 대한 호출이 많아지면 그만큼 성능이 떨어지게 된다.

이 때 instance가 null일 경우에만 synchronized가 동작하도록 하는 **Double Checked Locking** 을 사용해서 성능에 대한 문제를 해결할 수 있다.

```java
public static ThreadSafeSingleton getInstance(){
    if(INSTANCE == null){
        synchronized (ThreadSafeSingleton.class) {
            if(INSTANCE == null){
                INSTANCE = new ThreadSafeSingleton();
            }
        }
    }
    return INSTANCE;
}
```



5. Bill Pugh Singleton Implementaion

위에서 얘기했던 방식들이 가지고있는 문제점들을 대부분 해결한 방식으로, inner static helper class 를 사용해서 singleton pattern 을 구현한 방식이다.

```java
public class BillPughSingletonImplementaion {
  private BillPughSingletonImplementaion() {}

  private static class SingletonHelper {
      private static final BillPughSingletonImplementaion INSTANCE = new BillPughSingletonImplementaion();
  }

  public static BillPughSingletonImplementaion getInstance() {
      return SingletonHelper.INSTANCE;
  }
}
```

위의 예시에 대해 간단히 설명해보면, SingletonHelper 라는 BillPughSingletonImplementaion에 대한 instance 를 가지고 있는 private inner static class 를 두어서 instance 를 생성하려는 class 가 로드될 때에도 만들어지지 않고 getInstance() 가 호출 됐을 때 메모리에 로드되면서 instance 를 생성시킨다. 

그리고 synchronized 키워드를 사용하지 않았기 때문에 성능문제에 대한 부분도 해결가능하다.

