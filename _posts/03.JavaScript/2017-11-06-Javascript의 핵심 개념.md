---
title: "javascript의 핵심 개념"
categories: JavaScript
tags: javascript
comments: true
---

## 객체

-	자바스크립트의 거의 모든것은 객체이다.
-	객체 제외인 것들
	-	기본 데이터 타입 : boolean, number, string
	-	특별한값 : null, undefined

> **Note:** 기본 데이터 타입은 모두 객체처럼 다룰수 도 있다.

---

## 함수

-	자바스크립트에서는 함수도 객체로 취급한다.
-	함수는 일급 객체이다.

---

## 프로토타입

-	모든 객체는 숨겨진 링크인 프로토타입을 가진다.
-	이 링크는 해당 객체를 생성한 생성자의 프로토타입 객체를 가리킨다.

---

## 실행 컨텍스트와 클로저

-	자바스크립트는 자신만의 독특한 과정으로 실행 컨텍스트를 만들고 그 안에서 실행이 이루어진다.
-	실행 컨텍스트는 자신만의 유효범위(Scope)를 갖는데, 이 과정을 클로저로 구현할 수 있다.

---

참고
----

> -	[도서 - 인사이즈 자바스크립트](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788968480652&orderClick=LAG&Kc=)
