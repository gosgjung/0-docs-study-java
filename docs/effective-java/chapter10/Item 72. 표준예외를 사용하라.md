---
layout: default
title: Item 72. 표준예외를 사용하라
nav_order: 4
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item72
layout: default
---



# Item 72. 표준예외를 사용하라
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



### 표준 예외를 재사용하면 얻는 장점들
{: .fs-6 .fw-700 }

- 작성한 API가 다른 사람이 익히고 사용하기 편리해진다.
  - 많은 프로그래머에게 보편적으로 익숙해진 규약을 그대로 따르기 때문
- 다른 곳에서 호출시 낯선 예외를 사용하지 않게 되므로 읽기 쉽게 된다
- 예외 클래스 수가 적을 수록 메모리 사용량도 줄고, 클래스를 적재하는 시간도 적게 걸린다.
<br>

### `Exception`, `RuntimeException`, `Throwable` , `Error` 는 직접 재사용하지 말아야 한다.
{: .fs-6 .fw-700 }
- 이 클래스들은 추상 클래스라고 생각해야 한다.
- 다른 예외들의 상위 클래스 역할을 하는데, 여러 성격의 예외들을 포함하고 있기에 안정적으로 테스트할수 없다는 단점이 있다.
<br>

### 잘 알려진 표준 예외
{: .fs-6 .fw-700 }

| 예외                              | 쓰임                                                   |
| --------------------------------- | ------------------------------------------------------ |
| `IllegalArgumentException`        | 허용되지 않는 인자값이 인자로 전달됐을 때              |
| `IllegalStateException`           | 객체가 메서드를 수행하기에 적절하지 않은 상태일 때     |
| `NullPointerException`            | `null` 을 허용하지 않는 곳에 `null` 값이 전달되었을 때 |
| `IndexOutOfBoundsException`       | 인덱스가 범위를 넘어섰을 때                            |
| `ConcurrentModificationException` | 허용하지 않는 동시 수정이 발견됐을 때                  |
| `UnsupportedOperationException`   | 호출한 메서드를 지원하지 않을 때                       |
| `NumberFormatException`           | 복소수, 유리수 다룰때, 또는 숫자 관련 에러발생시       |
| `ArithmeticException`             | 복소수, 유리수 다룰때, 또는 숫자 관련 에러발생시       |

<br>

`IllegalArgumentException` 

- 가장 많이 재사용되는 예외
- 호출자(caller)에서 인수로 부적절한 값을 넘길 때 던지는 예외다.

`IllegalStateException` 

- 대상 객체의 상태가 호출된 메서드를 사용하기에 적절하지 않을때 던지는 편이다.
- ex) 반복횟수를 지정하는 매개변수에 음수를 보내는 경우

`NullPointerException`

- null 을 허용하지 않는 갓에  null 이 전달될 경우에 사용하는 편

`IndexOutOfBoundsException` 

- 배열,리스트의 허용 범위를 넘어설 경우에 사용

`ConcurrentModificationException`

- 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 수정하려고 할 때 발생
- 동시 수정을 확실히 검출할 수 있는 안정된 방법은 없으나, 이 예외는 문제가 생길 가능성을 알려주는 정도의 역할을 한다

e.g.
```java
@Test
public void test(){
    Map<String, String> map = new HashMap<>();

    map.put("1", "100");
    map.put("2", "200");
    map.put("3", "300");

    map.entrySet()
        .stream()
        .forEach(e -> map.remove(e.getKey()));
}
```
<br>

출력결과<br>
```plain
java.util.ConcurrentModificationException
	at java.base/java.util.HashMap$EntrySpliterator.forEachRemaining(HashMap.java:1855)
	at java.base/java.util.stream.ReferencePipeline$Head.forEach(ReferencePipeline.java:762)
	at io.study.lang.javastudy2022ty1.effectivejava_temp.item72.WellKnownExceptionsTest.test(WellKnownExceptionsTest.java:21)
```
<br>

`UnsupportedOperationException`
- 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 사용
- 대부분 객체는 자신이 정의한 메서드를 모두 지원하므로 흔히 쓰이는 예외는 아니다.
- 보통은 구현하려는 인터페이스의 메서드 일부를 구현할 수 없을 경우 사용한다.

