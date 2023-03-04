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
