---
title: "replace vs replaceAll"
categories: Java
tags: java
published: true
comments: true
---

Java에서 특정 문자열을 다른 문자열로 변환/치환하기 위해 사용하는 함수

```java
String str = "abcdefg";

System.out.println(str.replace("abc.", @));
System.out.println(str.replaceAll("abc.", @));

/*
Result :
abcdefg
@defg
*/
```

### String replace(char oldChar, char newChar)
**단일 문자** 치환
### String replace(CharSequence target, CharSequence replacement)
**문자열** 치환
### String replaceAll(String regex, String replacement)
**정규식 구조에 따른** 문자열을 치환

> **Note:** replaceAll의 경우, 정규식 표현상 사용되는 지정어가 포함된 경우, 오류가 발생할 수 있음.
