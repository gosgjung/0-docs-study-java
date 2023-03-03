---
layout: default
title: Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라
nav_order: 2
has_children: false
parent:  Ch02
grand_parent: Effective Java
permalink: /docs/effective-java/chapter02/item02
layout: default
---



# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라.
{: .no_toc }

정적 팩터리와 생성자는 똑같은 제약이 하나 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 점이다. [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284) 에서는 이런 경우에 대한 예제로 영양정보 표현 클래스를 예제로 들고 있다.<br>

그리고 이렇게 객체 생성시 초기화해야 하는 인자의 수가 많거나, 클라이언트 코드에 따라서 특정 인자들만 초기화해야 하는 코드들이 산재해 있을 경우 [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284) 에서는 아래의 세가지 해결책을 제시해주고 있고, 그 중 세 번째 해결책인 빌더 패턴을 사용할 것을 권장하고 있다.<br>

- 첫 번째 대안 - 점층적 생성자 (Telescoping Constructor)
- 두 번째 대안 - 자바빈즈 패턴 (JavaBeans Pattern)
- 세 번째 대안 - 빌더 패턴 (Builder Pattern)

<br>




## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---



### 참고자료
{: .fs-6 .fw-700 }

- [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284)
- [Effective Java 한국어판 예제 깃헙](https://github.com/WegraLee)
- [조슈아 블로크 Effective Java 깃헙](https://github.com/jbloch/effective-java-3e-source-code/tree/master/src/effectivejava)

<br>




### 핵심정리
{: .fs-6 .fw-700 }

생성자/정적 팩터리가 처리해야 하는 매개변수가 많다면, 보통 빌더를 선택하는 것이 권장되는 방식이다.<br>

생성자 또는 정적 팩터리가 처리해야 할 매개변수가 많을 경우 보통 아래의 세가지 방법으로 해결가능하다. 아래의 세가지 방법 중 보통 **빌더(Builder)를 사용하는 방식**이 권장되는 방식이다.

- 점층적 생성자(Telescoping Constructor)를 사용하기
  - 가독성 역시 좋지 않고, 의도치 않은 실수를 발생시키기 쉽다.
- 자바빈즈 패턴 (Java Beans Pattern)
  - 기본 생성자나 기타 생성자로 객체를 생성한 후에 Setter 를 이용해 필드를 추가하는 방식이다.
  - 이렇게 하면 객체 생성 후에 객체 안에 있는 값이 계속해서 바뀌어 가변적인 코드가 된다는 점이 단점으로 작용한다.
- 빌더 패턴(Builder Pattern)
  - 장점
    - 객체 생성 전에 inner class 내에서 버퍼를 두어 세팅을 하고 build() 를 호출하는 시점에 객체가 생성되도록 한다는 점을 생각하면, 멀티스레딩 환경에서 thread safe 하다는 점이 장점이다.
    - 매개변수 중 대부분의 필드가 필수가 아니거나, 몇몇 필드만을 필수로 해야 하는 경우 빌더패턴을 사용하면 훨씬 수월하다.
    - 자바 빈즈(Java Beans) 를 사용할 때보다는 코드가 안전하다.
  - 단점 
    - 필드가 굉장히 많을 경우는 필드들을 일일이 찾아서 입력해줘야 하는데, 이때 실수를 하기 쉽다.
      - 객체 생성 코드를 한번 만들때는 공수가 크게 들더라도, 작성해놓은 빌더를 사용하는 것은 편리해지기애, 초반에 공을 들여 빌더로 만들거나, 롬복의 @Builder를 사용하는 방법을 생각할 수 있을 것 같다.
    - 해결방법
      - 필수 인자를 정해놓고 초기화 되지 않으면 Exception 을 내서 의도치 않은 에러를 방지한다.
      - 롬복과 같은 라이브러리를 사용한다.
      - 데코레이터 패턴으로 한 번 더 감싼다. 시간이 허용되거나, 제품의 운영 연차가 쌓이면서 비즈니스 도메인이 확실해졌다면 이 방식을 선택할수도 있을 것 같다. 또는 고도화 프로젝트로 잡혔다면, 충분히 고려할 만한 생각이다.

<br>



### 첫 번째 대안 - 점층적 생성자 (Telescoping Constructor)
{: .fs-6 .fw-700 }

> 참고 : 아이템 51
객체 생성을 보통 생성자로 하는데, 이렇게 생성자를 통해 객체를 생성하려면 그 객체에 대해 알아야 하는 지식이 많다. 외부 호출 단에서 객체의 생성방법을 알고 있어야 한다. 이런 경우 객체 내에 정적 팩터리 메서드를 만들어두고 해당 팩터리 메서드를 통해 객체를 생성하게끔 한다.
<br>

영양 정보를 표현하는 NutiritionFact 라는 클래스가 있다고 해보자. 식품 포장을 위한 영양 정보를 표현하기 위한 클래스가 필요하다. 영양정보는 총 20개가 넘는 항목으로 이루어졌다고 해보자.<br>

> - 1회 내용량
> - 총 n회 제공량
> - 1회 제공량 당 칼로리
> - 총 지방, 트랜스지방, 콜레스테롤
> - 나트륨 
> - 그 외 다수...

<br>

각각의 항목은 필수 값이 있고, 선택값이 있다. 제품에 따라 선택값이 입력되어 생성되는 경우도 있고 필수값들만 지정해 객체를 생성할 수도 있다. 또는 선택 값에 해당하는 필드들을 각각 선택해서 사용해야 하는 경우 역시 있다. 이 경우 가장 간단한 해결방법은 점층적 생성자이다. 생성자를 여러 가지로 만들 필요가 없다면 점층적 생성자로 모두 해결 가능할 수 있다.<br>

예를 들면 아래와 같은 생성자 들을 예로 들 수 있다. ([참고](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/telescopingconstructor/NutritionFacts.java)) <br>

```java
package effectivejava.chapter2.item2.telescopingconstructor;

// 코드 2-1 점층적 생성자 패턴 - 확장하기 어렵다! (14~15쪽)
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
    
}
```

<br>



이렇게 여러 개의 생성자를 여러번 래핑해서 사용하는 것을 점층적 생성자 패턴(telescoping constructor pattern)이라고 이야기한다. 점층적 생성자 패턴을 사용하게 되면, 단점은 아래와 같다.

- 사용자가 원하지 않는 매개변수도 포함해야 하는데, 어쩔 수 없이 그런 매개변수에도 값을 지정해줘야 한다.
  - 위의 코드에서는 지방(fat)에 0 을 넘겼다.<br>
- 매개변수의 갯수가 많아지면, 클라이언트 코드를 작성하거나 읽기 어렵다.
  - 코드를 읽을 때 각 값의 의미가 무엇인지 헷갈리고, 매개변수가 몇 개인지도 주의해서 세어보아야 한다.
  - 타입이 같은 매개변수가 연달아 늘어서 있으면 찾기 어려운 버그로 이어질 수 있다.
  - 클라이언트가 실수로 매개변수의 순서를 바꿔서 건네주게 되면 컴파일러는 알아채지 못하고 결국 런타임에 엉뚱한 동작을 하게 된다. (아이템 51)

<br>



### 두 번째 대안 - 자바빈즈 패턴 (Java Beans Pattern)
{: .fs-6 .fw-700 }

> 참고) 아이템 17

매개변수가 없는 기본 생성자로 객체를 만든 후, setter 메서드를 호출해 원하는 매개변수의 값들을 직접 설정하는 방식이다.<br>

자바빈즈 패턴을 사용한 예제 ([참고](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/javabeans/NutritionFacts.java))<br>

```java
package effectivejava.chapter2.item2.javabeans;

// 코드 2-2 자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다. (16쪽)
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

<br>

점층적 생성자 패턴을 사용했을 때의 단점은, 각각의 생성자의 용도를 파악하기 어렵다는 점이었다. 클라이언트 측에서는 해당 생성자를 모두 열어봐야 한다. 점층적 생성자 패턴에서의 이런 단점들은 자바 빈즈 패턴에서는 어느 정도 해소되었다.<br>

하지만 setter 를 사용하는 자바빈즈 패턴은 심각한 단점을 가지고 있다. 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러개 호출해야 한다. 객체가 완전히 생성되기 전까지 일관성(consistency)이 무너진 상태에 놓이게 된다.<br>

<br>



**객체 freezing**<br>

이런 단점을 극복하기 위해 생성이 끝난 객체를 수동으로 얼리는(freezing) 개념 역시 [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284) 에서 언급되고 있다. 하지만, 정확하게 어디선가 개념이 정리된 것이 아니어서 이글을 쓰는 나도 이 freezing 이라는 개념에 대해 정확히 잘 모르겠다.[이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284) 에서도 freeze 메서드를 사용하더라도 컴파일러가 freeze 메서드를 확실히 호출해줬는지를 컴파일러가 보증할 방법이 없기에 런타임 오류에 취약하다고 설명하고 있다.<br>

<br>



### 세 번째 대안 - 빌더 패턴 (Builder Pattern)
{: .fs-6 .fw-700 }

>  참고 : 아이템 50, 아이템 75<br>

빌더패턴은 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 모두 가지고 있는 패턴이다.<br>

클라이언트에서 필요한 객체를 직접 만들기보다는 필요한 인자들을 차례로 전달해가고, 가장 마지막에 build 메서드를 호출해 필요한 객체를 생성하는 방식이다. 빌더 패턴이라는 다소 멋있어보이는(?) 영어 단어를 보면서 어려운 개념이라고 생각할지도 모르겠다. 그래서 간결하게 몇가지 문법적인 요소들만을 정리해보면 아래와 같다.

- 클래스 내에 inner class 인 Builder 라는 클래스를 정의해둔다.
  - 이 Builder 라는 클래스는 클래스 명을 꼭 Builder 로 지정할 필요는 없고, 다른 이름을 지정해도 되는데, 빌더 패턴을 사용하려 할 때, 네이밍 컨벤션이 보통 Builder 라는 이름을 붙이는 것이 관례여서 Builder 라는 이름을 붙이는 것이다.
- 그리고 inner class 인 Builder 의 생성자를 호출해 inner 클래스의 객체를 생성한다.
  - 만약 필수 값이 있다면 이 inner class 인 Builder 클래스의 인자값으로 지정해둔다.
  - 아래 예제에서는 serviingSize, servings 가 필수 인자값이고, Builder(int, int) 생성자로 세팅하고 있다.
- Inner class 인 Builder 생성자로 생성한 객체 내의 각각의 필드들을 세팅해준다.
  - 아래 예제에서는 calories, sodium, carbohydrate 필드를 inner class 내의 calories(int), sodium(int), carbohydrate(int) 메서드를 호출해 Builder 클래스 내에 임시적으로 값을 세팅해둔다.
- Inner class 인 Builder 클래스 내의 build() 메서드를 호출해 NutritionFacts 객체의 생성을 완료한다.
  - NutritioinFacts.Builder 클래스 의 build() 메서드는 자신의 멤버필드인 calories, sodium, carbohydrate, servingsize, servings 필드를 이용해 부모 클래스 NutiritionFacts 클래스의 멤버 클래스에 세팅하면서 NutiritionFacts 클래스의 객체 생성을 완료한다.

<br>



e.g. 기본적인 Builder 패턴

```java
package effectivejava.chapter2.item2.builder;

// 코드 2-3 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. (17~18쪽)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```

<br>



**빌더 패턴의 장점**<br>

- 가독성과 사용의 편리성
  - Builder 패턴은 다른 언어에서는 기본적으로 지원하는 이름이 있는 선택적 매개변수를 흉내낸 것인데, 확실히 Builder 패턴을 사용함으로써 가독성과 사용이 편리해진 것을 느낄 수 있다.
- 유효성 검사
  - Builder 클래스의 생성자와 각 필드를 세팅하는 메서드 각각에서 잘못된 필드에 대해 입력 매개변수에 대해 검사하고, build() 메서드 내에서도 각 매개변수에 대한 불변식(invariant) 를 검사할수 있게 된다.
  - 공격에 대비해 이런 불변식을 보장하려면, inner class 인 Builder 클래스에 임시로 세팅해둔 뒤로부터 각 필드들을 복사해와서 해당 객체 필드들을 검사하면 된다. (아이템 50)
  - Builder 패턴은 이렇게 생성자 내에 Builder 라는 내부클래스와 생성자 Builder(...) 을 둠으로써 입력 매개변수의 불변성과, 유효성 검사 측면에서 유리함을 가지고 있다.
  - 이렇게 유효성을 검사하면서 잘못된 입력인자가 있으면 IllegarArgumentException 을 던지거나, 자체 정의한 Exception 을 던지면 된다. (아이템 75)

<br>



**참고) 불변, 불변식이란?**

