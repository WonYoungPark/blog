---
title: "Illegal key size 에러 해결방안 및 JCE 정책 변경"
categories: Java
tags: java error security encrypt jce
published: true
comments: true
---



미국 수출 통상법에 따라 사용할 수 있는 키 길이 등에 제한이 걸려 있기 때문에 JAVA의 기본 정책으로 AES-128 암호화 방식까지만 사용이 가능합니다. 하지만 최근에 [JCE policy changes in Java SE 8u151, 8u152 and 8u162](https://golb.hplar.ch/2017/10/JCE-policy-changes-in-Java-SE-8u151-and-8u152.html)의 내용과 같이 JCE 정책이 변경된 것을 확인할 수 있습니다.

Java 8u162버전 미만에서 192bit 혹은 256bit를 사용하게 될 경우 다음과 같은 에러가 발생됩니다.

```
Exception in thread "main" java.security.InvalidKeyException: Illegal key size
```

이를 해결하는 방법은 아래와 같습니다.



## 1. 해결 방안

위의 에러를 해결할 수 있는 방법은 JDK 버전별로 방법이 다양합니다. 

### 1.1. Java 8u151 버전 미만

[웹 사이트](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)에서 *Unlimited Strength Jurisdiction Policy Files*를 다운로드 합니다. 다운로드 받은 파일을 압축 해제하면 두개의 파일(`local_policy.jar` 와 `US_export_policy.jar`)이 생성됩니다.  두 개의 파일을 `<java-home>/lib/security` 경로에 덮어씌웁니다.



### 1.2. Java 8u151, 152 버전 이상

JDK 1.8u151 버전 이상부터 위의 방법 이외에도 프로퍼티 변경을 통해서 에러를 해결할 수 있는 방법을 제시합니다.



#### 1.2.1. 프로퍼티 설정

JAVA 코드상에 다음과 같이 작성합니다.

```java
Security.setProperty("crypto.policy", "unlimited");
```



#### 1.2.2. 프로퍼티 파일 수정

`<jre_home>/lib/security/java.security` 파일을 에디터로 실행합니다. `crypto.policy=unlimited` 라인을 검색하고 `#`문자를 제거한 후 저장하면 에러가 해결됩니다.



### 1.3. Java 8u162 버전

Java 8u162버전은 기본적으로 **unlimited** 속성을 따르기 때문에 더이상 JRE에 정책 파일을 설치하거나 등록정보를 설정할 필요가 없습니다. 기본적으로 unlimited 설정을 사용하는 자바 버전은 [Enable unlimited cryptographic policy by default in Oracle JDK builds](https://bugs.openjdk.java.net/browse/JDK-8170157)에서 확인할 수 있습니다.



> **Tip: ** Java9은 unlimited 속성이 default이므로 수정할 필요가 없습니다.



## 2. Reference

- https://golb.hplar.ch/2017/10/JCE-policy-changes-in-Java-SE-8u151-and-8u152.html

