---
layout: default
title: Data 어노테이션 사용은 나쁘다
nav_order: 1
has_children: false
parent: Lombok 제대로 써보자
grand_parent: Java Basic
permalink: /docs/java-basic/lombok-howto/why-do-not-use-lombok-data
---


# Data 어노테이션 사용은 나쁘다
{: .no_toc }
<br>

객체는 그 객체 자체의 역할,책임이 분명하게 드러나게 만드는 게 좋다.
@Data 에는 각종 어노테이션들이 숨어서 함정을 파놓고 기다리고 있기에, 아무리 일정이 촉박해도 고민을 거듭한 뒤에 추가하는 것이 좋다.
쓰다보니, `Setter 는 나쁘다` 라는 내용이 더 커지고 있는 것 같다. 
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


### @Data 사용의 단점, 부작용
{: .fs-6 .fw-700 }

- `@Setter` 를 무분별하게 남용하게 된다.
- `@ToString` 양방향 순환 참조 문제 발생
- `@EqualsAndHashCode` 를 남용하게 된다.

<br>
<br>

### 1\) @Setter 를 사용하는 것으로 인한 단점
{: .fs-5 .fw-700 }
예를 들어 아래와 같은 코드가 있다고 해보자.

```java
@Data
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
    private String name;

    @OneToMany
    @JoinColumn(name = "coupon_id")
    private List<Coupon> coupons = new ArrayList<>();
}
```

<br>

흔히 setter로 데이터를 변경하는 방식은 흔히 'bean 방식' 이라고 부른다.

 `@Data` 내에는 `@Setter` 가 있기에 `Member` 객체는 setter 가 적용되어 email 을 변경할 수 있다는 의미가 된다. 그런데 만약 `Member`  의 email 을 변경하는 기능이 요구사항에 없다면 Setter 로 인해 기능의 불일치가 발생한다. <br>

즉, 소프트웨어의 어느 부분에서든 email 을 변경할 수 있는 소지를 남겨놓은 상태가 되어버린다.<br>

이미 한번 생성된 객체 내의 email을 setter로 접근해서 애플리케이션의 곳곳에서 수정하는 로직들이 독버섯처럼 퍼져있는 경우도 생각해볼 수 있다.

예를 들면 아래와 같은 경우다.

```java
public void encodeEmail(Member member){
    member.setEmail(Base64.getEncoder().encodeToString(member.getEmail().getBytes()));
}

public void addOAuth2PrefixVendor(Member member, String vendor){
    final String email = new StringBuilder("###OAuth2###").append(vendor)
            .append("###").append(member.getEmail())
            .toString();

    member.setEmail(email);
}

@Test
public void SETTER_TEST(){
    final Member member = new Member();
    member.setEmail("johndoe@gmail.com");

    encodeEmail(member); // 1)

    // 메서드 내부에서 EMAIL 을 수정하는지 알 수 없다.
    addOAuth2PrefixVendor(member, "GOOGLE"); // 2)

    member.setEmail("안녕하세요"); // 3)
}
```
<br>

위의 코드에서는 Member 객체를 생성한 후에 객체 외부에서 email 필드를 수정하는 부분이 총 4번 호출되었다.

**1)  Base64 로 Email 을 인코딩한다**<br>
- 하지만, Member 객체의 email 을 수정한다는 사실은 파악하기 쉽지 않다.
- Member 객체의 email 을 수정하는지 체크하기 위해 `encodeEmail(Member member)` 메서드의 내부 구현을 한번 더 체크해봐야 한다.
<br>


**2) OAuth2 방식으로 유입된 사용자임을 구분하기위한 처리를 한다**
- 2\) 의 코드 역시 메서드 내부에서 Member 객체의 필드를 수정하는지는 명확하게 알 수 없다.
- 메서드 내부를 체크해봐야 한다.
<br>

**3) 모두 수정해두고 Member 객체의 외부에서 email 필드를 수정중이다**
- 이미 생성된 객체의 email 필드를 직접 수정하고 있다.
<br>
<br>


이미 생성된 객체에 setter 를 사용하는 로직들은 아래의 문제가 있지 않을까 싶다.

- 특정 필드를 어디에서 수정하는지 연관된 부분들을 일일이 이잡듯이 뒤져서 찾아내야 한다.
- 코드가 방대해지면, 막상 영향이 가는 부분인지 알수 없어지기에 고치기 쉽지 않은 소프트웨어가 된다.
- 이미 생성된 객체에 대해 set 을 하는지, 새로 생성한 객체에 대해 set 을 하는지도 일일이 callstack 을 일일이 그려가며 체크해야 한다.


여기에 더해 동시성/병렬성 환경에서도 문제가 된다. 생성된 객체의 외부가 멀티스레드를 사용하고 있고, 객체의 외부에서 setter를 호출하고 있을 때 setter 로 인해 객체 내의 데이터가 모호해지는 것을 방지할 수 없게 된다.
e.g.
```java
public void SomeTest{
    // ...

    public void addOAuth2PrefixVendor(Member member, String vendor){
        final String email = new StringBuilder("###OAuth2###").append(vendor)
                .append("###").append(member.getEmail())
                .toString();

        member.setEmail(email);
    }
}
```
<br>
<br>

잠깐 백업을 위해 임시커밋...

### 2\) @ToString - 양방향 순환 참조 문제
{: .fs-6 .fw-700 }

```java
@Entity
@Table(name = "member")
@Data
public class Member{
	// ...
	@OneToMany
	@JoinColumn(name = "coupon_id")
	private List<Coupon> coupons = new ArrayList<>();

}

@Entity
@Table(name = "coupon")
@Data
public class Coupon{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private long id;

	@ManyToOne
	private Member member;

	public Coupon(Member member){
		this.member = member;
	}
}
```

<br>



```java
@Test
// Member 클래스 내에 @Exclude 어노테이션을 주석 처리 후 실행하면
// StackOverflow가 발생하게 된다.
public void toString_양방향_순한_참조_문제() {
    final Member member = new Member();
    member.setEmail("asd@asd.com");
    member.setName("name");

    final Coupon coupon = new Coupon();
    coupon.setMember(member);

    final List<Coupon> coupons = new ArrayList<>();
    coupons.add(coupon);
    member.setCouponList(coupons);

    System.out.println(member); // toString 순한 참조 발생
}
```

<br>



### 3\) @EqualsAndHashCode 의 남용
{: .fs-6 .fw-700 }

