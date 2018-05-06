---
title: "KSUG(Korea Srping User Group) 세미나 회고"
categories: Meetup
tags: meetup
comments: true
---

2017년 11월 26일 KSUG(Korea Spring User Group) 세미나를 참석하게 되었다. 원래는 티켓팅이 늦어서 참석을 못할 줄 알았는데, 운이 좋게도 티켓을 양도 받아서 참석 할 수 있었다. 금일 세미나의 시간표는 아래와 같다.

- Java 9 new Feature - 양수열
- HTTP/2세대의 Java - 정상혁
- Introduce JSON-B, JSON-P, JPA 2.2 Specification - 이명현
- What's New in Spring Freamework 5.0 - 정윤진



### 1. Java 9 new Feature 

해당 강의 발표자님께서는 양수열님으로 onFace에서 CTO로 재직 중이신 분이다. 자바9는 오라클 로드맵 상으로 2020년 까지 개발 예정이기 때문에 안전성 문제로 실무에서 쓰기는 다소 이를거 같다고 하신다. 

자바의 경우 버전 업이 되도라도 하위 호환성을 항상 보장해 주었던 반면에 자바9 경우에는 하위 버전 호환성을 보장해 주지 않는다.



#### 1.1. Standars

- HTML5 javadocs(JEP 224)
- Unicode 7.0/8.0
- PKCS12 key stores by default
- SHA 3 hash algorishtm

#### 1.2. Major Features

#####1.2.1. Project Jigsaw

- Modular source code(JEP 201)
  - 패키지 위에 모듈이 추가됨.
- Modular run-time images(JEP 220)
- Encapsulate Most Internal APIs (JEP 260)
- Compatability is an issue
- Leave everything classpath to start with
- **Big Kill switch** now has options

기존의 트리구조에서 모듈 구조로 변경되었다. 이것의 문제점이라면, 기존의 Java open source의 경우 패키지 구조로 되었기 때문에 기존의 모든 java 소스들이 모듈화 구조로 변경되야 한다는 점이다.

반대로 모듈화의 장점이라고 한다면 최근 cloud 환경이 대세인 가운데 cloud 환경에서 작은 모듈화 구조로 개발을 진행 했을 경우 많은 이득이 있을 것으로 예측된다.

모듈단위로 java 패키지를 선택하여 구성할 수 때문에 불필요한 모듈을 배제하여 시스템 자체의 크기를 줄일 수 있기 때문에 모든 리소스에 따라 금액을 지불하는 클라우드 환경에서 많은 장점이 될 것으로 보인다.



#### 1.2.2.Multi-Release Jar Files

기존의 자바의 패키지 구조에서 모듈화 구조로 변경됨으로서 해당 jar 릴리즈의 경우에도 호환성이 필요 하기 때문에 java버전 별로 릴리즈 방법을 선택할 수 있도록 변경 되었다.

#### 1.2.3. jshell:The Java Shell(Read-Evaluate-Print-Loop)

#### 1.2.4. Enhanced Deprecation

기존의 중복되거나 사용되지 않는 메소드 메시징 기능을 보완하였다.

#### 1.2.5. Make G1 the Default Garbage Collector

기존의 가비지 콜렉터가 새롭게 변경되었다. JVM상에서 GC가 발생하게 되면 모든 기능이 일시정지 되기 때문에 GC의 경우 굉장히 중요한 부분이다. 새롭게 바뀐 가비지 콜렉터의 경우 기존보다 성능적으로 우수해 졌다고 한다.

#### 1.2.6. Better String Performance

스트링 클래스가 보다 성능적으로 향상되었다.



### 2. HTTP/2세대의 Java

HTTP/2 프로토콜은 2015년 5월경에 스펙이 공표되었고, 이미 naver나 facebook 등 여러 사이트에서 적용되어 있다.

HTTP/1.1과 HTTP/2의 차이점은 전송 방법이 변경되었다.(텍스트 타입이 아닌 바이너리 방식)

####2.1. HTTP/2를 사용하는 이유

- 더 나은 선능
  - 헤더 압축

#### 2.2. Java에서의 HTTP/2 지원

- Java9 : HttpClient 추가
- Java9 : ALPN 지원
  - Java 서버에서 TLS offloading을 할 때 사용
- 이상적인 서버 구성
  - 서버 앞에서 TLS처리(자바 단에서 TLS처리 시 3.2ms -> 24ms로 느려졌다는 사례가 존재함.)



### 3. Introduce JSON-B, JSON-P, JPA 2.2 Specification

##### 3.1. JPA2.2

- @Repeatable marked
- java 8 Date and Time types
- Stream for query result
- AttrbuteConverters to support CDI injection





### 4. What's New in Spring Freamework 5.0



작성 중...

