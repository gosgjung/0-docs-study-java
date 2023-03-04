---
layout: default
title: 정적 컴파일러, 인터프리터, JIT 컴파일러
nav_order: 2
has_children: false
parent: JVM 구조 및 개념
grand_parent: JVM
permalink: /docs/jvm/JVM Structure/2-Compiler-Interpreter-JIT-Compiler
---



# 정적 컴파일러, 인터프리터, JIT 컴파일러
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

<br>

### 참고자료
{: .fs-6 .fw-700 }

- [JIT 컴파일 - 위키](https://ko.wikipedia.org/wiki/JIT_%EC%BB%B4%ED%8C%8C%EC%9D%BC)
- [kingsubin.com](https://kingsubin.com/248?category=896578)

<br>


### 정적 컴파일러
{: .fs-6 .fw-700 }
실행 전에 무조건 컴파일을 해서 기계어로 번역해놓는 방식
C언어의 경우 여기에 속한다.
참고로 C++ 등의 언어는 최근 개선을 거듭해서 JIT 컴파일러를 사용하는 것으로 알려져 있다.
<br>

### 인터프리터
{: .fs-6 .fw-700 }
바이트코드, 소스코드를 최적화 없이 번역만을 수행
과거에는 인터프리터를 사용하는 스크립트 언어가 많았는데, 대부분의 언어들이 JIT 컴파일러를 사용하는 방식으로 전환중이다.
<br>

### JIT 컴파일러
{: .fs-6 .fw-700 }
정적 컴파일러 만큼 빠르면서 인터프리터 언어의 빠른 응답 속도를 가진다.
캐시 등을 이용한다.
<br>

JVM 은 JIT 컴파일러를 지원한다. JIT 컴파일러는 Just In Time 이라는 단어의 약자다.<br>

정적 컴파일러보다 빠르면서 인터프리터 언어의 빠른 응답속도를 가진다.<br>
 <br>

자바 코드를 기계어로 변환해서 JVM 의 실행엔진이 실행되기 까지는 아래와 같은 순서를 통해 그 과정이 이뤄진다.
- Java 코드 → 바이트코드 → JVM Execution Engine 이 JIT 컴파일러로 기계어를 번역하며 실행

조금 자세히 설명해보면, `.java` 파일로 존재하는 소스코드를 javac 를 통해 바이트 코드로 변환한다. 이렇게 변한된 바이트 코드는 JRE ClassLoader 를 통해 JVM에 적재한다. 이렇게 적재한 바이트 코드는 JVM의 실행엔진(Execution Engine) 에 의해 번역하면서 실행되는데, 이 때 JVM의 실행엔진(Execution Engine) 은 JIT 컴파일러를 사용해 번역을 수행한다.
 <br>

JIT 컴파일러는 정적 컴파일러 만큼 빠르면서 인터프리터 언어의 빠른 응답속도를 가진다는 것이 장점이다. JIT 코드는 일반적인 인터프리터 언어에 비해 훨씬 좋은 성능을 낸다. 심지어 경우에 따라 정적 컴파일러 언어보다 좋은 성능을 낸다. 이것은 실행 과정에 컴파일을 할 수 있기 때문에 나타나는 장점이다.<br>