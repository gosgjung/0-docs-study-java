---
layout: default
title: (2021-2022) JVM 튜닝경험
nav_order: 2
parent: JVM
has_children: false
permalink: /docs/JVM/2021-2022-Experience
---
<br>

# 증권데이터 트래픽 처리 서버 개발
{: .fw-700 }
개발 초기에 웹 애플리케이션 서버의 성능을 보장하기 위해 JVM의 주요 Feature 들을 파악해서 최적화가 잘 수행되게끔 해주는 것은 중요하다.
예전에는 일이 너무 바빠서 느낄 시간도 없었지만, 시간이 어느 정도 흐른 지금에서는 전 직장에서 함께 JVM 관련 튜닝을 진행해줬던 동료들에게 감사한 마음을 가지고 있다.
<br>

예를 들어 내가 전 직장에서 증권데이터 처리 서버 개발 시에 작성했던 JVM 옵션들과 함께 구동하는 스크립트들을 서비스 명을 조금 바꿔서 정리해보면 아래와 같다.
<br>

jar 파일의 이름은 `trade_data_backend.jar` 이다. 
(jar 파일에 회사명이 포함되기에 다른 이름으로 대체했다.)
```bash
ExecStart=/home/ubuntu/env/java/java16/bin/java -jar -Xms16G -Xmx16G -Xlog:gc:/home/ubuntu/trade_data_backend/log-g1gc/trade_data-gc.log -XX:MaxDirectMemorySize=1G -server -jar -Dspring.profiles.active=data-live -XX:MetaspaceSize=1G -XX:MaxMetaspaceSize=1G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:ConcGCThreads=2 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8081 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false /home/ubuntu/trade_data_backend/trade_data_backend.jar
```

위의 옵션 들 중에서 주요 항목들만 나열해보면 아래와 같다.

- -XX:+UseG1GC
	- 가비지 컬렉터로 G1GC를 채택
- -XX:+ParallelRefProcEnabled
	- 레퍼런스 객체 처리 시에 참조 객체 업데이트를 병렬로 수행되도록 지정
- -XX:ConcGCThreads=2
	- 가비지 컬렉팅에 사용되는 스레드는 보수적으로 2개로 지정
- -XX:MaxDirectMemorySize=1G
	- 오프힙 캐시를 사용하는 경우 이 옵션을 사용한다.
	- 내 경우는 오프 힙 기반의 인메모리 캐시를 사용했기에 이 옵션을 사용했다.
- -XX:MetaspaceSize=1G
	- 자세한 내용은 [Java 의 메타스페이스(Metaspace)](https://jaemunbro.medium.com/java-metaspace%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-ac363816d35e) 를 참고

<br>