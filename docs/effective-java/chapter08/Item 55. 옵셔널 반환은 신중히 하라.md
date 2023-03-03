---
layout: default
title: Item 55. 옵셔널 반환은 신중히 하라 
nav_order: 7
has_children: false
parent:  Ch08. 메서드
grand_parent: Effective Java
permalink: /docs/effective-java/chapter08/item54
layout: default
---



# Item 55. 옵셔널 반환은 신중히 하라
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
값을 반환하지 못할 가능성이 있고, 호출할 때 마다 반환값이 있을 가능성을 염두에 둬야 하는 메서드라면 옵셔널을 반환해야 할 상황일 수 있다. 하지만 옵셔널 반환에는 성능저하가 뒤따르니, 성능에 민감한 메서드라면 `null` 을 반환하거나 예외를 던지는 편이 나을 수 있다. 그리고 **옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.**
<br>
<br>

### null vs Optional (Java 8 이전 vs Java 8 이후)
{: .fs-6 .fw-700 }
Java 8 이전

- 예외 던진다.
- `null` 을 리턴한다.
  - 별도의 null 처리 코드가 기하급수적으로 늘어난다는 것이 단점

Java 8 이후

- `Optional<T>` 라는 선택지
- 옵셔널은 원소를 최대 1 개 가질수 있는 `불변` 컬렉션 (장점이다.)
- 예외를 던지는 메서드보다 유연하다. 사용하기 쉽다.
<br>
<br>

### 예제) Optional 을 통해 로직을 단순하게 바꿔보기
{: .fs-6 .fw-700 }

#### before
{: .fs-5 .fw-700 }

```java
public List<Cheese> getCheeses(){
    return new ArrayList<>(cheesesInStock);
}
```
<br>