**불변(immutable, immutability)**<br>

- "어떠한 변경도 허용하지 않겠다." 는 의미.
- 가변(mutable) 객체와 대비되는 개념.
- 예) String 객체는 한번 만들어지면 절대 값을 바꿀 수 없는 불변객체다.

<br>

**불변식(invariant)**<br>

- 참고 : 아이템 50

- 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건.
- **"변경을 허용할 수는 있으나, 주어진 조건 내에서만 허용해야 한다는 의미"**
- ex)
  - List 클래스 : 리스트의 크기는 반드시 0 이상이어야 하는데, 한순간이라도 음수 값이 된다면 불변식이 깨진 것이다.
  - Period 클래스 : 기간을 나타내는 Period 클래스는 start 필드의 값이, end필드의 값보다 앞선 시간이어야 한다. 두 값이 역전 되면 역시 불변식이 깨진 것이다.<br>

- 가변 객체에도 불변식은 존재할 수 있다. 넓게 보면 불변은 불변식의 극단적인 예이다.

<br>



### 네 번째 대안 - 계층적 빌더 패턴, 추상 빌더 패턴
{: .fs-6 .fw-700 }

추상빌더 패턴은 빌더 패턴으로 생성하는 객체들의 공통적인 부분이 확실하게 보일 때 이 것을 abstract 클래스를 통해 공통화 하고, 구체클래스를 통해 다형성을 적용하는 방식이다. <br>

