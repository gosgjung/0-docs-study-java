---
layout: default
title: Item 54. null 이 아닌 빈 컬렉션이나 배열을 반환하라 
nav_order: 6
has_children: false
parent:  Ch08. 메서드
grand_parent: Effective Java
permalink: /docs/effective-java/chapter08/item54
layout: default
---



# Item 54. null 이 아닌 빈 컬렉션이나 배열을 반환하라 
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



### 핵심정리
{: .fs-6 .fw-700 }

- null 이 아닌, 빈 배열이나 컬렉션을 반환하라. null 을 반환하는 API는 사용하기 어렵고 오류처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.

<br>
<br>

### return 값으로 null 참조를 리턴하는 예
{: .fs-6 .fw-700 }
```java
private final List<Cheese> cheeseInStock = ... ;

public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(chessInStock);
}
```
<br>

자세히 보면 아래의 코드가 수상하다.<br>

- `return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheeseInStock);` 

<br>

리스트가 비어있으면 `null` 을 반환한다. 상식적으로도 리스트의 반환값으로 `null` 을 반환하는 것은 옳지 않은 방식이다. 재고가 없을때 `null` 을 반환하겠다는 코드인데, 이 `getCheeses()` 메서드를 호출하는 곳에서는 아래와 같이 부가적으로 `null` 처리를 할 수밖에 없게 된다.<br>
<br>

```java
List<Cheese> cheeses = shop.getCheeses();
if(cheeses != null && cheeses.contains(Cheese.STLTON))
    System.out.println("좋았어 바로 그거야");
```
<br>
이런 코드는 null 방어 로직을 다른 코드들이 작성하게끔 만드는 코드라 좋지 않은 코드다. 위 코드는 아래의 두가지 코드들로 고치는 것이 낫다.<br>
<br>


### 1\) 빈 컬렉션을 반환
{: .fs-6 .fw-700 }

```java
public List<Cheese> getCheeses(){
    return new ArrayList<>(cheesesInStock);
}
```

가능성이 그리 높은 것은 아니지만, 위의 코드는 빈 컬렉션 할당을 하면서 성능을 떨어뜨리게 할 수도 있다.<br>

이 경우에 대한 해법은 매번 빈 '불변' 컬렉션을 반환하는 것이다.<br>

아래의 `2) 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 했다.` 가 이에 해당한다.<br>

### 2 ) 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 했다.
{: .fs-6 .fw-700 }

```java
public List<Cheese> getCheeses(){
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```

빈 `불변` 컬렉션을 반환하는 것은 `Collections` 클래스 내의 아래의 메서드들을 사용하는 것으로 가능하다.

- `Collections.emptyList()`
- `Collections.emptySet()`
- `Collections.emptyMap()` 

<br>

> 이 역시 최적화에 해당하기에, 꼭 필요할 때에만 사용하는 것이 낫다, 최적화가 필요하다고 판단되면, 수정 전과 수정 후의 성능을측정해 실제로 성능이 개선되는지 꼭 확인해야 한다.<br>
<br>

### JVM 최적화 관점에서 빈 컬렉션보다는 null 을 반환하는게 나은지?
{: .fs-6 .fw-700 }
빈 컬렉션, 빈 컨테이너를 할당하는데에 드는 비용도 생각해서 오히려 null 을 반환하기에 낫다는 의견도 있을 수 있다. 하지만 아래의 두가지 이유로 인해 틀린 주장이다.<br>

- 첫 번째, 성능 분석결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능차이는 신경쓸만한 수준이 못된다.
  - 아이템 67 - 최적화는 신중히 하라
- 두 번째, 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 
  - Collections.emptyList();
  - new ArrayList<>()

개인적인 의견이지만, 컬렉션 반환시 `null`값을 리턴하게 되면, null 값에 대한 방어로직을 처리하는 코드들로 인해 라인수가 늘어나게 되면 그 메서드를 호출하는 다른 메서드 측에서 동시성 처리수준이 조금 떨어지게 되고, 모호한 값으로 인해 예상치 못한 에러가 발생할 수 있다.<br>
<br>


### 배열을 사용할 경우
{: .fs-6 .fw-700 }
절대 `null` 을 반환하지 말고 길이가 `0` 인 배열을 반환하는 것이 좋다.<br>

예를 들면 아래와 같은 코드를 사용해 길이가 0을 반환할 수 있다.<br>

```java
public Cheese[] getCheeses(){
    return CheesesInStock.toArray(new Cheese[0]);
}
```

<br>

이 방식이 성능을 떨어뜨릴 수 있다고 판단되면 아래와 같이 길이가 0인 배열을 미리 선언해두고 매번 그 배열을 반환하면 된다. 예를 들면 아래의 코드와 같은 방식이다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses(){
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

<br>



