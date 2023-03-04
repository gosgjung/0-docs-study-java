---
layout: default
title: Item 70. 필요없는 검사 예외 사용은 피하라
nav_order: 2
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item70
layout: default
---



# Item 70. 필요없는 검사 예외 사용은 피하라
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

- 꼭 필요한 곳에만 사용한다면 Checked Exception 은 프로그램의 안정성을 높여준다.
- 하지만, 남용하면 쓰기 어려운 API 로 전락한다.
- 해당 메서드를 호출하는 caller 측에서 예외를 복구할만한 방법이 없다면 `Unchecked Exception` 을 던지는 것이 낫다.
  - 쉽게 이야기하자면, 메서드 구현시, caller 측에서 `try ~ catch`  블록의 `catch` 구문 내에서 적절히 처리할 만한 방법이 없다면, `Unchecked Exception` 을 던지도록 구현하는 것이 낫다.
  - 예외 번역을 하는 경우를 예로 들 수 있다. (참고: [Item 73. 추상화 수준에 맞는 예외를 던지라.](https://github.com/soon-good/study-effective-java-3rd/blob/develop/ITEM-73-%EC%B6%94%EC%83%81%ED%99%94-%EC%88%98%EC%A4%80%EC%97%90-%EB%A7%9E%EB%8A%94-%EC%98%88%EC%99%B8%EB%A5%BC-%EB%8D%98%EC%A7%80%EB%9D%BC.md))
- 만약 복구가 가능하고, 그 처리를 caller 측에서 해야 한다면, Checked Exception 을 적용하기에 앞서서 `Optional` 을 반환할지 먼저 검토하자.
- `Optional` 을 사용하고 나서도 상황을 처리하기 위한 충분한 정보를 제공할 수 없다면 `Checked Exception` 을 던지자.
<br>

### CheckedException, try/catch, throws
{: .fs-6 .fw-700 }

어떤 메서드 하나를 `Checked Exception` 으로 선언했다고 해보자. 이 메서드를 호출하는 측에서는 아래와 같은 두가지 방식 중의 하나로 처리를 수행하게 된다.

- `try ~ catch` 블록에서 예외를 처리
- `throws` 해서 바깥으로 예외를 전파

<br>

**`try ~ catch` 를 사용하는 경우**

```java
} catch (TheCheckedException e){
    e.printStackTrace();
//    System.exit();
}
```

<br>

**throw 를 사용하는 경우**

```java
catch(TheCheckedException e){
    throw new AssertionError();
}
```

<br>

### Checked Exception 의 단점
{: .fs-6 .fw-700 }
한편, Checked Exception(검사 예외) 는 과하게 사용하면 오히려 쓰기 불편한 API가 된다.<br>

더욱이 `Checked Exception` 이 발생하는 메서드는 스트림 안에서 직접 사용할 수 없기 때문에(`아이템 45 ~ 48`) 자바 8 부터는 부담이 더욱 커졌다.<br>

책에서는 Checked Exception 을 하나만 던지는 메서드를 처리하기 위해 `try ~ catch` 문을 사용하는 것은 어느정도의 불편함이 더 커진다고 언급하고 있기도 하다.<br>
<br>


### `Checked Exception` 을 던지는 코드를 리팩토링 하는 다양한 방법들
{: .fs-6 .fw-700 }
중요한 코드가 아니라면 Checked Exception 을 던지는 코드를 피해하는 것이 좋겠지만, Checked Exception 을 `throw` 하는 메서드를 처리해야 하는 경우가 있다. 이와 같은 코드를 조금 더 유연한 구조로 바꾸기 위해서는 아래와 같은 방법들이 사용된다.<br>
<br>
- `Optional` 을 사용하는 방식
  - Item 55. 옵셔널 반환은 신중히 하라.를 참고하자.
  - 예외가 발생했을 때 예외가 발생한 이유를 알려주는 부가정보를 담을 수 없다는 점은 단점이다.
- `Checked Exception` 을 던지는 메서드를 2개로 분리해,`Unchecked Exception` 을 던지는 메서드로 변경하고, 예외가 던져질지 여부를 `boolean` 조건값으로 체크해 분류하는 방식
  - 아래에 예제를 정리했다.

예외가 던져질지 여부를 체크해서 `boolean` 값을 리턴하는 `actionPermitted()` 메서드로 예외 여부를 먼저 검사하고, 예외가 발생하지 않을 경우는 그대로 동작을 수행한다. 만약 `actionPermitted()` 메서드로 검사한 값이 `false` 라면 예외 처리를 수행하는데, 아래 코드에서는 `else {...}` 구문이 이것에 해당하는 부분이다.<br>

```java
if(obj.actionPermitted(args)){
    obj.action(args);
}
else{
    ... // 예외 상황에 대처
}
```
<br>

### 개인적인 생각
{: .fs-6 .fw-700 }

Effective Java 의 예외에 관련된 챕터에서는 체크드 익셉션 보다는 언체크드 익셉션을 사용하거나 체크드 익셉션을 사용해야 한다면 언체크드 익셉션으로 번역하라는 등의 지침들을 설명해준다. (바로 뒤의 여러 Item 들에 걸쳐서 설명해주고 있다.)

예외를 공부해야 하는 주된 이유는 스프링을 사용한다면, 스프링의 트랜잭션이 롤백하게 될지 커밋하게 될지를 파악해야 하기 때문에 중요하다.
<br>

### 스프링에서의 예외 발생시 커밋/롤백 기준
{: .fs-6 .fw-700 }

{: .important } 
**!!TODO!!** 아래 내용은 JPA,Querydsl 을 공부하면서 Spring Transaction 에 대한 내용도 정리를 했던 문서가 있는데 다시 정리가 필요해서 현재는 private 으로 처리해둔 상태다. 정리가 완료된다면 참고자료 링크로 추가해둘 예정. 꼭!! 링크로 추가해두자!!! ㅠㅠ
<BR>

스프링 트랜잭션에서는 기본적으로는 아래의 기준으로 예외를 처리하고 있다.<br>

Unchecked Exception 
- Rollback 한다.
- 런타임에서의 동적인 상횡에서의 Exception
- 확인되지 않은 예외로 간주하고, 이것에 대한 예외처리를 프로그래머가 작성한 예외처리 로직에서 수행되도록 유도
- e.g. IllegalArgumentException, ....
<br>

Checked Exception 
- Commit 한다.
- 컴파일 타임에서의 Exception
- 확인된 예외로 간주하고, 이것에 대한 예외처리 코드가 미리 작성되어 있다고 간주.
- e.g. 주문 완료시 잔액부족 → DB에 결과 기록 → 주문정보에서 결재유도


