---
layout: default
title: Item 7. 불필요한 객체 생성을 피하라
nav_order: 7
has_children: false
parent:  Ch01
grand_parent: Effective Java
permalink: /docs/effective-java/chapter01/item07
layout: default
---



# Item 7. 불필요한 객체 생성을 피하라
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

메모리 누수와 메모리 누수가 발생할 수도 있는 코드를 설명하고 있다. 다 쓴 객체는 null 참조를 두면 JVM이 이것을 인지해서 회수하게 된다.<br>

> 가비지 컬렉터가 보기에는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체다. 비활성 영역의 객체가 더 이상 쓸모 없다는 것은 프로그래머만 아는 사실이다. 따라서 프로그래머는 비활성영역이 되는 순간 null 처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.

<br>

### 메모리 누수
{: .fs-6 .fw-700 }
- 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하되는 현상
- 상대적으로 드문 경우이지만, 심하면 디스크 페이징, OutOfMemoryError 를 일으켜 프로그램이 예기치 않게 종료되기도 한다.
<br>
<br>

### 간단한 스택 프로그램의 예
{: .fs-6 .fw-700 }
아래의 이 스택을 사용하는 프로그램을 오래 실행하다 보면 점점 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하된다

- 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다.
- 즉, pop() 시에 스택에서 객체를 꺼내면서 가비지 컬렉터가 회수하게끔 작성되지 않은 코드다.
<br>

```java
public clsass Stack{
  private Object[] elemetns;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack(){
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e){
    ensureCapacity();
    elements[size++] = e;
  }
  
  public Object pop(){
    if(size == 0) throw new EmptyStackException();
    return elements[--size];
  }
  
  private void ensureCapacity(){
    if(elements.length == size())
      elements = Arrays.copyOf(elements, 2*size + 1);
  }
}
```
<br>

### pop 메서드 수정
{: .fs-6 .fw-700 }
pop() 시에 더 이상 사용하지 않을 데이터의 참조를 가비지 컬렉터가 인지할 수 있도록 코드를 작성해보면 아래와 같다.
```java
public Object pop(){
  if(size == 0) throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null;
  return result;
}
```
<br>

### 다 쓴 객체의 null 처리하는 것의 장점/단점
{: .fs-6 .fw-700 }
**장점**<br>
다 쓴 참조를 null 처리하면 다른 이점도 따라온다. 만약 null 처리한참조를 실수로 사용하려 하면 프로그램은 즉시 NullPointerException 을 던지면서 종료된다.<br>
<br>

**단점**<br>
모든 객체를 다 쓰자마자 일일이 null 처리하는 경우도 있다. 이럴 경우 프로그램을 필요 이상으로 지저분하게 만들게 된다. 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다. 만약 변수의 범위를 최소화해 정의(아이템 57)했다면, 이 일은 자연스럽게 이루어지게 된다.<br>
<br>


### null 처리를 해야 하는 경우들 
{: .fs-6 .fw-700 }
책에서는 `캐시` 에 대해 이야기하고 있는데, 책에서 이야기하는 '캐시' 는 RabbitMQ, Memcached 등과 같은 분산 캐시 서버를 의미하지는 않는다.<br>


- 프로그램 상의 로직 상에서 비활성영역이 되었을 경우 
  - 해당 영역을 null 처리해준다.
  - 가비지 컬렉터는 비활성 영역에서 참조하는 객체도 똑같이 유효한 객체다. 비활성영역의 객체가 더 이상 쓸모없다는 것은 프로그래머만 아는 사실이다.
  - 따라서 프로그래머는 비활성 영역이 되는 순간 null 처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.
  - 메모리를 직접 관리하는 클래스라면 프로그래머는 항상 메모리 누수에 주의해야 한다. 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 모두 null 처리해줘야 한다.
- 캐시(캐시서버가 아닌 Map\<String, Object\> 를 의미) 역시 메모리 누수를 일으키는 주범이다.
  - 객체 참조를 캐시에 넣고 나서, 이 사실을 까맣게 잊은채 그 객체를 다 쓴 뒤로도 한참을 그냥 뇌두는 일을 접할 수 있다.
  - WeakHashMap
    - 운 좋게 캐시 외부에서 키(key)를 참조하는 동안만(값이 아니다) 엔트리가 살아있는 캐시가 필요한 상황이라면, WeakHashMap 을 사용해 캐시를 만들자. 다쓴 엔트리는 즉시 자동으로 제거된다.
    - 단, WeakHashMap 은 이런 상황에서만 유용하다는 사실을 기억해야 한다.
    - 시간이 지날 수록 엔트리의 가치를 떨어뜨리는 방식을 사용
      - 쓰지 않는 엔트리를 백그라운드 스케쥴러( ex. ScheduledThreadPoolExecutor )를 활용하는 방식
      - 캐시에 새 엔트리를 추가할 때 부수작업으로 수행하는 방법
      - LinkedHashMap 의 경우 removeEldestEntry 메서드를 사용하는 방식으로 처리한다.
      - 더 복잡한 캐시를 만들려 한다면 `java.lang.ref` 패키지를 직접 활용해야 한다.
- 리스너, 콜백
  - 메모리 누수를 일으키는 세번째 주범은 리스너(listener) 또는 콜백(callback)이다.
  - 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않으면, 뭔가 조치를 취해주지 않는 한 콜백은 계속 쌓여간다. 이럴 때 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해간다. 
  - 예를 들어 WeakHashMap 에 키로 저장하면 된다.
<br>
<br>

