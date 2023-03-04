---
layout: default
title: Item 63. 문자열 연결은 느리니 주의하라
nav_order: 7
has_children: false
parent:  Ch09. 일반적인 프로그래밍 원칙
grand_parent: Effective Java
permalink: /docs/effective-java/chapter09/item61
layout: default
---



# Item 63. 문자열 연결은 느리니 주의하라
{: .no_toc }

문자열 연결이 많아질 수록 성능저하를 감내하기 어렵다. 문자열은 불변(아이템 17)이어서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 한다. 이로 인해 문자열 연결 연산자(`+`)로 문자열 n개를 잇는 시간은 n^2 에 비례하게 된다.<br>

문자열 연결연산자(`+`) 대신 `StringBuilder` 의 `append` 메서드를 사용하라. 문자 배열을 사용하거나, 문자열을 (연결하지 않고) 하나씩 처리하는 방법도 있다.<br>

자바 6 이후로 문자열 연결 성능을 다방면으로 개선했지만, `StringBuilder` 를 사용하는 것과 문자열 연결 연산자 `+` 를 사용하는 것의 성능 차이는 여전히 크다.<br>

품목을 100 개로 하고 `lineForItem` 이 길이 80 인 문자열을 반환하게 해서  작가의 컴퓨터에서 실행해보니 `StringBuilder` 버전의 문자열 연결이 63배 빨랐다고 한다.<br>

실제로 직접 테스트를 해본 결과를 문서에 따로 정리했는데, 해당 문서의 링크는 아래와 같다.<br>
[문자열 특징 정리 - 문자열결합하기 및 성능 측정](https://github.com/gosgjung/0-java-summary/blob/main/%EB%AC%B8%EC%9E%90%EC%97%B4/%EB%AC%B8%EC%9E%90%EC%97%B4-%ED%8A%B9%EC%A7%95%EC%A0%95%EB%A6%AC--%EB%AC%B8%EC%9E%90%EC%97%B4-%EA%B2%B0%ED%95%A9%ED%95%98%EA%B8%B0-%EB%B0%8F-%EC%84%B1%EB%8A%A5%EC%B8%A1%EC%A0%95.md)<br>
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
원칙은 간단하다. 성능에 신경써야 한다면 많은 문자열을 연결할 대는 문자열 연결자(`+`)를 피하자. 대신 `StringBuilder` 의 `append` 메서드를 사용하라. 문자 배열을 사용하거나, 문자열을 (연결하지 않고) 하나씩 처리하는 방법도 있다.<br>
<br>

### 문자열 연결 연산자(+)의 단점
{: .fs-6 .fw-700 }
문자열 연결 연산자(`+`) 는 여러 문자열을 하나로 합쳐줄 수 있는 편리한 수단이다. 한줄 또는 작고 크기가 고정된 문자열 표현을 만들때라면 크게 문제가 되지 않는다.<br>

하지만, 문자열 연결이 많아질 수록 성능저하를 감내하기 어렵다. 문자열은 불변(아이템 17)이어서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 한다. 이로 인해 문자열 연결 연산자(`+`)로 문자열 n개를 잇는 시간은 n^2 에 비례하게 된다.<br>

e.g. 아래는 문자열 연결을 사용하는 예다.
```java
public String statement(){
    String result = "";
    for(int i=0; i<numItems(); i++){
        result += lineForItem(i); // 문자열 연결
    }
    return result;
}
```

<br>

위 코드에서 `numItems()` 로 가져오는 아이템의 수가 많다면, 성능 저하를 유발할 수 있다. 따라서 `String` 대신 `StringBuilder` 를 사용해야 한다.<br>
<br>

### StringBuilder 를 사용해 개선하기
{: .fs-6 .fw-700 }

```java
public String statement2(){
    StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
    for(int i=0; i<numItems(); i++){
        b.append(lineForItem(i));
    }
    return b.toString();
}
```
