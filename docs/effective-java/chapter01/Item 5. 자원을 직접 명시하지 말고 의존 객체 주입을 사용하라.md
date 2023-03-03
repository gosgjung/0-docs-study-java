---
layout: default
title: Item 5. 자원을 직접 명시하지 말고 의존객체 주입을 사용하라
nav_order: 5
has_children: false
parent:  Ch01
grand_parent: Effective Java
permalink: /docs/effective-java/chapter01/item05
layout: default
---



# Item 5. 자원을 직접 명시하지 말고 의존객체 주입을 사용하라
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


<br>
<br>

### 참고자료
{: .fs-6 .fw-700 }

- [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284)
- [Effective Java 한국어판 예제 깃헙](https://github.com/WegraLee)
- [조슈아 블로크 Effective Java 깃헙](https://github.com/jbloch/effective-java-3e-source-code/tree/master/src/effectivejava)
  <br>

<br>



### 핵심정리
{: .fs-6 .fw-700 }

단순 정적인 계산을 통해 정해진 입력에 항상 같은 결과를 내는 메서드의 경우 static 메서드를 통해 함수를 구현하거나 싱글턴 방식을 사용하는 것이 나쁜 방식은 아니다.<br>

하지만, 외부의 파라미터나 요청에 의해 동적으로 동작이 변해야 하는 경우 경우에 따라 객체 생성시 객체 의존성의 동작이 변하도록 생성자를 설계하게 되며, 이 때 의존성 주입을 사용하는 것이 관례 중 하나로 자리 잡았고 스프링 프레임워크 역시 의존성 주입을 사용하고 있다.<br>

<br>



### 정적 유틸리티 클래스, 싱글턴
{: .fs-6 .fw-700 }

흔히 애플리케이션 곳곳에서 쓰이는 연산이나 공통 로직들에 대해 static 한 util 클래스로 분리하거나, 싱글턴으로 구현하는 경우가 있다. 필요한 경우에 한해 이렇게 구현하는 경우가 많다.

또는 enum 내에 유틸리티 클래스를 작성하기도 한다.

몇가지 예를 들면 아래와 같다.

<br>



#### e.g. enum 으로 유틸리티 클래스를 구성
{: .fs-5 .fw-700 }

```java
public class SpellChecker{
  private static final Lexicon dictionary = ...;
  private SpellChecker(){} // 객체 생성 방지
  
  public static boolean isValid(String word){ ... }
  public static List<String> suggestions(String typo) { ... }
}
```

<br>



#### e.g. 싱글턴을 사용해서 구성하는 경우
{: .fs-5 .fw-700 }

```java
public class SpellChecker{
  private final Lexicon dictionary = ... ;
  
  private SpellChecker(...) {}
  public static SpellChecker INSTANCE = new SpellChecker(...);
  
  public boolean isValid(String word){...}
  public List<String> suggestions(String typo){...}
}

```
<br>
<br>


### 의존 객체 주입방식
{: .fs-6 .fw-700 }

컴파일 타임에 정적인 연산을 하는 것은 정해진 입력에 대해 정해진 동작만을 하는 동작을 하는 경우에 적합하다. 하지만, 실행시간에 외부의 입력에 따라 다른 동작을 취하도록 다형성이 필요한 경우라면 정적인 연산보다는 의존 객체 주입방식이 더 적절하다.<br>

의존 객체 주입을 하기 위해서는 객체를 생성해야 한다. 객체를 생성하는 대표적인 방식들은 아래와 같은 방식들이 있다.<br>

<br>

- 생성자에 객체를 넘겨주는 방식
- 정적 팩터리 (ITEM1) 에 객체를 넘겨주는 방식
- 빌더 (ITEM 2) 에 객체를 넘겨주는 방식
- 자원 팩터리를 넘겨주는 방식
  - 1 ) 팩터리 메서드 패턴을 구현한 객체를 넘겨주는 방식
  - 2 ) `Supplier<T>` 를 이용해 팩터리를 표현한 람다식을 전달받아 의존성을 주입받는 방식

<br>



이렇게 객체 생성을 통해 객체를 생성하는 함수를 마련해 둔후, 외부의 클라이언트에서 필요한 로직에 의존성 객체를 주입해준다. 그리고 이 의존성을 사용하는 해당 함수는 해당 객체를 추상타입으로 받아서, 어떤 객체인지에 대한 지식에 의존하지 않고 정해진 동작을 할 수 있음을 가정한 상태에서 독립적으로 자신의 역할을 수행할 수 있어야 한다.<br>

자세한 내용은 [오브젝트](http://www.yes24.com/Product/Goods/74219491) 를 보는 것이 더 나을 수도 있을 것 같다.<br>

<br>



#### 장점
{: .fs-5 .fw-700 }

- 불변성이 확보된다.
- 스레드 안전하다.
- 테스트에 용이하다.
  - Mockito 등을 이용한 테스트에 용이하다.

<br>



#### 단점
{: .fs-5 .fw-700 }

- 의존성 주입시 다형성을 갖기 위해서는 인터페이스 기반으로 파라미터를 설계해야 하는데, 개발 초창기의 요구사항이 명확하지 않은 상태에서는 다소 막막할 수 있다. 
  - 뭘할지도 정해지지 않았는데 추상화하는 경우<br>
  - 이런 이유로 많은 회사에서 초기 개발시에 정해진 기간안에 정해진 만큼의 산출물을 만들어 낸 후 고도화를 하는 작업이나 레거시 개편 작업을 많이 수행하는 듯 해보인다.

- 의존성이 수천 개 이상이 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다.
  - 객체 설계나 기능에 대한 정의가 오랫동안 정립되지 않는다면, 코드가 갈수록 어지러워지게 된다.

- 대거(Dagger), 주스(Guice), 스프링(Spring) 과 같은 프레임워크를 사용하면 의존객체 주입을 수월하게 할 수 있도록 편의성을 제공하기에 이러한 복잡합을 해결할 수 있다.<br>

<br>