- 추상(abstract) 클래스로 빌더 패턴 형태의 클래스를 만든다. 
- 이 추상(abstract) 클래스를 상속하는 구체 클래스를 만든다.
- build 메서드를 추상메서드로 만든다.
  - 각 구현 클래스마다 객체를 생성할 때 사용하는 필드가 다를 수 있기 때문이다.

<br>



#### e.g. NyPizza.java
{: .fs-5 .fw-700}

- [예제 github](https://github.com/WegraLee/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item2/hierarchicalbuilder)<br>
- [Pizza.java](https://github.com/WegraLee/effective-java-3e-source-code/blob/master/src/effectivejava/chapter2/item2/hierarchicalbuilder/Pizza.java)<br>

<br>



```java
package effectivejava.chapter2.item2.hierarchicalbuilder;

import java.util.Objects;

// 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스 (20쪽)
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```

<br>



**addTopping(T)**<br>

Topping 이라는 Enum 들을 추가하는데, NYPizza 와 Calzone 라는 피자는 각각 사용하는 토핑이 다르다. 이 때 각각의 필드를 Topping 이라는 enum 으로 어느 정도의 명세화를 수행했고, 이 것을 addTopping(Topping topping) 이라는 함수로 공통화했다.<br>

<br>

**abstract Pizza build();**<br>

build() 메서드를 override 하는 각각의 NYPizza 와 Calzone 피자에서의 build 메서드는 각각 NYPizza, Calzone 피자를 리턴해야 하기에 각각의 구체 클래스에서 build() 메서드를 구현하도록 build() 메서드를 abstract 로 선언해두었다.<br>

<br>

**T self()**<br>

각각의 하위 클래스 에서 각각 자기 자신의 타입을 리턴해야 하는데, 이것을 위해 T self() 라는 추상메서드를 두었다. 이렇게 해서, NYPizza, Calzone 에서 각각 T self() 내에서 NYPizza.Builder, Calzone.Builder 를 리턴하게 된다.<br>





### 롬복의 @Builder
{: .fs-6 .fw-700 }

롬복은 정말 편리하면서도 어떤 때는 취약점을 만드는 맹점이 되기도 한다. 예를 들면 `@Builder` 를 생성자레벨이 아닌 클래스레벨에 선언해두는 경우를 예로 들 수 있다. 이 외에도 잘못 사용하는 예가 굉장히 많다. 다른 코드에서 오용하는 것으로 인한 사이드 이펙트가 발생할 수 있는 경우 역시 존재한다.

실무에서는 빌더를 직접 만들어서 사용했던적도 있고 @Builder 를 사용해본 경험이 모두 있다. 나중에 별도의 문서로 정리해둘 예정이다.  이 리포지터리에서 lombok을 잘못 사용하는 예에 대해서도 정리하기로 했기 때문에, 추후 정리가 된다면 이곳에 링크를 추가해둘 예정이다.

<br>



### 다른 언어들
{: .fs-6 .fw-700 }

**kotlin**<br>

kotlin 을 써보니 정말 편했었다. backing property 가 있다는 점도 편했다. 다만, backing property 는 자동으로 만들 수 있는 방법을 몰라서 일일이 손으로 구현하기는 하지만, 나중에 이것도 Refactor 툴이 생기지 않을까 싶다.<br>

kotlin 은 이름있는 매개변수를 사용가능하다. 디폴트 매개변수 역시 사용가능하다. 그래서 따로 빌더 패턴을 구현하지 않아도 되고, 매개변수의 순서를 지킬 필요도 없다.<br>

<br>



**c++, python, etc**<br>

이름있는 매개변수를 사용가능하다. 또한 디폴트 매개변수 역시 사용가능하다. 

<br>



