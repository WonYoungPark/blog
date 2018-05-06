---
title: "JSP 용량초과"
categories: Spring
tags: spring
---

메이븐은 자바 개발자가 자주 사용하는 빌드 툴[^1]중 하나입니다. 메이븐이라는 빌드툴을 사용하여 소스코드로 부터 개포 가능한 산출물(artifact)을 빌드(build)한다.



## 메이븐의 장점

1. 의존성 관리
2. 기본 규약 제공
3. 유용한 여러 플러그인

## 페이즈(Phase)와 골(Goal)

메이븐은 아무런 설정을 추가하지 않고도 메이븐에서 제공하는 기본적인 프로젝트 빌드기능을 사용할 수 있습니다. 각각의 프로젝트 빌드기능을 골(Goal)이라고 부르며, 다수의 공을 페이즈로 묶어서 한 번에 여러 작업을 순차적으로 실행할 수 있습니다.

## POM

메이븐의 핵심 설정파일이며, 프로젝트에 관한 모든 정보를 담고있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.one0</groupId>
	<artifactId>spring-boot-mybatis-xml</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<name>spring-boot-mybatis-xml</name>
	<description>spring-boot-mybatis-xml project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```



### 속성

> - groupId : 프로젝트의 package 명칭
> - artifactId : 산출물(artifact)의 명칭, **groupId 범위 내에서 유일해야만 한다.**
> - version : 산출물(artifact)의 현재 버전
> - name : 어플리케이션의 명칭
> - packaging : 산출물(artifact)의 유형[^2]
> - parent : 상위 프로젝트 명칭
> - dependency Management : 의존성 처리에 대한 기본 설정 영역
> - dependencies : 의존성 정의 및 설정 영역
> - properties : 설정에서 프로퍼티 파일로 사용하는 키/값
> - build : 소스 디렉토리, 리소스 디렉토리, 테스트 소스 디렉토리, 테스트 리소스 디렉토리 정보 설정

#### build

pom.xml에 build 설정값에 소스, 리소스, 테스트소스, 테스트 리소스 디렉토리를 설정해주지 않으면 default 디렉토리 구조에 맞춘다.

- 소스 디렉토리 : src/main/java
- 리소스 디렉토리 : src/main/resource
- 테스트 소스 디렉토리 : src/test/java
- 테스트 리소스 디렉토리 : src/test/resoirce

#### plugin

메이븐 플러그인을 정의합니다.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.5.1</version>
            <configuration>
                <source>1.6</source>
                <target>1.6</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-war-plugin</artifactId>
            <configuration>
                <warSourceDerectory>web</warSourceDerectory>
            </configuration>
        </plugin>
    </plugins>
</build>
```

- source : 소스코드에서 사용한 자바버전 명시
- target : 소스코드 컴파일할 때 사용할 버전 설정
- encoding : 파일 인코딩 설정
- warSourceDerectory : 메이븐 기본 웹 디렉토리 설정(default : webapp)

[^1]: Maven, Ant, Gradle등이 있다.
[^2]: jar, war, ear, ejb 등에서 선택 가능







