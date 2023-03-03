---
layout: default
title: Item 58. 전통적인 for문보다는 for-each 문을 사용하라 
nav_order: 2
has_children: false
parent:  Ch09. 일반적인 프로그래밍 원칙
grand_parent: Effective Java
permalink: /docs/effective-java/chapter09/item58
layout: default
---



# Item 58. 전통적인 for문보다는 for-each 문을 사용하라 
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
전통적인 for 문과 비교했을 때 `for-each` 문은 명료하고, 유연하고 버그를 예방한다. 성능 저하도 없다. 가능한 모든 곳에서 for 문이 아닌 `for-each` 문을 사용하자.
<br>

### for-each 란?
{: .fs-6 .fw-700 }
`for-each` 문은 아래와 같은 형태로 반복문을 작성한 것을 `for-each`문이라고 부른다.

```java
for(String tikcer : tickerList){
	System.out.println("ticker => " + ticker);
}
```

여기서 `:` 은 '안의(in)' 이라고 읽으면 된다. 이런 방식으로 위의 구문을 읽으면 ticker in tickerList가 된다.<br>

`for-each` 구문의 정식 이름은 '향상된 for 문(enhanced for statement)' 이다.<br>

`for-each` 구문을 사용한다고 해서 반복대상이 컬랙션이든, 배열이든 전통적인 for문과 반복자를 이용해 순회하는 것에 비해 속도 상에서 유리한 것은 아니다. 하지만, 과도한 인덱스 접근을 통해 나타나는 가독성 이슈를 해결해준다는 점에서 `for-each` 문은 실용성과 가독성 면에서 장점을 가지고 있다.<br>
<br>

### Iterable 인터페이스
{: .fs-6 .fw-700 }
이 `for-each` 문은 컬렉션, 배열에 대해 사용할 수 있다. 그리고 사용자가 직접 작성한 타입(클래스)이 원소들의 묶음을 표현해야 한다면, `Iterable`을 구현하는 쪽으로 고민해보는 것도 좋다. `Iterable` 인터페이스를 구현한 클래스는 무엇이든 순회할 수 있다.<br>

`Iterable` 인터페이스는 아래와 같이 정의되어 있다. `Iterable` 인터페이스는 메서드가 하나 뿐이다.<br>

```java
public interface Iterable<E>{
    // 이 객체의 원소를 순회하는 반복자를 반환한다.
    Iterator<E> iterator();
}
```

<br>

### `for-each` 구문을 사용할 수 없는 경우
{: .fs-6 .fw-700 }

`for-each` 문을 사용하지 못하는 경우가 있는데 대표적인 경우들을 정리해보면 아래와 같다.

파괴적인 필터링 (`destructive filtering` )

- 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
- 자바8 부터는 Collection 의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 것을 피할 수 있다.

변형(`transforming` )

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 할 경우 리스트의 반복자, 배열의 인덱스를 사용하는 것이 더 낫다.

병렬반복(`parallel iteration`)

- 여러 컬렉션을 병렬로 순회해야 할 경우 각각의 반복자, 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

<br>

### `for-each` 구문의 장점
{: .fs-6 .fw-700 }

`for-each` 문의 정식명칭은 향상된 for문(enhanced for statement)이다. `for-each` 구문을 사용하면 반복자와 인덱스 변수를 사용하는 것으로 인해 의도치 않은 실수를 하게 되는 것을 방지할 수 있다. 코드가 깔끔해지고 오류가 날 일도 없다.<br>

컬렉션을 중첩해서 순회하는 경우 `for-each` 구문의 이점이 더욱 커진다.<br>

아래코드에서 버그를 찾아보자.<br>
```java
public class IteratorTest {

    enum Suit {CLUB, DIAMOND, HEART, SPACE}
    enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    class Card{
        private Suit suit;
        private Rank rank;
        public Card(Suit suit, Rank rank){
            this.suit = suit;
            this.rank = rank;
        }
    }

    @Test
    public void 테스트1_suits_를_내부_for_문에서까지_순회하는_것으로_인해_에러가_발생한다(){
        List<Card> deck = new ArrayList<>();

        for(Iterator<Suit> i = suits.iterator(); i.hasNext();){
            for(Iterator<Rank> j = ranks.iterator(); j.hasNext();){
                Suit suit = i.next();
                Rank rank = j.next();
                System.out.println("suit = " + suit + ", rank = " + rank);
                deck.add(new Card(suit, rank));
            }
        }
    }
}
```
<br>

위 구문은 아래와 같은 에러를 낸다.

```plain
suit = CLUB, rank = ACE
suit = DIAMOND, rank = DEUCE
suit = HEART, rank = THREE
suit = SPACE, rank = FOUR

java.util.NoSuchElementException
	at java.base/java.util.Arrays$ArrayItr.next(Arrays.java:4245)
	at io.study.lang.javastudy2022ty1.effectivejava_temp.item58.IteratorTest.테스트1_suits_를_내부_for_문에서까지_순회하는_것으로_인해_에러가_발생한다(IteratorTest.java:30)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
// ...
```

<br>

이유는 `i.next()` 를 바깥 for 문에서 순회하게 두었어야 하는데, 내부 for 문 안에서 순회하게 되어서 `suits` 가 가진 요소 수를 넘어서서 순회를 하기 때문이다. 이에 대한 결과로 `NoSuchElementException` 이 발생하게 된다.<br>

<br>

이 경우 아래와 같이 구문을 수정해주면 훨씬 가독성이 있고, 인덱스 변수로 사용한 `i` , `j` 가 어디있는지 눈에 힘을 주어서 찾지 않아도 된다.<br>

```java
public class IteratorTest {

    enum Suit {CLUB, DIAMOND, HEART, SPACE}
    enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    class Card{
        private Suit suit;
        private Rank rank;
        public Card(Suit suit, Rank rank){
            this.suit = suit;
            this.rank = rank;
        }
    }

	// ... 

    @Test
    public void 테스트2_suits_순회를_for_each_구문으로_개선해보기(){
        List<Card> deck = new ArrayList<>();

        for(Suit suit : suits){
            for(Rank rank : ranks){
                System.out.println("suit = " + suit + ", rank = " + rank);
                deck.add(new Card(suit, rank));
            }
        }
    }
}
```