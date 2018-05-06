---
title: "Cannot get a connection, pool error Timeout waiting for idle object"
categories: Spring
tags: spring
---

현재 운영 중인 시스템에서 아래와 같은 오류가 발생하여 운영 중인 사이트가 먹통이 되는 현상이 발생하였다. 아래의 오류를 보면 커넥션 풀에서 커넥션을 빌려오는 동작이 타임 아웃되었다는 것이다.

해당 오류는 운영 중인 시스템에서 커넥션 풀에서 커넥션을 받은 후 반환하지 않아서 커넥션 풀에 여분의 커넥션이 없을때 발생한 것이다. 아래의 오류를 해결 방법은 다양하게 있는데 몇가지 알아보도록 하자.

```java
### Error updating database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is java.sql.SQLException: Cannot get a connection, pool error Timeout waiting for idle object
### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Could not get JDBC Connection; nested exception is java.sql.SQLException: Cannot get a connection, pool error Timeout waiting for idle object
```



### DBCP 설정

Spring을 사용 중이라면 대부분 dbcp를 사용 중일 것이다. 아래와 같은 설정을 통해서 사용하지 않은 커넥션을 주기적으로 검사하여 제거해 주도록 할 수 있지만, 이 설정의 경우 에러의 **근본적인 원인을 해결한 것이 아닌 임시 방편**에 불과하기 때문에 추천하지 않는 방법이다.

```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
	<property name="testWhileIdle" value="true" />
	<property name="timeBetweenEvictionRunsMillis" value="900000" />
	<property name="removeAbandonedOnMaintenance" value="true"/>
	<property name="removeAbandonedTimeout" value="600"/> 
</bean>
```

- testWhileIdle : 비활성 커넥션 유효검사 후 제거(timeBetweenEvictionRunsMillis 설정 활성화 되어 있을 경우 동작)

- timeBetweenEvictionRunsMillis : 설정된 주기를 통해 Evict(유효하지 않는 커넥션/정의된 시간이 만료된 커넥션을 풀에서 제거) 쓰레드를 수행

