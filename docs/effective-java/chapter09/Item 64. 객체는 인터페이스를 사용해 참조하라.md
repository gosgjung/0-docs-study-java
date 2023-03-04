---
layout: default
title: Item 64. 객체는 인터페이스를 사용해 참조하라
nav_order: 8
has_children: false
parent:  Ch09. 일반적인 프로그래밍 원칙
grand_parent: Effective Java
permalink: /docs/effective-java/chapter09/item61
layout: default
---



# Item 64. 객체는 인터페이스를 사용해 참조하라
{: .no_toc }

> `아이템 51` 에서는 "매개변수 타입으로 클래스가 아니라 인터페이스를 사용하라" 는 내용을 다룬다. 이와 비슷하게 이번 아이템은 "객체는 인터페이스를 사용해 참조하라"는 내용을 다룬다.<br>
<br>

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

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
적합한 인터페이스만 있다면 매개변수 뿐 아니라 반환값, 변수, 필드를 전부 인터페이스로 선언하자. 객체의 실제 클래스를 사용해야 할 상황은 '오직' 생성자로 생성할 때 뿐이다.<br>

객체를 인터페이스 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해진다.<br>

선언 타입과 구현 타입을 동시에 바꿀수 있으니 변수를 구현 타입으로 선언해도 괜찮을 거라고 생각할 수도 있다. 하지만 이렇게 할 경우, 어떤 경우에는 프로그램이 컴파일되지 않을 수 있다.<br>

예를 들면, 클라이언트에서 기존 타입에서만 제공하는 메서드를 사용했거나, 기존 타입을 사용해야 하는 다른 메서드에 그 인스턴스를 넘겼다고 해보자. 새로운 코드에서는 컴파일되지 않는다.<br>

<br>실전에서는 주어진 객체를 표현할 적절한 인터페이스가 있는지 찾아서 그 인터페이스로 참조하면, 더 유연하고 세련된 프로그램을 만들 수 있다.<br>

적합한 인터페이스가 없다면, 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인(상위의)클래스를 타입으로 사용해야 한다.<br>
<br>


### e.g. HashSet, LinkedHashSet
{: .fs-6 .fw-700 }
아래는 객체를 구체 클래스 타입으로 받는 좋지 않은 예이다.<br>

```java
HashSet<Son> sonSet = new HashSet<>();
```

`HashSet` 인스턴스를 구체 타입인 `HashSet` 으로 받고 있다.<br>

아래는 `HashSet` 의 인스턴스를 인터페이스 타입으로 받는 좋은 예이다.<br>

```java
Set<Son> sonSet = new LinkedHashSet<>();
```

이렇게 객체를 인터페이스를 타입으로 받는 것은 좋은 습관이다. 인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해진다.<br>

위의 선언은 아래와 같이 작성하면 다른 클래스의 생성자를 호출하도록 바꿔주는 것만으로 `sonSet` 을 사용하는 다른 코드의 변경 없이 구체클래스의 변경을 할 수 있다.<br>

```java
Set<Son> sonSet = new HashSet<>();
```
<br>

### 인터페이스 타입으로 참조하기에 좋지 않은 예
{: .fs-6 .fw-700 }

위에서 사용한 예제를 보자.

```java
Set<Son> sonSet = new LinkedHashSet<>();
Set<Son> sonSet = new HashSet<>();
```

`LinkedHashSet` 은 순서를 보장하는 `Set` 이다. 만약, `sonSet` 을 사용하는 다른 부분의 메서드나 코드가 순서를 보장하는 연산을 해야 할 경우에는, `sonSet` 의 구체 타입을 `HashSet` 타입으로 변경시 문제가 될 수 있다.<br>
<br>

### 구현 클래스 기반으로 참조해야만 하는 경우들
{: .fs-6 .fw-700 }

구현클래스 기반으로 참조해야 하는 예외상황들에 대해 책에서 이야기를 해주고 있다.
줄줄이 외울 필요가 없는 내용이지만, 책의 내용을 요약해두는 입장이기에 단순하게나마 정리를 해두었다. <br>
<br>

**1 ) 적합한 인터페이스가 없으면 클래스로 참조해야한다.**<br>
예를 들면 `String`, `BigInteger` 와 같은 값 클래스를 예로 들 수 있다. 이런 숫자 등을 표현하기 위한 `값` 클래스 종류의 값 클래스는 설계 시에 여러가지로 구현될 가능성을 생각해서 설계하는 일은 거의 없다. 이렇게 값을 표현하기 위한 클래스는 매개변수, 변수, 필드, 반환 타입으로 바로 사용해도 된다.<br>
<br>

**2 ) 클래스 기반으로 작성된 프레임워크가 제공하는 객체들**<br>
예를 들면 `OutputStream` 과 같은 `java.io` 패키지의 여러 클래스를 예로 들 수 있다.<br>
<br>

**3 ) 인터페이스에는 없는 특별한 메서드를 제공하는 클래스들**<br>
예를 들면 `PriorityQueue` 와 같은 클래스는 `Queue` 인터페이스에는 없는 `comparator` 메서드를 제공한다. 이런 경우는 구체 클래스인 `PriorityQueue` 클래스를 사용해야 한다.<br>
<br>

