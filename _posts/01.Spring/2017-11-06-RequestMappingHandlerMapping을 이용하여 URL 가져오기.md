---
title:  "RequestMappingHandlerMapping을 이용하여 URL 가져오기"
categories: Spring
tags: spring
---

메뉴관리 화면에서 사용자 편리성의 위해 스프링에서 RequestMapping 어노테이션을 통해 등록한 url을 조회할 수 있는 기능을 만들기 위해 예제를 만들어 보았다.

> **Note:** RequestMappingHandlerMapping 클래스는 Controller와 url을 매핑시켜주는 HandlerMapping의 구현클래스이다. (RequestMappingHandlerMapping --(인터페이스)--> MatchableHandlerMapping --(상속)--> HandlerMapping)

### Controller

테스트를 위한 url을 RequestMapping 어노테이션을 통해 매핑하였다. 

```java
package com.tistory.one0.ctrl;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Won Young, Park
 * @date 2017-01-06
 */
@RestController
@RequestMapping("home")
public class HomeController {

    @GetMapping(value = "/test1.do")
    public String getGetTest() {
        return "GetMapping Test1";
    }

    @GetMapping(value = "/test2.do")
    public String getGetTest2() {
        return "GetMapping Test2";
    }

    @PostMapping(value = "/test1.do")
    public String getPostTest() {
        return "PostMapping Test1";
    }

    @PostMapping(value = "/test2.do")
    public String getPostTest2() {
        return "PostMapping Test2";
    }
}
  
```



### 실행 클래스

```java
package com.tistory.one0;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

import java.util.*;

@SpringBootApplication
public class SpringbootHandlermappingApplication {

	 @Autowired
	 private RequestMappingHandlerMapping handlerMapping;

	private static final Logger logger = LoggerFactory.getLogger(SpringbootHandlermappingApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(SpringbootHandlermappingApplication.class, args);
	}

	@Bean
	public CommandLineRunner run() throws Exception {
		return args -> {
			logger.info("###############################################");
			Map<RequestMappingInfo, HandlerMethod> mappings = handlerMapping.getHandlerMethods();
			Set<RequestMappingInfo> keys = mappings.keySet();
			Iterator<RequestMappingInfo> iterator = keys.iterator();

			List<String> urlList = new ArrayList<String>();
			while (iterator.hasNext()) {
				RequestMappingInfo key = iterator.next();
				HandlerMethod value = mappings.get(key);
				urlList.add(key.getPatternsCondition().toString());

				logger.info(key.getPatternsCondition()  + "[" + key.getMethodsCondition()  + "] : " + value);
			}
			logger.info("###############################################");
		};
	}

	@Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
       RequestMappingHandlerMapping requestMappingHandlerMapping = new RequestMappingHandlerMapping();
       return requestMappingHandlerMapping;
    }
}

```



### 결과

RequestMapping 어노테이션을 이용해 매핑한 url이외에 error라는 url이 보인다. 필자도 자세히는 모르겠지만 springboot 기본 옵션으로 추측된다.

```java
2017-01-06 21:38:10.698  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : ###############################################
2017-01-06 21:38:10.698  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/home/test2.do][[GET]] : public java.lang.String com.tistory.one0.ctrl.HomeController.getGetTest2()
2017-01-06 21:38:10.698  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/home/test2.do][[POST]] : public java.lang.String com.tistory.one0.ctrl.HomeController.getPostTest2()
2017-01-06 21:38:10.698  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/home/test1.do][[POST]] : public java.lang.String com.tistory.one0.ctrl.HomeController.getPostTest()
2017-01-06 21:38:10.698  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/home/test1.do][[GET]] : public java.lang.String com.tistory.one0.ctrl.HomeController.getGetTest()
2017-01-06 21:38:10.699  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/error][[]] : public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-01-06 21:38:10.699  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : [/error][[]] : public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-01-06 21:38:10.699  INFO 19676 --- [           main] .t.o.SpringbootHandlermappingApplication : ###############################################
```