#### after
{: .fs-5 .fw-700 }

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> e){
  if(c.isEmpty())
    return Optional.empty();
  
  E result = null;
  
  for(E e : c)
    if(result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  
  return Optional.of(result);
}
```
<br>

### Optional.empty(), Optional.of(), Optional.ofNullable()
{: .fs-6 .fw-700 }

옵셔널을 반환하는 메서드에서 `null` 값을 명시적으로 던지지 말자.
<br>

- Optional.empty()
  - 빈값을 처리할 때 사용. 비어 있는 옵셔널
- Optional.of()
  - 값이 들어있는 옵셔널. 널 값은 허용하지 않는 메서드
  - 만약, Optional.of()에 null 값을 넘기면, `NullPointerException` 을 던진다.
- Optional.ofNullable()
  - null 값도 허용하는 옵셔널을 만들 때 사용
<br>

### 스트림의 종단연산 들 중 상당수는 옵셔널을 반환한다
{: .fs-6 .fw-700 }

스트림의 종단연산 들 중 상당수는 옵셔널을 반환한다.
```java
public static <E extends Comparable<E>> Optional<E> max (Comparable<E> c){
  return c.stream().max(Comparator.naturalOrder());
}
```
<br>

### 옵셔널을 반환해야 하는 경우에 대한 선택기준
{: .fs-6 .fw-700 }

- 옵셔널은 검사예외(CheckedException)과 취지가 비슷하다.
- `Optional.empty` 일 수 있음을 명확히 한다는 것이 장점이다.
<br>

어떤 함수에서 비검사예외를 throw 하거나, null을 리턴하는 경우
- 처리가 쉽지 않고 예상치 못한 결과를 낼 수 있다.
반면, 검사예외를 throw 하면
- 클라이언트 측에서 대응 코드를 작성하게 된다.
<br>
책에서는 이렇게 비검사예외와 검사예외의 차이를 비교하면서 Optional 이 검사예외처럼 클라이언트 측에서 대응코드를 작성하게 한다는 점에서 Optional을 사용하는 것의 장점을 언급하고 있다. <br>

참고로, 검사 예외는 굉장히 신중하게 throw 해야 한다. 하지만 Optional 은 검사예외 만큼 신중하게 사용하지 않아도 된다. 즉, 검사예외보다는 편리하다.(위의 내용 까지는 책에서 언급하는 비유였는데, 뺄지 말지 고민을 하다가, 빼지를 못하겠어서 정리는 해두었다. 개인적으로는 굳이 검사 예외로까지 비교할 필요는 없다는 생각을 했었다.)<br>
<br>

값이 없을 수 있음을 `NullPointerException` 없이 명확하게 표현해준다는 점을 떠올려보면 장점이 꽤 명확하다.<br>
예를 들면, 값이 비어있을 수도 있는 메서드를 호출해서 반환값을 처리할 때 아래와 같은 경우 들로 처리하는 것이 가능해진다.<br>
<br>
e.g. 1\)
```java
public void orderSomething(){
  Optional<String> menu = getMenu();
  
  menu.ifPresent(m -> {
    orderService.order(m);
  });
}
```
<br>

또는 아래와 같이 처리하는 것도 가능하다.<br>
<br>

e.g. 2\)
```java
public void orderSomething(){
  Optional<String> menu = getMenu();
  if(menu.isEmpty()) return;
  
  orderService.order(m);
}
```
<br>
<br>

### 여러가지 옵셔널 함수 예제
{: .fs-6 .fw-700 }

#### orElse, orElseGet, orElseThrow, get
{: .fs-5 .fw-700 }

##### orElse
{: .fs-4 .fw-700 }

```java
@Test
public void TEST_OR_ELSE(){
    String word1 = "안뇽하세요";
    String optWord1 = Optional.ofNullable(word1).orElse("인사말을 입력해주세요!!!");
    System.out.println(optWord1);

    String word2 = null;
    String optWord2 = Optional.ofNullable(word2).orElse("인사말을 입력해주세용!!!");
    System.out.println(optWord2);
}
```

**출력결과**<br>
```plain
안뇽하세요
인사말을 입력해주세용!!!
```
<br>

##### orElseGet
{: .fs-4 .fw-700 }
orElseGet에 들어가는 인자값은 `Supplier<T>` 다. 

```java
@Test
public void TEST_OR_ELSE_GET(){
    String word1 = "안뇽하세요요용!!!";
    String optWord1 = Optional.ofNullable(word1).orElseGet(() -> "반갑습니다!!!");
    System.out.println(optWord1);

    String word2 = null;
    String optWord2 = Optional.ofNullable(word2).orElseGet(() -> "반갑습니당 !!!");
    System.out.println(optWord2);
}
```
<br>

출력결과

```plain
안뇽하세요요용!!!
반갑습니당 !!!
```

<br>

##### orElseThrow
{: .fs-4 .fw-700 }
orElseThrow 는 Optional 로 받은 값이 비어있는 값일 때 Exception 을 던진다.

```java
@Test
public void TEST_OR_ELSE_THROW(){
    String word1 = "반갑습니당";
    String optWord1 = Optional.ofNullable(word1).orElseThrow(RuntimeException::new);
    System.out.println(optWord1);

    String word2 = null;
    assertThatThrownBy(()->{
        Optional.ofNullable(word2).orElseThrow(RuntimeException::new);
    }).isInstanceOf(RuntimeException.class);

    assertThatThrownBy(()->{
        Object word3 = Optional.empty().orElseThrow(RuntimeException::new);
        System.out.println(word3);
    }).isInstanceOf(RuntimeException.class);

    assertThatThrownBy(()->{
        Object o = Optional.empty().orElseThrow(() -> new RuntimeException("비어있는 값은 처리할 수 없어요!!!"));
    }).isInstanceOf(RuntimeException.class)
        .hasMessageContaining("처리할 수 없어요!!!");
}
```
<br>

**출력결과**<br>

```plain
반갑습니당
```

<br>

##### Optional.get
{: .fs-4 .fw-700 }

`Optional` 에 값이 있다는 확신이 든다면, 곧바로 `Optional.get()` 메서드로 값을 꺼내서 사용하는 것도 나쁜 방법은 아니다. <br>

```java
@Test
public void TEST_IF_OPTIONAL_HAS_VALUE_YOU_CAN_USE_OPTIONAL_GET_RIGHT_NOW(){
    Optional<String> notEmpty = Optional.of("TEST");

    if(notEmpty.isEmpty()) return; // 옵셔널에 값이 없으면 리턴해버린다.

    String s = notEmpty.get();  // 이미 값이 있는 경우에만 거치도록 필터링되어 있는 상태다.
    System.out.println(s);
}
```

<br>

출력결과

```plain
TEST
```

<br>

#### filter, map, flatMap, ifPresent
{: .fs-5 .fw-700 }

##### Optional.filter
{: .fs-4 .fw-700 }

```java
@Test
public void OPTIONAL_FILTER(){
    Optional<Integer> odd = Optional.ofNullable(3)
        .filter(num -> num % 2 == 0);

    System.out.println(odd);
    assertThat(odd).isEmpty();

    Optional<Integer> even = Optional.ofNullable(2)
        .filter(num -> num % 2 == 0);
    System.out.println(even);
    assertThat(even).isNotEmpty();
}
```

출력결과
```plain
Optional.empty
Optional[2]
```

##### Optional.map
{: .fs-4 .fw-700 }

```java
@Test
public void OPTIONAL_MAP(){
    String hello1 = "Hello";
    Optional<Integer> optNumber1 = Optional.ofNullable(hello1)
        .map(str -> str.length());
    System.out.println(optNumber1);
    assertThat(optNumber1).isNotEmpty();

    String hello2 = null;
    Optional<Integer> optNumber2 = Optional.ofNullable(hello2)
        .map(str -> str.length());
    System.out.println(optNumber2);
    assertThat(optNumber2).isEmpty();
}
```
<br>

출력결과
```plain
Optional[5]
Optional.empty
```
<br>

#### Optional.flatMap
{: .fs-5 .fw-700 }
> 참고한 예제 : [Java 8 Optional flatMap() Method Example](https://java8example.blogspot.com/2019/08/optional-flatmap.html) <br>

Optional 이 아니더라도 flatMap 은 보통 Stream 에서는 Stream 안에서 Stream을 만들어내서 서로 다른 두 Stream을 합칠때 사용하는 편이다. flatMap 에 대해 감이 잘 안잡힌다면 [은혜로운 flatMap](https://github.com/soon-good/modern-java-in-action/blob/develop/%EC%9E%90%EB%B0%94%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%A5%BC-%EC%9C%84%ED%95%9C-97%EA%B0%80%EC%A7%80-%EC%A0%9C%EC%95%88/47.%EC%9D%80%ED%98%9C%EB%A1%9C%EC%9A%B4-flatMap-%EB%8B%A4%EB%8B%88%EC%97%98-%EC%9D%B4%EB%85%B8%ED%98%B8%EC%82%AC.md) 을 참고하자.<br>


**예제 1)**<br>

`hello2` 는 옵셔널 하나를 감싸는 중첩 옵셔널이다.<br>

즉 `hello2` 는 옵셔널 두개가 중첩되어 있다.<br>

그런데 아래 구문을 실행하고 나면 중첩되어 있던 `Optional` 이  `Optional` 하나로 변환되는 것을 확인할 수 있다.

```java
@Test
public void TEST_OPTIONAL_FLATMAP_CASE1(){
    Optional<String> hello1 = Optional.of("Hello1");
    Optional<Optional<String>> hello2 = Optional.of(hello1);

    Optional<String> result = hello2.flatMap(optHello -> optHello.map(String::toUpperCase));
    System.out.println(result);

    assertThat(result).isNotEmpty();
}
```

<br>

출력결과<br>

```plain
Optional[HELLO1]
```

<br>


예제 2)<br>
- `--1)`
  - `Optional<Optional<Optional<Double>>> ` 타입의 `calorie3` 를 그대로 출력하고 있다.
- `--2)` 
  - `Optional<Optional<Optional<Double>>>` 을 `Optional<Double>` 로 변환해서 출력하고 있다.
- `--3)`
  - `Optional<<Optional<Optional<Double>>>`  을 `Optional<Double>` 로 변환해서 출력하고 있다.

`--2` 와 `--3` 의 차이점은 중간에 `Optional.isPresent()` 로 값이 존재하는지 처리를 할지 `Optional.map` 메서드로 null 이 없으면 없는대로 리턴하게끔 처리를 할지 결정한다는 점이 다르다. 두 코드 모두 의도하는 결과값은 같다. 두 방식중에 마음에 드는 것을 사용하면 될 것 같다.<br>

```java
@Test
public void TEST_OPTIONAL_FLATMAP_CASE2(){
    Optional<Double> calorie = Optional.of(1.1D);
    Optional<Optional<Double>> calorie2 = Optional.of(calorie);
    Optional<Optional<Optional<Double>>> calorie3 = Optional.of(calorie2);

    System.out.println(calorie3); 		// -- 1)

    Optional<Double> calorieData1 = calorie3.flatMap(optOptCalorie -> optOptCalorie.flatMap(
        optCalorie -> {
            if (optCalorie.isPresent()) {
                return Optional.of(optCalorie.get() * 1000);
            }
            return Optional.empty();
        })
                                                    );

    System.out.println(calorieData1); 	// -- 2)

    Optional<Double> calorieData2 = calorie3.flatMap(optOptCalorie -> optOptCalorie.flatMap(
        optCalorie -> {
            return optCalorie.map(aDouble -> aDouble * 1000);
        })
                                                    );

    System.out.println(calorieData2);	// -- 3)
}
```

출력결과<br>

```plain
Optional[Optional[Optional[1.1]]]
Optional[1100.0]
Optional[1100.0]
```

<br>

예제 3)<br>
```java
@Test
@DisplayName("null 값을 flatMap 으로 돌릴 경우의 예제")
public void TEST_OPTIONAL_FLATMAP_NULL_VALUE(){
    Optional<Double> emptyCalorie = Optional.ofNullable(null);
    Optional<Double> flatMappedEmptyCalorie = emptyCalorie.flatMap(calorie -> Optional.of(Double.MAX_VALUE));

    System.out.println(flatMappedEmptyCalorie);
}
```

출력결과<br>

```plain
Optional.empty
```

<br>



#### Optional.isPresent, Optional.ifPresent, Optional.map
{: .fs-5 .fw-700 }

##### Optional.isPresent
{: .fs-4 .fw-700 }

```java
@Test
public void OPTIONAL_IS_PRESENT(){
    String bookName = null;
    if(Optional.ofNullable(bookName).isPresent()){
        System.out.println(bookName.length());
    }
}
```

<br>

##### Optional.ifPresent
{: .fs-4 .fw-700 } 

```java
@Test
public void OPTIONAL_IF_RESENT_TEST(){
    String bookName = null;
    Optional.ofNullable(bookName).ifPresent(book -> System.out.println(book));
}
```

<br>

##### Optional.map
{: .fs-4 .fw-700 }

```java
@Test
public void OPTIONAL_MAP_TEST(){
    String bookName = null;
    Optional<Integer> len = Optional.ofNullable(bookName)
        .map(book -> book.length());

    len.ifPresent(l -> System.out.println(l));
}
```

<br>


