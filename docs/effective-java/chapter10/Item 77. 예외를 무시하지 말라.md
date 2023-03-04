---
layout: default
title: Item 77. 예외를 무시하지 말라
nav_order: 9
has_children: false
parent:  Ch10. 예외
grand_parent: Effective Java
permalink: /docs/effective-java/chapter10/item77
layout: default
---



# Item 77. 예외를 무시하지 말라
{: .no_toc }

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

### Item 77. 예외를 무시하지 말라
{: .fs-6 .fw-700 }
이번 챕터는 조금 단순한 내용이다. 예를 들면 아래와 같은 방식으로 `try ~ catch` 를 명시해두었지만, `catch` 블록에서는 아무런 처리도 하지 않는 경우가 있다.

```java
try{
    // ...
}
catch(SomeException e){
}
```

가급적이면 이렇게 `try~catch` 내의 `catch` 블록 내부를 비워놓지 말자. `catch` 블록을 비워두면 예외를 사용할 이유가 없다.<br>

또는 예외를 무시해야 하는 상황이라면 가급적 아래와 같이 예외를 무시하기로 한 이유를 명시한다. 그리고 가급적 예외 변수 명도 `ignored` 와 같이 명시적으로 변경해두어 예외 확인이 필요하지 않음을 확인할 수 있게끔해야 한다.

```java
Future<Integer> f = exec.submit(plannerMap::chromaticNumber);
int numColors = 4;

try{
    numColors = f.get(1L, TimeUnit.SECONDS);
}
catch (TimeoutException | ExecutionException ignored){
    // 기본값을 사용한다.(색상수를 최소화 하면 좋지만, 필수는 아니다.)
}
```

<br>

이런 원칙은 `Unchecked Exception` 에도 적용된다. `try ~ catch` 를 사용하는 것이 `Checked Exception` 에서는 강제로 사용해야 하고, Unchecked Exception 에서는 필수적으로 `try ~ catch` 를 사용하는 것은 아니다보니, 책에서는 10장 예외 전체에서 전반적으로 `이것은 Unchecked Exception에도 똑같이 적용된다.`  와 같은 이야기를 자주 언급하는 편이다.<br>
<br>
<br>
