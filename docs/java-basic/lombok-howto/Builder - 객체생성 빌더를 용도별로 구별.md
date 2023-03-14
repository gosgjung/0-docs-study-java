---
layout: default
title: Builder - 객체생성 빌더를 용도별로 구별
nav_order: 3
has_children: false
parent: Lombok 제대로 써보자
grand_parent: Java Basic
permalink: /docs/java-basic/lombok-howto/lombok-builder-specify-builder-by-usage
---


# Builder - 객체생성 빌더를 용도별로 구별
{: .no_toc }
<br>
- 빌더 내에 경우에 따라 멤버필드를 서로 다르게 구성해야 하는 경우가 있다.
- 이런 경우를 대비해 빌더는 용도별로 구별해서 사용하자.
<br>

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

<br>

### 참고자료
{: .fs-6 .fw-700 }
<br>
<br>


### 쓰임에 따라 데이터의 바인딩이 달라질때
{: .fs-6 .fw-700 }

생성자는 이름을 부여해서 서로 다른 용도의 생성자로 분류할 수 없다. 이런 이유로 생성자를 오버로딩하거나, 계층적 생성자를 사용하기도 한다. 또는 정적 팩터리 메서드로 분류해두기도 한다.<br>

정적 팩터리 메서드로 분류한다고 하더라도 그 많은 인자들의 경우의 수를 감당할 수 없기에 정적 팩터리 메서드의 내부는 빌더 패턴을 통해 객체를 생성하게 되는 경우가 많을것 같다.<br>

오늘 예제는 고객이 어떤 주문을 했을 때 환불을 하는 경우에 대한 예제다.<br>

- 고객이 환불하려는 주문이 무통장입금으로 주문한 주문일 경우
- 고객이 환불하려는 주문이 신용카드로 주문한 주문일 경우

이렇게 두 가지 경우에 대해 빌더 구문을 작성해보고 테스트 코드를 작성하는 예제를 살펴보는 과정을 예제로 정리했다. 예제의 목적을 요약해보자면 'null 체크를 하는 경우의 수를 용도에 맞게 따로 분리해서 깔끔하게 빌더메서드로 분류' 하는 것이 되지 않을까 싶다.<br>

실무에서도 얼마든지 부딪힐 수 있는 경우의 수 체크문제가 될 수 있어서 정리를 시작했다.
<br>
<br>

#### Refund 객체 생성
{: .fs-5 .fw-700 }

Refund 객체는 아래와 같이 무통장입금 주문인지, 신용카드입금 주문인지에 따라 서로 다른 builder 메서드에서 객체 생성을 각각 다르게 하고 유효성 체크 역시 빌더 메서드마다 서로 다르게 수행되게끔 했다.
<br>

**Refund.java**<br>
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "refund")
public class Refund {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Account account;

    @Embedded
    private CreditCard creditCard;

    @OneToOne
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;

    // 무통장입금으로 환불
    @Builder(builderClassName = "ByAccountBuilder", builderMethodName = "byAccountBuilder")
    public Refund(Account account, Order order) {
        Assert.notNull(account, "계좌번호는 비어있을 수 없습니다.");
        Assert.notNull(order, "주문은 비어있을 수 없습니다.");
        this.account = account;
        this.order = order;
    }

    // 신용카드로 환불
    @Builder(builderClassName = "ByCreditCardBuilder", builderMethodName = "byCreditCardBuilder")
    public Refund(CreditCard creditCard, Order order) {
        Assert.notNull(creditCard, "신용카드는 비어있을 수 없습니다.");
        Assert.notNull(order, "주문은 비어있을 수 없습니다.");
        this.creditCard = creditCard;
        this.order = order;
    }
}
```

무통장 입금으로 환불할지, 신용카드로 환불할지 여부에 따라 빌더 메서드를 다르게 정의했다.<br>

이제 테스트 코드를 작성해보자.<br>
<br>

#### 무통장입금 주문 건 환불객체 생성 테스트
{: .fs-5 .fw-700 }

먼저 무통장 입금에 대한 환불 객체 생성 테스트다.

```java
@Test
public void 환불테스트__무통장입금(){
    Refund refund = Refund.byAccountBuilder()
            .account(account)
            .order(order)
            .build();

    assertThat(refund.getAccount()).isEqualTo(account);
    assertThat(refund.getOrder()).isEqualTo(order);
}

