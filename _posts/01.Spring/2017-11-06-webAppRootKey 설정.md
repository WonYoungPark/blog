---
title:  "webAppRootKey 설정"
categories: Spring
tags: spring
---

하나의 Tomcat에 context가 두개 이상 적재되어 있고, log4j나 logback과 같은 로깅 프레임워크를 사용하고 있다면 아래와 같은 에러가 발생한다.

```xml
Exception sending context initialized event to listener instance of class org.springframework.web.util.Log4jConfigListener
```

> **Note:** webAppRootKey값을 지정해주지 않으면 default 값인 webapp.root로 중복되어 에러가 발생한다.



```
- web.xml -

<context-param>
	<param-name>webAppRootKey</param-name>
	<param-value>webapp.aaa</param-value>
</context-param>
```

