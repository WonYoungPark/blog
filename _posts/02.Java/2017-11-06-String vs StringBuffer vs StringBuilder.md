---
title: "String vs StringBuffer vs StringBuilder"
categories: Java
tags: java
published: true
comments: true
---

String 클래스는 잘못 사용하면 메모리에 많은 영향을 끼치기 때문에 String 클래스에 대해 알아보도록 하겠습니다.
자바기반의 프로그래밍에서 문자열을 만드는 클래스는 String, StringBuffer, StringBuilder가 가장 많이 사용된다. StringBuffer클래스와  StringBuilder 클래스는 제공하는 메서드는 동일하다. 그러면 둘의 차이점은 무엇일까요? StringBuffer 클래스는 **ThreadSafe** 하게 설계되어 있으므로, 여러 개의 스레드에서 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않는다. 반면에 StrinBuilder는 **단일 스레드** 에서만 안전성을 보장한다.

## String
```java
Stirng str = new String();

System.out.println(str += "abc"); // 1번째 문자열 추가
System.out.println(str += "def"); // 2번째 문자열 추가
System.out.println(str += "ghi"); // 3번째 문자열 추가

/*
Result :
abc
abcdef
abcdefghi
*/
```

### String 동작원리

|          | 1st  | 2nd  | 3rd  |
| :------: | :--: | :--: | :--: |
| 주소 = 100 | abc  |      |      |
| 주소 = 150 | abc  | def  |      |
| 주소 = 200 | abc  | def  | ghi  |

> **Note:**
> - String 객체에 문자열을 더하면 새로운 String객체가 생성된다.
> - 이전 String 객체는 GC대상이 되어진다.
---
## StringBuffer, StirngBuilder
```java
Stirng str = new StringBufer();

System.out.println(str.append("abc").toStirng); // 1번째 문자열 추가
System.out.println(str.append("def").toStirng); // 2번째 문자열 추가
System.out.println(str.append("ghi").toStirng); // 3번째 문자열 추가

/*
Result :
abc
abcdef
abcdefghi
*/
```

### StringBuffer, StringBuilder 동작원리
|          | 1st  | 2nd  | 3rd  |
| :------: | :--: | :--: | :--: |
| 주소 = 100 | abc  |      |      |
| 주소 = 100 | abc  | def  |      |
| 주소 = 100 | abc  | def  | ghi  |

> **Note:**
> - String 객체와는 다르게 새로운 객체를 생성하지 않는다.
> - 기존의 String객체의 크기를 증가시키면서 값을 더한다.
---

## String, StringBuffer, StirngBuilder 사용해야할 경우
그렇다면 String을 쓰는 것이 무조건 나쁜것이고, 무도건 StringBuffer 또는 StringBuilder만을 사용해야만 하는 것인가?? 간단하게 정리하면 아래와 같이 경우를 나눌수 있습니다.
- Stirng : 짧은 문자열을 더할 경우 사용
- StringBuffer : 멀티 쓰레드 환경에서 사용
- StringBuilder : 단일 쓰레드 환경에서 사용

> **Tip:** JDK 5.0 이상을 사용한다면 컴파일러에서 자동으로 Stinrg 객체를 StringBuilder로 변환해 준다. 하지만 반복 루프를 사용해서 문자열을 더할 떄는 객체를 게속 추가한다는 사실은 변함이 없으므로, String클래스를 쓰는 대신, StringBuffer(멀티쓰레드), StringBuilder(단일쓰레드)를 사용하도록 한다.

---
참고
---
> -	[도서 - 자바선능 튜닝 이야기](http://book.naver.com/bookdb/book_detail.nhn?bid=7333658)
