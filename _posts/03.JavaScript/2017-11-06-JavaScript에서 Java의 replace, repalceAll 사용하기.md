---
title: "JavaScript에서 Java의 replace, replaceAll 사용하기"
categories: JavaScript
tags: javascript
comments: true
---

# replace

javascript에서 replace() Method는 어떤 패턴에 일치하는 일부 또는 모든 부분이 교체된 새로운 문자열을 반환합니다. 그 패턴은 문자열이나 정규식(RegExp)이 될 수 있으며, 교체 문자열은 문자열이나 모든 매치에 대해서 호출된 함수일 수 있습니다.

## replace Method

javascript에서의 replace Method는 문자열 중에 **처음으로 찾은 문자** 만을 변환해 줍니다.(java의 replace 또는 replaceAll 처럼 **모든 문자열** 을 치환해주지 않습니다.)
```javascript
var date = '2016-08-23';
var str  = date.replace('-', '');

console.log(str);
/*
Result :
201608-23
*/
```
그렇다면 문자열 **전체** 를 변환하기 위해서는 어떻게 하면 좋을까요??
```javascript
var date = '2016-08-23';
var str  = date.replace(/-/g, '');

console.log(str);
/*
Result :
20160823
*/
```
위의 예제 코드와 같이 replace Method 인자값에 정규식을 입력하여 처리하면 된다.
> **Note:**
> - g(global) : 모든 문자를 검색
> - i(ignoreCase) : 대소문자 구분 안함
> - m : 여러 줄 검색
---
참고
---
> - [JavaScript | MDN][1]

[1]: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String/replace
