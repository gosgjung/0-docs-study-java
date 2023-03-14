---
layout: default
title: Builder - 객체생성시 유효성 체크는 철저히
nav_order: 2
has_children: false
parent: Lombok 제대로 써보자
grand_parent: Java Basic
permalink: /docs/java-basic/lombok-howto/lombok-builder-validation-1
---


# Builder - 객체생성시 유효성 체크는 철저히
{: .no_toc }
<br>

- @Builder 에 비어있는 값이 있을 경우를 체크하자
- 빌더 구문 내에서의 필수값 체크는 확실하게 하자
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


### Builder 패턴의 허점
{: .fs-6 .fw-700 }

빌더는 객체 생성을 편리하고 안전하게 할 수 있다는 점은 장점이다. 다만, 빌더 패턴을 통해 객체 생성시 누락될 수 있는 필드들에 대한 유효성 체크를 확실하게 해야 한다는 점도 간과하면 안된다.

예를 들어 주식 Ticker 에 대한 클래스 Stock 이 있을 때 주식 종목에 대한 국가 코드는 없으면 안된다. 그리고 ticker 명, 종목 영문명, 종목명 은 비어있으면 안된다.

주식 Ticker 에 대한 클래스인 `Stock` 클래스 예제를 통해서 나쁜 예와 좋은 예를 확인해보자.
<br>
<br>

### 나쁜 예 : Stock1.java
{: .fs-6 .fw-700 }

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Stock1 {

    private String ticker;
    private String companyNameEn;
    private String companyName;
    private String sector;
    private String countryCode;

    @Builder(builderMethodName = "defaultBuilder")
    public Stock1(String ticker, String companyNameEn, String companyName, String sector, String countryCode) {
        this.ticker = ticker;
        this.companyNameEn = companyNameEn;
        this.companyName = companyName;
        this.sector = sector;
        this.countryCode = countryCode;
    }

}
```
<br>

@Builder 를 이용해서 객체를 생성하는 평이한 구문이다.<br>

Stock 클래스의 필수 필드는 ticker 명, 종목 영문명, 종목명, 국가코드다.<br>

그런데 위의 @Builder가 적용된 생성자 구문은 아래의 문제점이 있다.<br>

- ticker 명이 null 인지, 비어있는지를 체크하지 않는다.
- companyNameEn 명이 null 인지, 비어있는지를 체크하지 않는다.
- companyName 명이 null 인지, 비어있는지를 체크하지 않는다.
- countryCode 명이 null 인지, 비어있는지를 체크하지 않는다.

<br>

테스트코드
```java
@Test
public void TEST_NULL_1(){
    Stock1 amzn = Stock1.defaultBuilder()
            .ticker("AMZN")
            .sector("SALES")
            .companyName("Amazon")
            .companyNameEn("Amazon")
            .countryCode(" ") // 국가 코드가 비어있는 상태로 객체를 생성해보자.
            .build();

    assertThat(amzn.getCountryCode()).isBlank();

    Stock1 meta = Stock1.defaultBuilder()
            .ticker("META")
            .sector("IT SERVICE")
            .companyName("Meta")
            .companyNameEn("Meta")
            .countryCode(null) // 국가 코드를 null 로 대입했다.
            .build();

    assertThat(meta.getCountryCode()).isNull();
}
```
<br>
<br>

### 좋은 예 : Stock2.java
{: .fs-6 .fw-700 }

위의 코드에서 객체 생성을 완전하게 하기위해 유효성체크를 하는 구문을 추가해보자.

```java
@Getter
@NoArgsConstructor
public class Stock2 {

    private String ticker;
    private String companyNameEn;
    private String companyName;
    private String sector;
    private String countryCode;

    @Builder(builderMethodName = "defaultBuilder")
    public Stock2(String ticker, String companyNameEn, String companyName, String sector, String countryCode) {
        Assert.hasText(ticker, "Ticker must not be empty.");
        Assert.hasText(companyName, "Company name must not be empty.");
        Assert.hasText(companyNameEn, "Company name in English must not be empty");
        Assert.hasText(countryCode, "Country code must not be empty");

        this.ticker = ticker;
        this.companyNameEn = companyNameEn;
        this.companyName = companyName;
        this.sector = sector;
        this.countryCode = countryCode;
    }
}
```
<br>

Stock 데이터에 대한 필수 조건은 아래와 같았다.
- ticker 명이 null 인지, 비어있는지를 체크하지 않는다.
- companyNameEn 명이 null 인지, 비어있는지를 체크하지 않는다.
- companyName 명이 null 인지, 비어있는지를 체크하지 않는다.
- countryCode 명이 null 인지, 비어있는지를 체크하지 않는다.

위의 빌더 구문에서는 위의 조건에 대한 유효성 체크를 수행하고 있다.<br>

이렇게 유효성 체크 후 예외를 발생시키는 구문을 객체 생성 구문에 작성해두면, 객체 외부에서 null 체크를 쓸데없이 안해도 된다.<br>

그런데 혹시라도 다른 사람이든, 다른 팀의 직원이든 Stock 도메인에 대해 이해하지 못하고 Stock 도메인의 코드를 건드려서 장애가 나는 상황도 있을 수 있다. 조금 무리수를 둔 상상이긴 하다. 이렇게 의도치 않은 코드 변경시에 그 코드를 수정한 것이 잘한 짓?인지를 체크하는 방법이 있다.<br>

**테스트 코드**를 작성해두면 된다.<br>

테스트코드
```java
@Test
public void TEST_NULL_2(){
    Assertions.assertThatThrownBy(()->{
        Stock2 amzn = Stock2.defaultBuilder()
                .ticker("AMZN")
                .sector("SALES")
                .companyName("Amazon")
                .companyNameEn("Amazon")
                .countryCode(" ") // 국가 코드가 비어있는 상태로 객체를 생성해보자.
                .build();
    }).isInstanceOf(IllegalArgumentException.class);

    Assertions.assertThatThrownBy(()->{
        Stock2 meta = Stock2.defaultBuilder()
                .ticker("META")
                .sector("IT SERVICE")
                .companyName("Meta")
                .companyNameEn("Meta")
                .countryCode(null) // 국가 코드를 null 로 대입했다.
                .build();
    }).isInstanceOf(IllegalArgumentException.class);
}
```
<br>
<br>

이렇게 해두면 변경사항 발생시 매번 화면 보고 빠진 부분 있는지 체크하다가 어디까지 체크했는지 기억 안나서 처음부터 다시 체크하고 하는 일을 방지할 수 있다. 그냥 버튼 한번 눌러서 계속 체크해보면서 개발하다가 화면확인만 나중에 개발서버 배포전에 관련 한꺼번에 테스트 하면 된다.<br>

테스트 코드 작성은 개발 속도를 느리게 하는게 아니다!!! 라는 것을 위의 코드를 보면 증명이 되는 것 같다.<br> 


