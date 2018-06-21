---
title: "X-Frame-Options 이란?"
categories: ETC
tags: "X-Frame-Options"
published: true
comments: true
---



이번에 사내 타팀에서 서비스를 런칭하게 되었는데 일정이 빠듯하여 저희팀이 지원을 해주게 되었습니다. 지원을 하면서 부하/스트레스 테스트와 모의해킹 부분을 지원해 줄 수 있도록 저희 팀장님께서 저에게 요청을 하셨습니다. 모의해킹을 하면서 개발적으로 도움이 많이 될 것이라고 팀장님께서 알려주셨고, 실제로 모의해킹을 진행하면서 모르던 부분들을 많이 알게되었습니다. 그 중 X-Frame-Options 라는 Http Header에 대해 알아보도록 하겠습니다.



## X-Frame-Options

X-Frame-Options를 알기 위해서는 선행되야할 것이 있습니다. 바로 [클릭재킹(Clickjacking)](https://ko.wikipedia.org/wiki/%ED%81%B4%EB%A6%AD%EC%9E%AC%ED%82%B9) 공격기법입니다. 눈치가 빠르신 분들이라면 이미 아셨겠지만 X-Frame-Options 는 클릭재킹(Clickjacking) 공격기법을 방어하기 위해서 2009년도에 IE8에 추가되었고 다른 브라우저에서도 지금까지 적용되고 있습니다.

클릭재킹은 frame, iframe 를 이용해서 사용자가 인지하지 못하는 투명한 레이어를 추가하여 악성 링크로 요청을 보내도록하는 수법을 말합니다.

이러한 클릭재킹 공격기법을 방어하기 위해서 브라우저에서는 Http Header에 X-Frame-Options이 있을 경우 속성에 따라 frame에 표시할 수 있는 리소스를 제한합니다.



### 속성

X-Frame-Options 속성은 다음과 같습니다.

```http
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
X-Frame-Options: ALLOW-FROM https://example.com/
```

- DENY: 페이지는 시도하는 사이트에 관계없이 프레임에 표시 할 수 없습니다.
- SAMEORIGIN: 페이지는 페이지 자체와 동일한 출처의 프레임에만 표시 될 수 있습니다
- ALLOW-FROM: 페이지는 지정된 도메인을 가지는 프레임에만 표시 될 수 있습니다.