@Test
public void 환불테스트__환불계좌가_null_일경우_IllegalArgumentException이_발생해야_한다(){
    Assertions.assertThatThrownBy(()->{
        Refund.byAccountBuilder()
                .account(null)
                .order(order)
                .build();
    }).isInstanceOf(IllegalArgumentException.class);
}
```

무통장입금의 경우 환불계좌 정보가 비어있으면 안되는데, 이와 관련해서 올바르게 Exception 을 던지고 있는 것을 확인 가능하다.<br>
<br>

#### 신용카드 주문 건 환불객체 생성 테스트
{: .fs-5 .fw-700 }

이번에는 신용카드 주문에 대한 환불 객체 생성 테스트다.

```java
@Test
public void 환불테스트__신용카드(){
    Refund refund = Refund.byCreditCardBuilder()
            .creditCard(creditCard)
            .order(order)
            .build();

    assertThat(refund.getCreditCard()).isEqualTo(creditCard);
    assertThat(refund.getOrder()).isEqualTo(order);
}

@Test
public void 환불테스트__환불하려는_신용카드가_null_인경우_IllegalArgumentException이_발생해야_한다(){
    Assertions.assertThatThrownBy(()->{
        Refund.byCreditCardBuilder()
                .creditCard(null)
                .order(order)
                .build();
    }).isInstanceOf(IllegalArgumentException.class);
}
```
<br>

신용카드 주문의 경우 신용카드 객체가 없으면 환불 객체가 환불할 수 없도록 IllegalArgumentException 을 올바로 던지고 있다.<br>
<br>
<br>

#### 전체 테스트 코드
{: .fs-5 .fw-700 }

테스트 코드 전체를 코드로 남겨뒀다.
```java
package io.study.lombok.lombok_howto.lomok_data;

import io.study.lombok.lombok_howto.address.Address;
import io.study.lombok.lombok_howto.order.Order;
import io.study.lombok.lombok_howto.payment.Account;
import io.study.lombok.lombok_howto.payment.CreditCard;
import io.study.lombok.lombok_howto.refund.Refund;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class MultipleBuilderTest {

    private Account account;
    private CreditCard creditCard;
    private Address address;

    private Order order;

    @BeforeEach
    public void init(){
        account = Account.defaultBuilder()
                .accountNumber("111111")
                .accountHolder("Warren Buffet")
                .bankName("신한은행")
                .build();

        creditCard = CreditCard.defaultBuilder()
                .creditHolder("Warren Buffet")
                .creditNumber("99999999")
                .build();

        address = Address.defaultBuilder()
                .address1("경기도 성남시 분당구 판교로 777777")
                .address2("신한은행빌딩")
                .zip("1111111")
                .build();

        order = Order.defaultBuilder()
                .address(address)
                .build();
    }

    @Test
    public void 환불테스트__무통장입금(){
        Refund refund = Refund.byAccountBuilder()
                .account(account)
                .order(order)
                .build();

        assertThat(refund.getAccount()).isEqualTo(account);
        assertThat(refund.getOrder()).isEqualTo(order);
    }

    @Test
    public void 환불테스트__환불계좌가_null_일경우_IllegalArgumentException이_발생해야_한다(){
        Assertions.assertThatThrownBy(()->{
            Refund.byAccountBuilder()
                    .account(null)
                    .order(order)
                    .build();
        }).isInstanceOf(IllegalArgumentException.class);
    }

    @Test
    public void 환불테스트__신용카드(){
        Refund refund = Refund.byCreditCardBuilder()
                .creditCard(creditCard)
                .order(order)
                .build();

        assertThat(refund.getCreditCard()).isEqualTo(creditCard);
        assertThat(refund.getOrder()).isEqualTo(order);
    }

    @Test
    public void 환불테스트__환불하려는_신용카드가_null_인경우_IllegalArgumentException이_발생해야_한다(){
        Assertions.assertThatThrownBy(()->{
            Refund.byCreditCardBuilder()
                    .creditCard(null)
                    .order(order)
                    .build();
        }).isInstanceOf(IllegalArgumentException.class);
    }

}
```
<br>
<br>

### 나머지 참고용도 소스코드
{: .fs-6 .fw-700 }

Order, Address, Account, CreditCard 에 대한 코드들은 아래에 남겨뒀다.
아래의 코드를 보는 것보다 깃허브 코드를 보는게 더 보기 편할 것 같아 [깃허브 링크](https://github.com/gosgjung/0-code-docs-study-backend/tree/main/lombok_howto)도 남겨뒀다.
- [깃허브 링크](https://github.com/gosgjung/0-code-docs-study-backend/tree/main/lombok_howto)
<br>
<Br>

**Order.java**<br>
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Address address;

    @Builder(builderMethodName = "defaultBuilder")
    public Order(Address address) {
        Assert.notNull(address, "주소는 비어있을 수 없습니다.");
        this.address = address;
    }
}
```
<br>
<br>

