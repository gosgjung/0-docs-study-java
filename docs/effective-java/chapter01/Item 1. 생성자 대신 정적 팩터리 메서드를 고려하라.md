---
layout: default
title: Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라
nav_order: 1
has_children: false
parent:  Ch01
grand_parent: Effective Java
permalink: /docs/effective-java/chapter01/item01
layout: default
---

# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라.
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


여기서 이야기하는 정적 팩터리 메서드는 디자인 패턴에서 이야기하는 팩터리 메서드(Factory Method)가 아니다. 디자인 패턴에는 여기서 이야기하는 정적 팩터리 메서드 패턴이 없다.


### 핵심정리
{: .fs-6 .fw-700 }
정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있다. 따라서 각각의 상대적인 장단점을 이해하고 사용하는 것이 좋다. 정적 팩터리를 사용하는 게 유리한 경우가 더 많은 경우도 있기때문에, 무작정 pubilc 생성자를 제공하던 습관이 있다면 고치자.<br>



### [](#header-3) 정적 팩터리 메서드란?
{: .fs-6 .fw-700 }
객체 생성을 보통 생성자로 하는데, 이렇게 생성자를 통해 객체를 생성하려면 그 객체에 대해 알아야 하는 지식이 많다. 외부 호출 단에서 객체의 생성방법을 알고 있어야 한다. 이런 경우 객체 내에 정적 팩터리 메서드를 만들어두고 해당 팩터리 메서드를 통해 객체를 생성하게끔 한다.

생성자를 그대로 사용하면 생성자의 순서를 고치는 것이나, 인자 갯수를 수정하는 것이 다른 코드에 직접적으로 영향을 줄수 있다는 단점이 있다. 객체 생성을 정적 팩터리 메서드를 통해서 수행하면, 객체의 생성자를 수정할 때 정적 팩터리 메서드를 수정하면 되기에 수정 범위가 객체 내로 한정된다는 장점이 있다.

이런 이유로 정적 팩터리를 이용해 객체를 생성하면 좋다.

객체의 의존성을 주입하는 것은 클라이언트 레벨에서 주입할수 있어야 한다. 라든가 여러가지 이야기들이 있지만, 이번 문서에서는 정적 팩터리 메서드 하나에 한정해서 개념을 정리하기로 했다.<br>
<br>


#### 정적팩터리 메서드 들의 일반적인 예
{: .fs-5 .fw-700}
Date 라는 클래스 내에 Date 인스턴스를 생성하는 `from` 이라는 정적 팩터리 메서드를 사용하는 예는 아래와 같다.

```java
Date d = Date.from(instant);
```



이번에는 아래와 같이 `of` 라는 정적 팩터리 메서드를 통해 객체를 생성한다.

```java
Set<Rank> fareCards = EnumSet.of(JACK, QUEEN, KING);
```



이 외에도 of, from, newInstance, new클래스명 등과 같은 네이밍 컨벤션을 사용해 정적 팩터리 메서드를 정의하는 편이다
<br>


#### 네이밍 컨벤션
{: .fs-5 .fw-700}

- `from()` 
  - 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드에 사용
  - ex) `Date d = Date.from(instant)`
- `of()` 
  - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계메서드
  - 33ex) `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING)` 
- valueOf\(\)
  - from 과 of 의 더 자세한 버전
  - ex) `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE)` 
- instance, getInstance\(\)
  - (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
  - ex) `StackWalker Luke = StackWalker.getInstance(options)` 
- create, newInstance\(\)
  - instance 혹은 getInstance와 같지만 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
  - ex) `Object newArray = Array.newInstance(classObject, arrayLen)`
- get[Type명]\(\)
  - getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
  - `Type` 은 팩터리 메서드가 반환할 객체의 타입이다.
  - ex) `FileStore fs = Files.getFileStore(path)` 

