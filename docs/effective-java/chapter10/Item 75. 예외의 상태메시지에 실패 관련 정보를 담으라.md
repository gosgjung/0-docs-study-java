---
layout: default
title: Item 75. 예외의 상태메시지에 실패 관련 정보를 담으라
nav_order: 7
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item74
layout: default
---



# Item 75. 예외의 상태메시지에 실패 관련 정보를 담으라
{: .no_toc }
Item 75 에서 언급하는 StackTrace 에 에러 내용을 추가하는 이야기가 나온다. 이와 관련해서는  [Item73. 추상화 수준에 맞는 예외를 던지라](https://github.com/soon-good/study-effective-java-3rd/blob/develop/ITEM-73-%EC%B6%94%EC%83%81%ED%99%94-%EC%88%98%EC%A4%80%EC%97%90-%EB%A7%9E%EB%8A%94-%EC%98%88%EC%99%B8%EB%A5%BC-%EB%8D%98%EC%A7%80%EB%9D%BC.md#%EC%98%88%EC%99%B8-%EC%97%B0%EC%87%84-exception-chaining) 에서 언급하는 예외 연쇄(Exception Chaining) 에서도 비슷한 내용을 이야기해주고 있다.<br>
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
메서드가 던질 가능성이 있는 모든 예외를 문서화하라. 검사예외든 비검사 예외든 추상메서드든 구체 메서드든 모두 마찬가지다. 문서화에는 자바독의 `@throws` 태그를 사용하면 된다. 검사예외만 메서드 선언의 `throws` 문에 일일이 선언하고, 비검사 예외는 메서드 선언에는 기입하지 말자. 발생가능한 예외를 문서로 남기지 않으면 다른 사람이 그 클래스나 인터페이스를 효과적으로 사용하기 어렵거나 심지어 불가능할 수 도 있다.<br>
<br>

### `Stack Trace` 란?
{: .fs-6 .fw-700 }
자바 기반의 프로그램을 돌려보다보면 스택 트레이스(Stack Trace) 라는 것을 자주 접한다.<br>

Stack Trace 는 예외객체의 `toString` 메서드를 호출해서 얻어내는 문자열이다.

보통 클래스 이름 - 상태 메시지의 순으로 메시지가 구성되어 있다. <br>

StackTrace는 실패원인을 분석할때 매우 요긴하게 사용된다.<br>

따라서 예외의 toString 메서드에 실패에 대한 가능한 많은 정보를 담아 반환하는 것이 중요하다.<br>

일반적으로 Stack Trace 에는 호출한 다른 메서드들의 파일이름과 줄 번호까지 정확히 기록되어 있는 것이 보통이다. 따라서 문서와 소스코드로 얻을 수 있는 정보까지 길게 늘어놓을 필요까지는 없다.<br>

Stack Trace 는 상태 추적과정에서 가능한 많은 사람들이 볼수 있기에 보안에 민감한 비밀번호나 암호키 같은 정보까지 담지 않는 것이 좋다.<br>
<br>

### 예외의 생성자
{: .fs-6 .fw-700 }
책에서는 가급적 예외의 생성자에서 필요한 정보를 모두 받아서 상태 메시지까지 미리 생성해놓는 방식을 권장하고 있다. 하지만 모든 자바의 라이브러리가 이런 관례를 지키지는 않는다고 설명한다. 예를 들면 `IndexOutOfBoundsException` 생성자가 원칙을 지키지 않는 대표적 예라고 설명한다.<br>
<br>

### 접근자 메서드
{: .fs-6 .fw-700 }
예외는 실패와 관련된 정보를 얻을수 있는 접근자 메서드를 적절히 제공하는 것이 좋다. Unchecked Exception 을 사용하더라도, getter 가 제공되면, toString() 만으로 예외에 대한 정보를 얻어오기보다, 조금 더 상세한 정보를 활용할 수 있다는 점에서 유용하다는 점을 책에서는 언급하고 있다.<br>
<br>

