# Factory Pattern

## 개요

Factory Pattern 은 instance 를 생성하는 interface 를 미리 정의하고, instance 를 만들 class 의 결정은 sub class 쪽에서 내리는 pattern 이다.  
즉 하나의 super class 밑에 여러개의 sub class 가 존재하고 요청에 따라 하나의 instance 를 반환해주는 방식이다.

이 pattern 은 인스턴스화에 대한 책임을 instance 를 사용하는 client 에서 factory class 로 가져온다.

## 특징

1. 코드로부터 sub class 의 인스턴스화를 제거하여 결합도를 느슨하게 만들어, 확장을 쉽게 한다.
2. client 와 구현 객체들 사이에 추상화를 제공한다.

Factory pattern 에 사용되는 super class 는 interface 나 abstract class, 혹은 일반 class 여도 상관없다.  
밑에 예시에서는 abstract class 를 사용해서 Shape 라는 super class 를 만들고, Square, Circle 이라는 sub classes 를 만들어서 factory pattern 을 구현했다.

 ```java
public abstract class Shape {
	
    public abstract String getWidth();
    public abstract String getHeight();
	
    @Override
    public String toString() {
        return "Width = " + this.getWidth() + ", Height = " + this.getHeight();
    }
}
 ```

 ```java
public class Square extends Shape {
 
    private String width;
    private String height;
	
    public Square(String width, String height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public String getWidth() {
        return this.width;
    }
 
    @Override
    public String getHeight() {
    return this.height;
    }

}
 ```

```java
public class Circle extends Shape {
 
    private String width;
    private String height;
	
    public Circle(String width, String height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public String getWidth() {
        return this.width;
    }
 
    @Override
    public String getHeight() {
    return this.height;
    }

}
```

```java
public class ShapeFactory {
 
    public static Shape getShape(String type, String width, String height) {
        if("Square".equalsIgnoreCase(type)) {
            return new Square(width, height);
        } else if("Circle".equalsIgnoreCase(type)) {
            return new Circle(width, height);
        } else {
            return null;
        }
    }

}
```

```java
public class FactoryTest {
 
    public static void main(String[] args) {
        Shape square = ShapeFactory.getShape("Square", "100", "200");
        Shape circle = ShapeFactory.getShape("Circle", "100", "100");
        System.out.println("Square : " + square);
        System.out.println("Circle : " + circle);
    }
 
}

// Square : Widht = 100, Height = 200
// Circle : Widht = 100, Height = 100
```

ShapeFactory 의 getShape method 는 static method 로 구현되어 있고, type의 값에 따라 다른 instance 를 반환하고 있다.  
"Square"일 경우 Square class의 instance를, "Circle"일 경우 Circle class의 instance를 반환한다.

Factory pattern 을 사용하면 instance 를 필요로 하는 Application 에서 Shape 의 sub class 에 대한 정보 없이 instance 를 생성할 수 있다.  
그리고, 더 많은 Shape 를 super class 를 상속받는 sub class 가 생성된다고 하더라도 ShapeFactory 의 getShape 라는 method 로 유연하게 대처할 수 있다.

---

Factory pattern 은 일반적으로도 많이 사용되는 패턴이다.

- java.util 의 Calendar, ResourceBundle, NumberFormat 등에서 정의된 **getInstance()**
- Boolean, Integer, Long 등의 Wrapper class 안에서 정의된 **valueOf()**
