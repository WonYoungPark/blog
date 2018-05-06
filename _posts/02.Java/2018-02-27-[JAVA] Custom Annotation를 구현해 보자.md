---
title: "[JAVA] Custom Annotation를 구현해 보자"
categories: Java
tags: java annotation
published: true
comments: true
---



시스템에서 코드성 데이터의 경우 DB로 관리하는 경우가 대부분이지만, 자바 소스코드에서 Enum으로 정의하고 해당 Enum을 HTTP API 호출을 통해 조회해야 하는 요건이 생겼습니다. 클래스명에 규칙을 정해서 사용하는 방법과 Custom Annotation을 명시하여 클래스의 용도를 지정하는 방법 중에서 Custom Annotation으로 명시하는 것이 가독성과 코드 관리에 이점이 있을것으로 판단하여 Custom Annotation 방법으로 개발을 진행하였습니다. 개발 진행 중 정리해야 하는 부분이 필요하다고 판단하여 정리해 보도록 하겠습니다.



## 1. 사용방법

### 1.1. 선언 방법

다음과 같이 클래스, 필드, 메서드 등 다양한 부분에 선언을 할 수 있습니다.

```java
@CustomAnnotation
public class MyClass {
    @CustomId
    private Long Id;
    ...
}
```



### 1.2. 정의 방법

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({
    ElementType.TYPE,
    ElementType.FIELD
})
public @interface CustomAnnotation {
    /* enum 타입을 선언할 수 있습니다. */
    public enum Priority { LOW, MEDIUM, HIGH }
    String value();
    String[] changedBy() default "";
    String createdBy() default "James Gosling";
    String lastChanged() default "2011-07-08";
    Priority priority() default Priority.MEDIUM;
}
```



#### 1.2.1. Retention(용도)

| 이름    | 범위                                                         |
| ------- | ------------------------------------------------------------ |
| SOURCE  | 컴파일러가 사용하고 클래스 파일 안에 포함되지 않음(단순 주석용, 컴파일용) |
| CLASS   | 컴파일러가 클래스를 참조할 때까지만 유효                     |
| RUNTIME | 컴파일 이후에도 JVM에 의해서 참조가 가능                     |

#### 1.2.2. Target(범위)

선언되지 않은 범위에 Annotation을 명시하게 되면, 무시되거나 IDE에서 에러 메시지가 출력됩니다.

| 이름            | 범위                         |
| --------------- | ---------------------------- |
| TYPE            | 클래스, 인터페이스, 열거형   |
| FIELD           | 필드                         |
| METHOD          | 메서드                       |
| PARAMETER       | 파라미터                     |
| CONSTRUCTOR     | 생성자                       |
| LOCAL_VARIABLE  | 로컬 변수                    |
| ANNOTATION_TYPE | 애노테이션 타입              |
| PACKAGE         | 패키지                       |
| TYPE_PARAMETER  | 파라미터 타입 (JDK 1.8 추가) |
| TYPE_USE        | 타입 (JDK 1.8 추가)          |



### 2. Reference

- https://en.wikipedia.org/wiki/Java_annotation
- https://kang594.blog.me/39704853