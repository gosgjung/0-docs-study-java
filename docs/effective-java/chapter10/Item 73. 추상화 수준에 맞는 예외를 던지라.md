---
layout: default
title: Item 73. 추상화 수준에 맞는 예외를 던지라
nav_order: 5
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item73
layout: default
---



# Item 73. 추상화 수준에 맞는 예외를 던지라
{: .no_toc }

검사 예외라는 용어는 되도록 `Checked Exception` 이라는 용어로 바꿔서 정리하기로 했다.<br>

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
하위 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 이때 예외 번역을 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다. (아이템 75)<br>

### 저수준 예외를 바깥으로 전파하는 것의 단점
{: .fs-6 .fw-700 }
메서드가 저수준 예외를 처리하지 않고 바깥으로 전파하는 경우
- 수행하려는 일과 관련 없어보이는 예외가 튀어나온다.
- 내부 구현 방식을 드러내어, 해당 내용에 맞게 윗 레벨 API를 개발해야 하게끔 만들어 윗 레벨 API를 오염시킨다.
<br>

### 예외번역 (Exception translation)
{: .fs-6 .fw-700 }
상위 계층에서 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꾼다.<br>

책에서는 `AbstractSequentialList` 클래스의 `E get (index)` 메서드가 수행하는 예외 번역을 예로 들고 있다.<br>

<br>

```java
public abstract class AbstractSequentialList<E> extends AbstractList<E> {
    // ...
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
    // ...
}
```

get 메서드에서 발생하는 `NoSuchElementException` 예외를`IndexOutOfBoundsException` 로 상위 호출단에서 에러 판단이 쉽도록 번역해서 처리하고 있다.<br>
<br>

### 예외 연쇄 (Exception Chaining)
{: .fs-6 .fw-700 }
문제의 근본 원인(`cause`)인 저수준 예외를 고수준 예외에 실어보내는 방식이다. <br>

예외 번역시 저수준 예외가 예외 처리에 도움이 되면 예외 연쇄를 사용한다.<br>

- 문제의 근본 원인(`cause`)인 저수준 예외를 고수준 예외에 실어보내는 방식이다.
- 이렇게 할  경우 `Throwable` 의 `getCause()` 메서드로 저수준 예외를 꺼내볼 수 있다.
- 예외 연쇄를 사용하면 생성자를 연쇄적으로 호출하게 되면서 최종적으로 `Throwable(cause)` 가 호출된다.
- 또는 `initCause(cause)` 메서드를 사용한다.