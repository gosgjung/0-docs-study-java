---
layout: default
title: Java 코드가 컴파일되어 실행되기까지
nav_order: 1
has_children: false
parent: JVM 구조 및 개념
grand_parent: JVM
permalink: /docs/jvm/JVM Structure/1-JVM-How-It-Works-When-Java-Compile-And-Run
---



# Java 코드가 컴파일되어 실행되기까지
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


### Java 코드를 컴파일하고 이것을 실행하기 까지
{: .fs-6 .fw-700 }
- 1\) JAVA 소스코드는 javac 라는 자바 컴파일러로 바이트코드로 컴파일한다. 
(.class 파일이 생성된다.)<br>
- 2\) JRE ClassLoader 는 JVM에 바이트코드를 적재한다. (JRE Class Loader -> JVM)<br>
- 3\) JVM 의 실행엔진(Execution Engine)은 바이트코드를 기계어로 변환하고 명령어 단위로 실행하는 역할을 수행한다.<br>
 (이때, 바이트 코드를 기계어로 변환할 때 사용하는 것이 JIT 컴파일러 또는 인터프리터다.)<br>

 <br>

### 1\) Java코드 → javac 로 바이트코드로 변환
{: .fs-6 .fw-700 }
Java 소스코드는 javac 컴파일러를 통해 .class 확장자를 가진 파일로 변환되는데, 이것을 바이트 코드라고 한다.<br>
<br>

### 2\) JRE ClassLoader 가 JVM 에 바이트코드를 적재
JRE ClassLoader 는 바이트코드를 JVM에 적재한다.<br>
<br>

### 3\) JVM 의 Execution Engine 이 바이트코드의 번역과 실행을 수행
JVM 의 실행엔진(Execution Engine)을 이용해 바이트코드를 기계어로 변환하고 명령어 단위로 실행하는 역할을 수행한다. (이때, 바이트 코드를 기계어로 변환할 때 사용하는 것이 JIT 컴파일러 또는 인터프리터다.)<br>