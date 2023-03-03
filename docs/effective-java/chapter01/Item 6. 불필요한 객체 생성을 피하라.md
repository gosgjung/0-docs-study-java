---
layout: default
title: Item 6. 불필요한 객체 생성을 피하라
nav_order: 6
has_children: false
parent:  Ch01
grand_parent: Effective Java
permalink: /docs/effective-java/chapter01/item06
layout: default
---



# Item 6. 불필요한 객체 생성을 피하라
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



### 요약
{: .fs-6 .fw-700 }

기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라.
<br>

### 참고
{: .fs-6 .fw-700 }

- Item 1) 생성자 대신 정적 팩터리 메서드를 고려하라
- Item 17) 변경 가능성을 최소화하라
- Item 50) 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라

<br>


### 불필요한 객체가 생성되는 사례들 
{: .fs-6 .fw-700 }

#### 불변객체 (e.g. String)
{: .fs-6 .fw-700 }

당연한 이야기이지만, 굉장히 자주 언급되는 String의 Constant Pool 관련 이야기다.
String 을 아래와 같이 선언하면 객체를 매번 새로 사용하게 된다.
```java
String s = new String("새해 복 많이 받으세요");
```
<br>

위와 같은 코드는 아래와 같이 바꿔서 Constant Pool 을 사용할 수 있게끔 해줘야 한다.
```java
String s = "새해 복 많이 받으세요";
```

이렇게 하면 새로운 인스탄스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다. 같은 가상머신 안에서 이와 같은 문자열을 사용하는 모든 코드가 같은 객체를 재사용하는 것을 보장하게 된다. (JLS, 3.10.5)

자세한 내용은 이 리포지터리 내의 Java Basic 내에 따로 정리해둘 예정이다.<br>
<br>

#### 정적팩터리가 아닌 생성자를 사용하는 경우 (Item 1)
{: .fs-5 .fw-700 }

자주 사용되고, 유틸리티 성격의 클래스는 정적 팩터리 상태가 낫다.
생성자는 호출할 때마다 새로운 객체를 만들지만 정적 팩터리 메서드는 그렇지 않게 만들어둘 수 있다. 
e.g. Boolean(String) 을 쓰는 대신 Boolean.valueOf(String) 을 사용하는 경우를 예로 들 수 있다.<br>
<br>

#### 생성비용이 비싼 객체
{: .fs-5 .fw-700 }

e.g. String.matches
- 성능이 중요한 상황에서 반복문 내에서 사용하기에는 적합하지 않다.
<br>
<br>

#### 어댑터 패턴
{: .fs-5 .fw-700 }

e.g. Map 인터페이스의 keySet 메서드
- 객체가 불변이라면 재사용해도 안전함이 명백하다. 하지만 덜 명확하거나 직관에 반대되는 경우도 있는데, 어댑터 패턴을 사용하는 경우를 예로 들 수 있다.
- 흔히 어댑터 패턴을 뷰(View) 라고 이야기하기도 한다.
<br>
<br>

#### 오토박싱
{: .fs-5 .fw-700 }
- 오토박싱은 기본 타입과 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아니다. 
- 아이템 61
- 기본 자료형과 참조자료형을 사용했을 때의 의미는 같을 수 있지만 오토박싱이 for 문에서 발생할 때, for 문으로 순회해야 하는 양이 많을 경우 성능상으로 느려질 여지가 있다.
<br>
<br>

### 참고) 어댑터 패턴
{: .fs-6 .fw-700 }

실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체다. <br>
어댑터는 뒷단 객체만 관리하면 된다. 즉, 뒷단 객체 외에는 관리할 상태가 없기 때문에 뒷단 객체 하나당 어댑터 하나씩만 만들어지면 충분하다.<br>
