---
title: "CharSequence이란?"
categories: Java
tags: java
comments: true
---

CharSequence는 **인터페이스** 이다. 다시 말하면, 클래스가 아니기 때문에 이 인스턴스로 객체를 생성할 수 없다. 이 인터페이스를 구현한 클래스로는 CharBuffer, String, StirngBuffer, StringBuilder가 있으며, StringBuffer나 StirngBuilder로 생성한 객체를 전달할 때 사용된다.

```java
public class StringBufferTest{
    public static void main(Stirng args[]){
        StirngBuffer sb = new StirngBuffer();
        sb.append("ABCD");

        StirngBufferTest sbt = new StringBufferTest();
        sbt.check(sb);
    }

    public void check(CharSequence cs){
        StirngBuffer sb = new StringBuffer(cs);
        System.out.println("sb.length =" + sb.length());
    }
}

/*
Result
sb.length = 4
*/
```

> **Note:** StringBuffer나 StringBuilder로 값을 만든 후 굳이 toStirng()을 수행하여 필요 없는 객체를 만들어서 넘겨주기 보다는 CharSequence로 받아서 처리하는것이 **메모리 효율에 더 좋다.**

---

# 참고
> -	[도서 - 자바선능 튜닝 이야기](http://book.naver.com/bookdb/book_detail.nhn?bid=7333658)
