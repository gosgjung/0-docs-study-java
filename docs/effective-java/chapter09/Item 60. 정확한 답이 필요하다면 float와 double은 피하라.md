---
layout: default
title: Item 60. 정확한 답이 필요하다면 float와 double은 피하라
nav_order: 4
has_children: false
parent:  Ch09. 일반적인 프로그래밍 원칙
grand_parent: Effective Java
permalink: /docs/effective-java/chapter09/item60
layout: default
---



# Item 60. 정확한 답이 필요하다면 float와 double은 피하라
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

### 요약
{: .fs-6 .fw-700 }
`float` 과 `double` 은 공학용으로 설계되었다.  넓은 범위의 수를 `근사치` 로 계산하도록 설계되었다. 그리고 `float` 과 `double` 은 이진 부동소숫점 연산을 수행한다. `근사치` 로 계산이 수행되므로, 정확한 결과가 필요할 때는 사용하면 안된다. **`float`, `double` 은 특히 금융관련 계산과는 맞지 않는다.**<br>

이 내용은 자바 퍼즐러 - 2번째 퍼즐, 변화를 위한 시간 에서도 다루는 내용이다. 자바퍼즐러의 내용들은 현재 다른 문서에서 정리하고 있어서 해당 문서의 정리가 끝나면 여기에도 링크를 추가해놓을 예정이다.<br>

### 핵심정리
{: .fs-6 .fw-700 }
정확한 답이 필요한 계산에는 `float` 나 `double` 을 피하라. 소수점 추적은 시스템에 맡기고, 코딩 시의 불편함이나 성능저하를 신경쓰지 않겠다면 `BigDecimal` 을 사용하라. `BigDecimal` 이 제공하는 여덟가지 반올림 모드를 이용해 반올림을 완벽히 제어할 수 있다. 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서 아주 편리한 기능이다.<br>

반면 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 `int` 나 `long` 을 사용하라. 숫자를 아홉자리 십진수로 표현할 수 있다면 `int` 를 사용하고 열여덟 자리 십진수로 표현할 수 있다면 `long` 을 사용하라. 열어덟자리를 넘어가면 `BigDecimal` 을 사용해야 한다.<br>
<br>

### 근사치로 표현하는 float 과 double 자료형
{: .fs-6 .fw-700 }
`float` 과 `double` 은 공학용으로 설계되었다.  넓은 범위의 수를 `근사치` 로 계산하도록 설계되었다. 그리고 `float` 과 `double` 은 이진 부동소숫점 연산을 수행한다. `근사치` 로 계산이 수행되므로, 정확한 결과가 필요할 때는 사용하면 안된다. **`float`, `double` 은 특히 금융관련 계산과는 맞지 않는다.**<br>
<br>

### float, double 을 사용하는 예제
{: .fs-6 .fw-700 }
**예제 1) 1.03 - 0.42**<br>

1.03 - 0.42 를 직접 해보면 0.61 이다.

```java
public class FloatDoubleDontUseTest {

    @Test
    public void 간단한_뺄셈연산에_사용해보기_1(){
        System.out.println(1.03 - 0.42);
    }
}
```

<br>

위 코드의 출력결과는 아래와 같다.<br>

0.61 이 나와야 하지만 소숫점 맨 끝자리에 0.0000000000000001 이 더해졌다.

```plain
0.6100000000000001
```

<br>

**예제 2) 1.00 - 9 * 0.10**<br>

0.1 이 출력되어야 정상이다.

```java
public class FloatDoubleDontUseTest {

    @Test
    public void 간단한_뺄셈연산에_사용해보기_2(){
        System.out.println(1.00 - 9 * 0.10);
    }
}
```

<br>

위 코드의 출력결과는 아래와 같다.<br>

0.1 이 나와야 하지만 근사치 값인 `0.09999999999999998` 가 나왔다.

```plain
0.09999999999999998
```

<br>

**예제 3) 잔돈계산 프로그램**<br>