- removeAbandonedOnMaintenance : Evict 쓰레드를 수행시, 커넥션은 유지되고 있지만 사용하지 않은 커넥션 제거(커넥션 종료 안하고 커넥션을 유지하고 있을 경우

- removeAbandonedTimeout : 미사용 커넥션 제한시간(초)

  ​

### Transaction Commit OR RollBack

그렇다면 오류가 나는 근본적인 원인을 해결해 보도록 하자.

```xml
[2017-06-01 19:02:15,150] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Creating new transaction with name [AuthServiceSaveInfo]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
[2017-06-01 19:02:15,151] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Acquired Connection [405020982, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver] for JDBC transaction
[2017-06-01 19:02:15,154] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Switching JDBC Connection [405020982, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver] to manual commit
[2017-06-01 19:02:15,156] DEBUG: org.mybatis.spring.SqlSessionUtils - Creating a new SqlSession
[2017-06-01 19:02:15,157] DEBUG: org.mybatis.spring.SqlSessionUtils - Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6942bf69]
[2017-06-01 19:02:15,160] DEBUG: org.mybatis.spring.transaction.SpringManagedTransaction - JDBC Connection [405020982, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=TWAY_TEST2, Oracle JDBC driver] will be managed by Spring
[2017-06-01 19:02:15,160] DEBUG: userDAO.selectUserIdDupYn - ooo Using Connection [405020982, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC 
[2017-06-01 19:02:15,166] DEBUG: org.mybatis.spring.SqlSessionUtils - Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6942bf69]

```



```xml
017-06-01 19:15:34,472] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Creating new transaction with name [AuthServiceSaveInfo]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT
[2017-06-01 19:15:34,473] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Acquired Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver] for JDBC transaction
[2017-06-01 19:15:34,476] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Switching JDBC Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver] to manual commit
[2017-06-01 19:15:34,477] DEBUG: org.mybatis.spring.SqlSessionUtils - Creating a new SqlSession
[2017-06-01 19:15:34,478] DEBUG: org.mybatis.spring.SqlSessionUtils - Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6b150c1]
[2017-06-01 19:15:34,480] DEBUG: org.mybatis.spring.transaction.SpringManagedTransaction - JDBC Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxxx, Oracle JDBC driver] will be managed by Spring
[2017-06-01 19:15:34,481] DEBUG: userDAO.selectUserIdDupYn - ooo Using Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver]
[2017-06-01 19:15:34,485] DEBUG: org.mybatis.spring.SqlSessionUtils - Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6b150c1]
[2017-06-01 19:15:34,485] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Initiating transaction rollback
[2017-06-01 19:15:34,486] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Rolling back JDBC transaction on Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver]
[2017-06-01 19:15:34,488] DEBUG: org.mybatis.spring.SqlSessionUtils - Transaction synchronization rolling back SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6b150c1]
[2017-06-01 19:15:34,488] DEBUG: org.mybatis.spring.SqlSessionUtils - Transaction synchronization closing SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6b150c1]
[2017-06-01 19:15:34,488] DEBUG: org.springframework.jdbc.datasource.DataSourceTransactionManager - Releasing JDBC Connection [1049253751, URL=jdbc:oracle:thin:@xxx.xxx.xx.xxx:1522:XE, UserName=xxxx, Oracle JDBC driver] after transaction
[2017-06-01 19:15:34,488] DEBUG: org.springframework.jdbc.datasource.DataSourceUtils - Returning JDBC Connection to DataSource

```

위의 두개의 로그는 특정 시스템에서 Insert 로직을 실행 했을 때의 로그이다. 첫 번째 로그는 잘못 된 시스템의 로그이고, 두 번째 로그는 정상적으로 동작하는 시스템의 로그이다. 위의 두개의 로그의 차이점이 보이는가?

`[2017-06-01 19:15:34,488] DEBUG: org.springframework.jdbc.datasource.DataSourceUtils - Returning JDBC Connection to DataSource`

위의 로그 중 다른 점을 위와 같이 JDBC Connection을 반환하는 부분이다.

그렇다면 위의 코드는 어떤 점 때문에 JDBC Connection을 반환하지 않을까?? 알아보던 중 운영 중인 시스템의 코드 중 의심가는 부분이 있었다. 바로 트랜잭션 처리 방식이 달랐었다.



#### Transaction 방식

Spring에서 트랜잭션을 처리 하는 방식은 여러가지 존재하는데 현재 운영하고 있는 시스템에서는 두가지 방식을 동시에 사용했다.(프리랜서 분들이 개발한 시스템인데, 관리했던 사람이 아무도 없다보니 프리분들마다 다른 방식으로 개발을 한거같다 -_-)

- @Transactional 어노테이션 사용
- Programmatic 트랜잭션 처리

@Transaction 어노테이션 사용 시 별 문제는 발생하지 않았으나 Programmatic 트랜잭션 방식[^1]으로 처리 한 코드에서 의심가는 부분이 있었다.

```java
public Map<String, Object> save(TempVO tempVo){
  	Map<String, Object> result = new HashMap<String, Object>();
	
  	DefaultTransactionDefinition def = new DefaultTransactionDefinition();
  	def.setName("transaction"); //TR명 설정
  	def.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); //전달방식 지정
  	TransactionStatus status = transactionManager.getTransaction(def); //TR 매니저에게 상태제어
  	try {
    	int dataResult = tempDao.insertTemp(tempVo);
    	if(dataResult > 0) transactionManager.commit(status); //마지막 Commit 처리		
  	} catch (Exception e) {
    	transactionManager.rollback(status);
    	e.printStackTrace();
    	result.put("error",e.toString());
  	}

  	result.put("result", ( saveResult ? "OK" : "FAIL" ));
  	return result;
}
```

예를 들어 위와 같은 코드가 있다고 가정을 해보자. 위의 소스에서 문제점이 무엇있까? 위의 소스에서 문제점은 논리적 에러처리가 부족하다는 것이다.

insertTemp() 메서드의 결과 값을 dataResut변수에 반환하게 되는데 insert 결과값이 0일 경우의 처리가 부족한 로직이다. insert의 결과값이 0일 경우 commit 혹은 rollback 처리가 안되어 있다. 위의 코드와 같이 Programmatic 트랜잭션 처리를 사용 할 경우 **메서드가 끝난 시점에서 commit 혹은 rollback이 되지 않았을 경우 JDBC Connection이 반환되지 않는 문제점이 발생**한다.

추가적으로 Spring안에서 트랜잭션이 어떻게 동작하는지에 대해 공부하면 좋을거 같다.



[^1]: 프로그램 코드 안에 직접 트랜잭션 처리를 넣는 방식을 말한다. 