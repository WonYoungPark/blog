---
title: "[5th] Oracle Developer Meetup"
categories: Meetup
tags: meetup
published: true
comments: true
---



오라클에서 주최하여 진행하고 있는 Oracle Developer Meetup을 다녀왔습니다. 같은 날에 스프링캠프를 진행하기 때문인지 금일 방문하신 분들이 지난번보다는 적었지만, 좋은 발표주제와 발표자 분들 덕분에 유익한 시간이였습니다.

금일 발표 주제는 크게 **보안을 고려한 애플리케이션 개발 공정 및 실무적 수행 방법 소개**와 **CQRS(Command Query Responsibility Segregation) 와 Event Sourcing 패턴 실습**이였습니다. 개인적으로 CQRS가 궁금하기 위해서 밋업에 참가를 하였는데 두가지 발표 모두 유익한 내용이였습니다.



## Microservie

기존의 모놀리티 기반의 서비스에서 마이크로 서비스로 패러다임이 바뀌고 있는 이유는 현재 우리의 시대 상황들이 원하는 요건을 충족하기 위해서는 마이크로 서비스가 필요하기 때문이다. 마이크로서비스 자체도 최근에 나온 기술이 아니라 예전부터 있었던 개념입니다.

마이크로 서비스의 복잡한 내용들을 패턴화 하여 정리하신 분이 계신데, 이 분의 자료는 [여기서]() 보시면 됩니다. 이 패턴들 중에서 데이터 패턴에 대한 발표입니다.



### Data Patterns

기존의 모놀리티 시스템의 경우 Database 또한 하나의 데이터베이스를 이용하기 때문에 트랜잭션 처리가 쉬웠지만 마이크로서비스 처럼 데이터베이스가 분산되어있을 경우 트랜잭션이 어려운 문제가 있습니다.



#### 마이크로서비스과 모노리티서비스의 차이점

- 트랜잭션
- 쿼리(조인을 사용할 수 없음)
  - 이벤트 드레이븐 방식으로 해결할 수 있음 -> But 하나의 트랜잭션과는 다르게 약간의 소요시간이 발생함.
- Materialized View 사용



## Event Sourcing

> 애플리케이션의 모든 상태 변화를 발생 순서에 따라 이벤트로 보관한다. - 마틴파울러

각각의 발생하는 이벤트 자체를 저장하는 것을 말합니다. 기존에 결과만을 저장하는 history와는 다른 관점입니다.

데이터를 변경하지 않고, 저장만 한다.(과거데이터를 저장하기 위함)



#### 장점

- Event Driven Architecuter를 구현하면서 Atomic한 데이터 Pubilsh가 가능
- 특정 시점의 상태값을 조회할 수 있다.



#### 단점

- 러닝커브가 높다.

- Event Store기반의 쿼리에 한계점으로 인해 CQRS가 필수

  ​

## CQRS

명령과 쿼리의 역할 구분(Command and Query Responsibility Segregation)의 약자입니다. 오브젝트 차원에서 데이터를 변경하는 클래스와 리드하는 클래스를 분리해야 한다.



## Refernece

- https://github.com/DannyKang/CQRS-ESwithAxon



## Hands-on



### 결론

