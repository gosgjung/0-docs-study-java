---
layout: default
title: Item 69. 예외는 진짜 예외상황에만 사용하라
nav_order: 3
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item69
layout: default
---



# Item 69. 예외는 진짜 예외상황에만 사용하라
{: .no_toc }

굉장히 특이한 케이스이지만 JVM 의 동작을 임의로 추론해서 실제 문법적인 용도를 벗어난 채로 예외를 사용하는 경우가 있다. 이번 챕터에서는 이런 경우에 대한 예제와 이에 따르게 될 부작용을 이야기한다.<br>

책에서 이야기하는 예제는 for문을 탈출하는 것에 대한 예제다.<br>

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

예외는 예외 상황에서 쓸 의도로 설계되었다. 정상적인 제어 흐름에서 사용해서는 안되며, 이를 프로그래머에게 강요하는 API 를 만들어서도 안된다.<br>
<br>

### 문제 도입
{: .fs-6 .fw-700 }

아래의 코드가 있다고 해보자.

```java
for(Mountain m : range){
    m.climb();
}
```

`range` 라는 배열의 모든 `Mountain` 객체를 `climb()` 하는 코드다.<br>

위의 코드를 최적화하기 위해 JVM 의 동작을 추측해서 아래와 같이 수정했다고 해보자.<br>
<br>

### 특이한 용도로 변형 (예외의 비정상적인 사용)
{: .fs-6 .fw-700 }

```java
try{
    int i = 0;
    while(true)
        range[i++].climb();
}
catch(ArrayIndexOutOfBoundsException e){
    
}
```
코드만 봐서는 용도가 무엇인지는 확인하기 쉽지 않다.<br>
`range` 라는 배열을 순회하는데, while 문에서 무한루프를 통해 순회하게끔 하고 있다. 이렇게 되면 i++ 를 반복하면서 i가 range의 마지막 요소를 넘어섰을 때 `ArrayIndexOutOfBoundsException` 예외를 내게 된다.<br>
이 코드는 while 문을 무조건 탈출한다는 것을 간접적으로 추론하고 파악해야 한다는 단점이 있다.<br>
<br>

**위 코드의 의도**<br>
위 코드가 예외를 써서 루프를 종료하려고 한 추론의 근거는 아래와 같다.<br>

- JVM 은 배열에 접근할 때마다 배열의 경계에 넘어서는지 검사를 한다.
- 그런데 반복문에서 배열을 순회시키려보니 반복문에서도 `index` 가 배열의 length 를 넘어서는지 검사를 해야 한다. 위 코드를 작성한 프로그래머는 `반복문 내에 배열 index 조건 검사식을 빼도 되겠다.` 하는 판단을 한다.
- 그 결과로, while 문 내에 배열을 순회시키지만, index가 넘어섰는지는 예외가 발생했는지 여부로 판단하게끔 하는 코드를 만들어내게 되었다.

<br>

**위 코드의 잘못된 점 3가지**<br>

실제로는 예외를 사용해 조금은 이상하게 작성한 코드가 일반적인 for/while 문을 사용할 때보다 느리다. 요소 100개 짜리 배열로 테스트해봐도 2배 정도 느리다.

- 예외는 예외 상황에서만 사용할 용도로 설계된 키워드다.
  - JVM 의 동작을 예측해서 예외를 만들어내는 코드를 구현했지만, 예외에 관련된 코드가 JVM 상으로 최적화될지는 장담하기 어렵다.
- 코드를 `try ~ catch` 블록안에 넣으면 JVM 이 적용할 수 있는 최적화가 제한된다.
- 배열을 순회하는 표준 순회 코드(첫번째 예제)는 위 코드를 작성한 프로그래머가 추측한 대로 중복되는 index 조건 검사를 실제로 중복해서 검사하지 않는다. JVM 이 알아서 최적화해 없애준다.
<br>