**Address.java**<br>
```java
package io.study.lombok.lombok_howto.address;

import jakarta.persistence.Embeddable;
import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.util.Assert;

@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Embeddable
public class Address {

    private String address1;

    private String address2;

    private String zip;

    @Builder(builderMethodName = "defaultBuilder")
    public Address(String address1, String address2, String zip) {
        Assert.hasText(address1, "시/군/구 건물번호 는 비어있을수 없습니다.");
        Assert.hasText(address2, "상세 주소는 비어있을 수 없습니다.");
        Assert.hasText(zip, "우편번호는 비어있을 수 없습니다.");

        this.address1 = address1;
        this.address2 = address2;
        this.zip = zip;
    }

}
```
<br>
<br>

**Account.java**<br>
```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Embeddable
public class Account {

    @Column(name = "bank_name", nullable = false)
    private String bankName;

    @Column(name = "account_number", nullable = false)
    private String accountNumber;

    @Column(name = "account_holder", nullable = false)
    private String accountHolder;

    @Builder(builderMethodName = "defaultBuilder")
    public Account(String bankName, String accountNumber, String accountHolder) {
        Assert.hasText(bankName, "계좌명은 비어있으면 안됩니다.");
        Assert.hasText(accountNumber, "계좌번호는 비어있으면 안됩니다.");
        Assert.hasText(accountHolder, "예금주는 비어있으면 안됩니다.");
        this.bankName = bankName;
        this.accountNumber = accountNumber;
        this.accountHolder = accountHolder;
    }
}
```
<br>
<br>


**CreditCard.java**<br>
```java
package io.study.lombok.lombok_howto.payment;

import jakarta.persistence.Column;
import jakarta.persistence.Embeddable;
import lombok.AccessLevel;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.springframework.util.Assert;

@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Embeddable
public class CreditCard {

    @Column(name = "credit_number", nullable = false)
    private String creditNumber;

    @Column(name = "credit_holder", nullable = false)
    private String creditHolder;

    @Builder(builderMethodName = "defaultBuilder")
    public CreditCard(String creditNumber, String creditHolder) {
        Assert.hasText(creditNumber, "신용카드 번호는 비어있을수 없습니다.");
        Assert.hasText(creditHolder, "신용카드 소유자는 비어있을수 없습니다.");
        this.creditNumber = creditNumber;
        this.creditHolder = creditHolder;
    }
}
```
