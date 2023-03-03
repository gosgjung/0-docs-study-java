---
layout: default
title: Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
nav_order: 3
has_children: false
parent:  Ch01
grand_parent: Effective Java
permalink: /docs/effective-java/chapter01/item03
layout: default
---



# Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
책에서 설명하고 있는 싱글턴을 만드는 방법은 3가지가 있다.
싱글턴은 주로 아래와 같이 무상태 객체 또는 유일하게 존재해야하는 컴포넌트일 경우에 사용한다.
- 무상태(stateless) 객체
- 설계상 유일해야 하는 시스템 컴포넌트
<br>
<br>

### 참고자료
{: .fs-6 .fw-700 }

- [이펙티브 자바 3/E](http://www.yes24.com/Product/Goods/65551284)
- [Effective Java 한국어판 예제 깃헙](https://github.com/WegraLee)
- [조슈아 블로크 Effective Java 깃헙](https://github.com/jbloch/effective-java-3e-source-code/tree/master/src/effectivejava)
<br>
<br>


### 싱글턴 클래스를 만드는 여러가지 방식들
{: .fs-6 .fw-700 }
싱글턴을 만드는 방식은 여러가지가 있다. 아래에 정리한 내용 중 4 번째 방식은 Bill Pugh 라는 사람이 고안한 방식인데, Effective Java 에서는 언급하고 있지 않은 내용이다.

- 1 ) private 생성자를 가진 클래스 내의 인스턴스를 즉시 생성해서 초기화 하는 방식이다.
  - 좋은 방식은 아니다.
  - 병렬프로그래밍 환경에서 객체가 두개 이상 생길 수 있다는 단점이 있다.
- 2 ) 정적 팩터리 메서드를 public static 멤버로 제공한다.
  - 흔히 getInstance() 라는 이름의 메서드로 제공하는 방식을 이야기한다.
  - 이 방식 역시 병렬 프로그래밍 환경에서 객체가 두개 이상 생길 수 있다는 단점이 있다.
- 3 ) 열거 타입으로 선언하는 것이다.
  - 열거타입으로 싱글턴을 구현할 경우 리필렉션 공격이나 복잡한 직렬화 상황에 대해서 제 2의 인스턴스가 생성되는 것을 막아준다고 책에서는 설명하고 있다.
  - 하지만, 특수한 경우가 아니라면 사용하지 않는 것이 좋다.
- 4 ) inner class 에서 멤버 필드의 선언과 동시에 객체를 생성해 초기화하는 방식
  - 추천되는 방식이다.
  - Bill Pugh Singleton 방식이다. 
  - 참고
    - [dzone.com - Singletons: Bill Pugh Solution or Enum](https://dzone.com/articles/singleton-bill-pugh-solution-or-enum)
    - [Yaboong.github.io](https://yaboong.github.io/design-pattern/2018/09/28/thread-safe-singleton-patterns/)
<br>

### 싱글턴 클래스의 단점 
{: .fs-6 .fw-700 }
싱글턴 클래스의 동작을 테스트하기 쉽지 않다. 이런 경우 래퍼(Wrapper) 매서드를 작성해서 Mocking 이 불가한 문제를 해결할 수 있다.

e.g.
```java
public class StockPrice{
  // ...
  
  public List<ServerTime> getServerTimeList(String countryCode){
    ServerStockTime.getInstance().getStockTimeList(countryCode);
  }
}
```

