# BuilderPattern

## 개요

Builder pattern 은 많은 Optional 한 variable(혹은 parameter)나 지속성 없는 상태 값들에 대해 처리해야 할 때 사용하는 방식이다.  
이 패턴을 사용하면 아래의 문제점들을 해결할 수 있다.

1. Factory class 로 많은 parameter 를 넘기거나 할 때 null 등의 값을 일일이 넘겨줘야 해서 관리가 어려워져 에러가 발생할 확률이 높아진다.
2. 생성해야 하는 sub class 가 무거워지고 복잡해짐에 따라 factory class 또한 복잡해진다.

Builder pattern 은 이러한 문제들을 해결하기 위해 별도의 Builder 클래스를 만들어 필수 값에 대해서는 constructor 로,  
optional 한 값들에 대해서는 method 로 일일이 값을 입력받은 후 build() 를 통해 최종적으로 하나의 인스턴스를 리턴하는 방식이다.

## 구현방법

1. `*Builder` 라는 이름으로 생성하고자 하는 클래스 이름 뒤에 Builder를 붙여서 Static Nested Class 를 생성한다
2. Constructor 는 public 으로 해서, 필수 값들에 대해 constructor 의 parameter 로 값을 받는다
3. Optional 값들에 대해서는 각각마다 builder 자기 자신을 반환하는 method 로 제공한다
4. 마지막 단계로, build() method 를 정의하여 최종 생성된 결과물을 제공하기 때문에 생성 대상이 되는 클래스의 constructor 는 private 으로 만든다

```java
public class Shape {
	
    //required parameters
    private String width;
    private String height;
	
    //optional parameters
    private boolean hasRadius;
 
    public String getWidth() {
        return width;
    }
 
    public String getHeight() {
        return height;
    }
 
    public boolean hasRadius() {
        return hasRadius;
    }
	
    private Shape(ShapeBuilder builder) {
        this.width = builder.width;
        this.height = builder.height;
        this.hasRadius = builder.hasRadius;
    }
	
    //Builder Class
    public static class ShapeBuilder {
 
        // required parameters
        private String width;
        private String height;
 
        // optional parameters
        private boolean hasRadius;
		
        public ShapeBuilder(String width, String height) {
            this.width = width;
            this.height = height;
        }
 
        public ShapeBuilder hasRadius(boolean hasRadius) {
            this.hasRadius = hasRadius;
            return this;
        }
		
        public Shape build(){
            return new Shape(this);
        }
 
    }
 
}
```

위 예시에서 확인할 수 있듯이 Shape class 는 setter 없이 getter 만 가진고 public constructor 를 가지고 있지 않는다.  
그렇기 때문에 Shape 객체를 얻기 위해서는 오직 ShapeBuilder class를 통해서만 가능하다.

```java
public class BuilderPatternTest {
 
    public static void main(String[] args) {
        Shape shape = new Shape.ShapeBuilder("100", "100")
                .hasRadius(true)
                .build();
    }
 
}
```

위 실제 사용에서 확인할 수 있듯이 Shape 객체를 얻기 위해 ShapeBuilder 를 사용하고 있으며 **필수 값인 width와 height 속성에 대해서는 constructor** 로 받고  
**Optional한 값인 hasRadius 에 대해서는 method** 를 통해 받고 있다.

---

Builder pattern 은 굉장히 많이 사용되는 패턴이다.

- android 에 있는 dialog 에서 확인할 수 있다
- Retrofit, OkHttp 등과 같은 다양한 오픈소스에서 확인할 수 있다

