---
layout: default
title: Item 61. 박싱된 기본타입보다는 기본타입을 사용하라
nav_order: 5
has_children: false
parent:  Ch09. 일반적인 프로그래밍 원칙
grand_parent: Effective Java
permalink: /docs/effective-java/chapter09/item61
layout: default
---



# Item 61. 박싱된 기본타입보다는 기본타입을 사용하라
{: .no_toc }

[아이템 6](https://github.com/soon-good/study-effective-java-3rd/blob/develop/ITEM-6-%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C-%EA%B0%9D%EC%B2%B4-%EC%83%9D%EC%84%B1%EC%9D%84-%ED%94%BC%ED%95%98%EB%9D%BC.md) 에서 언급했듯 오토박싱과 오토언박싱 덕분에 두 타입을 크게 구분하지 않고 사용할 수는 있지만, 그렇다고 해도 차이가 사라지는 것은 아니다. 박싱된 기본타입과 기본타입 사이에는 분명 차이가 있기 때문에 어떤 타입을 사용하는지는 매우 중요하다.<br>

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

### == 연산 수행시 의도치 않은 동치성검사
{: .fs-6 .fw-700 }
**`==` 연산 수행시 , 의도치 않은 객체식별성 동치검사 & `NullPointerException`** <br>

박싱된 기본 타입을 사용할 때의 `==` 연산 수행시, 박싱된 기본 타입은 언박싱이 이루어지는데, 특정 경우에 따라 `NullPointerException` 이 일어나기도 하고, 어떤 때는 객체간의 연산이 수행된다. <br>

박싱된 기본 타입보다는 기본타입을 사용하는 것을 권장하고 있지만, 모든 경우에 기본타입을 항상 사용할 수 있지는 않다. 예를 들면 제너릭으로 구현된 메서드의 경우 박싱된 기본타입으로 연산을 수행되어야 하기 때문이다.<br>


### 핵심정리
{: .fs-6 .fw-700 }
기본타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하라. 기본 타입은 간단하고 빠르다. 박싱된 기본 타입을 써야 한다면 주의를 기울이자. 오토 박싱이 박싱된 기본 타입을 사용할 때의 번거로움을 줄여주지만, 그 위험까지 없애주지는 않는다.<br>

두 박싱된 기본 타입을 `==` 연산자로 비교한다면 식별성(identity) 비교가 이루어지는데, 이는 여러분이 원한게 아닐 가능성이 크다. 같은 연산에서 기본 타입과 박싱된 기본 타입을 혼용하면 언박싱이 이뤄지며, 언박싱과정에서 `NullPointerException` 을 던질 수 있다. 마지막으로, 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용을 나을 수 있다.<br>
<br>

### 기본 타입과 박싱된 기본 타입의 차이점
{: .fs-6 .fw-700 }

**첫번째, `식별성`** <br>

기본 타입은 값만 가지고 있다. 하지만 박싱된 기본 타입은 객체기반이기에, 식별성(`identity`)이라는 속성 역시도 가지고 있다. `Integer`, `Double` 등과 같은 박싱된 기본 타입을 사용할 경우, 동치 또는 `==` 연산 수행시 `==  ` 연산 대신 `equals` 연산을 수행하도록 별도의 처리를 해줘야 한다. <br>

즉, 같은 숫자값을 가지도록 인자값을 주고 `new` 연산자로 각각 생성한 박싱된 기본타입 변수들은 `==` 연산시 서로 다르게 판별된다.<br>

<br>

**두번째, `null 허용`** <br>

기본 타입의 변수에는 null 을 할당할 수 없고 초기화를 안하면 0으로 할당된다. 하지만, 박싱된 기본타입은 null 타입이 존재한다.<br>

<br>

**세번째, `메모리 사용면에서 효율성`**<br>

기본 타입이 박싱된 기본 타입보다 메모리 공간을 덜 사용하기에 메모리 사용 면에서 더 효율적이다.<br>

<br>


### e.g. Integer.compare(...)
{: .fs-6 .fw-700 }
아래의 예제는 에러를 낸다. 박싱된 기본 타입을 사용할 때 실수할 수도 있는 동치성 검사에 대한 예제다.

```java
Comparator<Integer> naturalOrder = 
    (i,j) -> (i<j) ? -1 : (i == j ? 0 : 1);
```

위의 비교연산은 컴파일에러나 런타임에러를 내지는 않지만, 버그가 있다. 아래의 코드를 실행하면 값에 대한 비교가 아닌 객체간의 비교를 수행하게 되기 때문이다.

```java
naturalOrder.compare(new Integer(42), new Integer(42));
```

이렇게 비교를 하면 첫 번째 `new Integer(42)` 와 두 번째 `new Integer(42)` 는 서로 다르다고 판별된다. 

우선 아래의 비교식은 문제가 되지 않는다. <br>아래의 비교 연산에서는 대소 비교 (`<`) 연산자가 사용되었기에 오토박싱된 상태로 연산을 수행하기 때문이다.

```java
(i<j)
```

하지만 아래의 식에서 문제가 된다.

```java
(i == j ? 0 : 1)
```

`i` 는 `new Integer(42)` 다. 그리고 `j` 는 `new Integer(42)` 다. 이때 동등 비교 `==` 수행시에 객체간의 비교를 하게 된다. `i` 와 `j` 의 객체 식별값은 서로 다르기에 `i == j` 의 연산 결과 값은 `false` 가 된다.<br>

<br>

**해결책**

실무에서는 이와 같은 기본타입에 대한 비교자(`Comparator`) 연산이 필요한 경우 지역변수 2개를 두어 각각 박싱된 `Integer` 매개변수의 값을 기본 타입 정수로 저장하고, 모든 비교를 이 기본 타입의 변수로 수행하면 된다. <br>

아래는 그 예제다.

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed;	// 오토박싱
    int j = jBoxed; // 오토박싱
    return i < j ? -1 : (i == j ? 0 : 1);
};
```

<br>

비교자를 직접 만들 때는 비교자 생성 메서드나, 기본 타입을 받는 정적 `compare` 메서드를 사용해야 한다.([아이템 14](https://github.com/soon-good/study-effective-java-3rd/blob/develop/ITEM-14-Comparable%EC%9D%84-%EA%B5%AC%ED%98%84%ED%95%A0%EC%A7%80-%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md))<br>

<br>

### `NullPointerException`
{: .fs-6 .fw-700 }

박싱된 기본 타입에 `null` 로 초기화 된 값과 함께 `==` 연산을 수행하면 `NullPointerException` 이 발생한다. 이유는 박싱된 기본 타입에 대해 `==` 연산을 수행할 때 박싱된 기본 타입은 언박싱을 수행한다. 이때 `null` 값에 대해서는 언박싱을 수행할 수 없기 때문이다.<br>

먼저 첫번째 예제를 보자.<br>

**ex) 초기화 되지 않은 값과 `==` 비교는 컴파일 에러다.**<br>

아래 예제는 컴파일러마다 다르겠지만, 현재 `JDK 16` 을 사용하고 있는 내 노트북 에서는 에러를 낸다.

```java
@Test
public void 초기화하지않을_경우는_컴파일에러_발생(){
    Integer i;

    if(i == 2022) {
        System.out.println("믿을 수 없군");
    }
}
```

이유는 초기화 되지 않은 변수 `i` 를 사용했기 때문이다. 박싱된 기본 타입의 경우 비교연산 수행시 언박싱이 되는데, 언박싱 수행시 초기화되지 않은 값을 비교하는 것이 불가능하기 때문이다.<br>

<br>

**ex) null 값과 `==` 비교를 수행하면 `NullPointerException` 을 발생시킨다.**<br>

```java
@Test
public void NullPointerException_예외_발생(){
    Integer i = null;

    if(i == 2022) {
        System.out.println("믿을 수 없군");
    }
}
```

출력결과

```plain
java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "i" is null
```

이렇게 `NullPointerException` 을 일으키는 이유는 이렇다.<br>

박싱된 기본 타입과 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다. 위 예제에서 박싱된 기본 타입 변수는 `i` 다. 기본타입 값은 숫자 리터럴 `2022` 다. 이 때 박싱된 기본 타입 변수를 언박싱해야 하는데, `null` 참조를 언박싱해야 하므로 `NullPointerException` 이 발생하게 되는 것이다.<br>

<br>

### ex) 수행 속도가 느린 예제
{: .fs-6 .fw-700 }

아래의 예제를 보자. 이 예제는 아이템 6에서 살펴봤었다. <br>

오류나 경고 없이 컴파일이 되지만, 박싱과 언박싱이 반복하여 일어나기에 체감될 정도로 성능이 느리다는 단점이 있다. 모두 수행되는 데에 내 컴퓨터에서는 5초 899 밀리세컨드가 소모됬다.

```java
@Test
public void 수행속도가_느린_예제(){
    Long sum = 0L;
    for(long i = 0; i <= Integer.MAX_VALUE; i++){
        sum += i;
    }
    System.out.println(sum);
}
```

<br>

