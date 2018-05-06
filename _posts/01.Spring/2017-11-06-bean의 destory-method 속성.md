---
title:  "bean의 destory-method 속성"
categories: Spring
tags: spring
---

spring에서 dbcp를 설정하다보면 destroy-method 속성을 흔치않게 확인 할 수 있다. destroy-method 속성이 무엇인지에 대해 알아보도록 하겠다.

### destory-method 속성

해당 속성을 확인해 보면 bean 객체의 스코프가 끝날을 경우(스프링에서는 어플리케이션 컨텍스트가 종료되었을 경우로 생각하면 되겠다.) class 속성에 선언한 클래스의 close 메서드를 호출하는 의미이다.

```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  
    destroy-method="close"
    p:driverClassName="${db.driverClassName }"
    p:url="${db.url}"
    p:username="${db.username}"
    p:password="${db.password}"
    p:maxActive="${db.maxActive}"
    p:maxIdle="${db.maxIdle}"
    p:maxWait="${db.maxWait}"
/>
```

### close Method

close메서드를 디컴파일하여 확인해 본 결과 exception 처리와 connection close 처리 이외에 다른 기능은 없었다.

```java
public synchronized void close() throws SQLException {
        closed = true;
        GenericObjectPool oldpool = connectionPool;
        connectionPool = null;
        dataSource = null;
        try {
            if (oldpool != null) {
                oldpool.close();
            }
        } catch(SQLException e) {
            throw e;
        } catch(RuntimeException e) {
            throw e;
        } catch(Exception e) {
            throw new SQLNestedException("Cannot close connection pool", e);
        }
    }
```

### Test

여기서 필자의 궁금증이 발생하여서 아래와 같은 테스트를 진행해 보았다.

1. 톰캣 정상 종료 후 destroy-method 속성 설정 또는 미설정시 connection 유지 여부

   -> connection close

2. 톰캣 비정상 종료 후 destroy-method 속성 설정 또는 미설정시 connection 유지 여부

   -> connection close

### 결론

destroy-method속성을 하지 않더라도 connection close 처리를 해주고 있었다. default 처리가 되어있다고 생각하여 커뮤니티 게시글에에 질문글을 남겨보았더니 어떤 분께서 spring 문서를 확인보라는 답글과 [spring 문서 링크](http://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/javadoc-api/org/springframework/context/annotation/Bean.html) 를 남겨주셨다.
문서를 번역하면 아래와 같다.

> **Note :** 사용자가 편리하게 사용할 수 있도록 컨테이너는 @Bean 메서드에서 반환 된 객체에 대해 destroy 메소드를 유추하려고합니다. 예를 들어, Apache Commons DBCP BasicDataSource를 반환하는 @Bean 메서드가 있으면 컨테이너는 해당 객체에서 close () 메서드를 사용할 수 있음을 확인하고 자동으로 destroyMethod로 등록합니다. 이 'destroy 메소드 추론'은 현재 'close'또는 'shutdown'이라는 public, no-arg 메소드 만 탐지하는 것으로 제한됩니다. (즉, 생성시 Bean 인스턴스 자체에 대해 탐지가 반영됩니다)