> 가지고 있는 돈이 1달러가 있고, 편의점의 선반에 있는 사탕은 10센트, 20센트, 30센트, ... ,1달러 의 사탕들이 있다고 해보자. 이때 10센트 부터 1달러까지 차례대로 가격이 낮은 순으로 1달러로 살수 있는 사탕들을 계속해서 살때 몇개나 사탕을 살수 있는지, 잔돈은 얼마나 남는지 알아보는 **'어설픈'** 프로그램이다.<br>

<br>

달러를 냈는데 몇센트 정도가 남았을때 센트를 달러로 표시하면서 소숫점으로 표시하게 된다. 이때 `float` , `double` 을 사용하는 것으로 인한 오차가 발생한다.

```java
package io.study.lang.javastudy2022ty1.effectivejava_temp.item60;

import org.junit.jupiter.api.Test;

public class FloatDoubleDontUseTest {

    @Test
    public void 간단한_뺄셈연산에_사용해보기_3(){
        double funds = 1.00;
        int itemBought = 0;
        for(double price = 0.10; funds >= price; price += 0.10){
            funds -= price;
            itemBought++;
        }

        System.out.println(itemBought + "개 구입");
        System.out.println("잔돈(달러) : " + funds);
    }
}
```

<br>

출력결과

```plain
3개 구입
잔돈(달러) : 0.3999999999999999
```

잘못된 결과다. 이 문제를 바로 잡으려면 `BigDecimal` 을 이용해 계산하거나, 달러 단위로 계산하는 것을 센트 단위로 기준을 바꿔서 실수자료형으로 표시할 것을 정수 타입(int, long) 으로 계산하게끔 처리하면 된다.

<br>

### 1\) BigDecimal 을 사용해 개선
{: .fs-6 .fw-700 }

아래는 `BigDecimal` 을 이용해 계산하는 경우의 예제다.
```java
public class FloatDoubleDontUseTest {
	
    @Test
    public void 간단한_뺄셈연산에_사용해보기_4(){
        final BigDecimal TEN_CENTS = new BigDecimal(".10");

        int itemBought = 0;
        BigDecimal funds = new BigDecimal("1.00");
        for(BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)){
            funds = funds.subtract(price);
            itemBought++;
        }

        System.out.println(itemBought + " 개 구입");
        System.out.println("잔돈(달러) : " + funds);
    }
}
```
위의 프로그램을 실행시킨 후의 출력결과다.<br>
잔돈이 0달러가 되어야 정상이다.<br>

**출력결과**<br>
```plain
4 개 구입
잔돈(달러) : 0.00
```
<br>

### 2\) 단위를 작은 단위로 축소시켜서 정수형으로 변환 후 계산하는 방식
{: .fs-6 .fw-700 }


두번째 해결방식은 단위를 조금 변경시켜서, 가급적 소숫점 연산을 회피하고 정수자료형 연산을 하도록 프로그램을 변경하는 것이다. 

아래 예제에서는 `BigDecimal` 을 사용하지 않고, `int` , `long` 을 사용해서 거스름돈을 계산했다. 1 달러는 100센트인데, 1 달러에서 사탕 가격으로 0.1 달러, 0.2달러 0.3달러를 차례로 빼는 대신, 100센트에서 10센트, 20센트, 30센트... 를 차례로 뺀다면, 불필요하게 `BigDecimal`을 사용하지 않아도 된다.<br>

아래는 그 예제다.<br>

```java
package io.study.lang.javastudy2022ty1.effectivejava_temp.item60;

import org.junit.jupiter.api.Test;

import java.math.BigDecimal;

public class FloatDoubleDontUseTest {
    @Test
    public void 간단한_뺄셈연산에_사용해보기_5(){
        int itemBought = 0;
        int funds = 100;
        for(int price = 10; funds >= price; price += 10){
            funds -= price;
            itemBought++;
        }
        System.out.println(itemBought + "개 구입");
        System.out.println("잔돈(센트) : " + funds);
    }
}
```
<br>

<br>

출력결과<br>

```plain
4개 구입
잔돈(센트) : 0
```