- new[Type명]\(\)
  - newInstance 와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용한다.
  - `Type` 은 팩터리 메서드가 반환할 객체의 타입이다.
  - ex) `BufferedReader br = Files.newBufferedReader(path)` 

- [type명]\(\)
  - getType과 newType 의 간결한 버전
  - ex) `List<Complaint> litany = Collections.list(legacyLitany)` 
  <br>



### 정적팩터리 메서드의 장점
{: .fs-6 .fw-700}

- 첫 번째, 이름을 가질 수 있다.
- 두 번째, 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- 세 번째, 반환타입의 하위 객체를 반환할 수 있는 능력이 있다.
- 네 번째, 입력 매개 변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 다섯 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

 

#### 1) 첫번째, 이름을 가질 수 있다.
{: .fs-5 .fw-700}

##### 생성자에 인자가 매우 많을 경우의 단점
{: .fs-4 .fw-700}

생성자에 인자가 많으면 입력매개변수들을 모두 기억하기 쉽지 않다. 또는 외부 로직을 작성하는 도중에 해당 생성자의 내부 구현을 확인해야 한다. 또한 생성자의 내부 구현이 변경되었을 때 사이드 이펙트에 대해서도 고려해야 한다.

생성자 시그니처 하나당 생성자 하나만 만들수 있다는 점 역시 단점이다. 입력 매개변수들의 순서를 다르게 하거나 텔레스코핑을 할 수 도 있지만 제한적인 우회방법이다.
<br>


##### e.g. BigInteger(int, int, Random) → BigInteger.probablePrime()
{: .fs-4 .fw-700}
BigInteger.probablePime() 은 java 4 에서 추가된 정적 팩터리 메서드다.<br>
<br>


##### 정적 팩터리 메서드는 이름을 가질수 있다.
{: .fs-4 .fw-700}
 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 이 정적 팩터리 메서드의 이름을 각각의 차이를 잘 드러내는 이름으로 지어주면 좋다.<br>
<br>



#### 2) 두 번째, 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
{: .fs-5 .fw-700}

> 참고 : 싱글턴 (아이템 3), 인스턴스화 불가(아이템 4), 불변값 클래스(아이템 17), 플라이웨이트패턴(Gamma 95), 열거타입(34)

<br>

##### 인스턴스 통제(instance-controlled) 메서드
{: .fs-4 .fw-700}

정적 팩터리 메서드 처럼 반복되는 요청에 같은 객체를 반환하는 역할을 하는 메서드를 제공하는 클래스를 인스턴스 통제(instance-controlled) 클래스라고 한다.

인스턴스를 통제하면 클래스를 싱글턴으로 만들 수 있다. 또는인스턴스화 불가로 만들 수도 있다. <br>

이 외에도 정해진 값의 범위만을 리턴한다면, 객체 내에 미리 생성해둔 값의 범위를 리턴하도록 해서 불필요한 객체 생성을 방지할 수 있다.<br>

즉, 인스턴스의 생성을 통제 할 수 있게 된다.



##### e.g. Boolean.valueOf(boolean)
{: .fs-4 .fw-700}

Boolean 값은 새로 생성할 필요 없이 True, False 에 대해 미리 생성되어 있는 불변객체를 리턴해주면 된다. 정적 팩터리 메서드 valueOf(boolean)은 이렇게 객체를 무분별하게 생성하지 않고 미리 생성되어 있는 불변객체를 리턴한다.



> 참고) 불변클래스 타입의 인스턴스 생성(Item 17)
> 불변 클래스는 인스턴스를 미리 만들어놓거나 새로 생성한 인스턴스를 재활용하는 방식으로 불필요한 객체 생성을 피한다. 위의 Boolean.valueOf(boolean) 이 대표적인 예다.

<br>



#### 2) 세 번째, 반환타입의 하위타입 객체를 반환할 수 있는 능력이 있다.
{: .fs-5 .fw-700}

> 참고 : 아이템 20, 아이템 64

