---
title: "Maven Artifact Search"
categories: IntelliJ
tags: [intellij, 인텔리, 인텔리제이]
published: true
comments: true
---



프로젝트를 구축하면서 우리는 Maven, Gradle과 같은 라이브러리 의존성 관리 툴을 사용하고 있습니다. 필자의 경우는 필요한 라이브러를 추가 할때 [Maven Repository](https://mvnrepository.com/) 에서 의존성 코드를 복사해서 붙여 넣는 방식으로 사용해 왔습니다. 지금 다룰 인텔리제이의 기능은 IDE에서 직접 Maven Repository를 검색 할 수 있는 기능을 제공하여 개발자들에게 편리성을 제공해 주는 기능입니다.



## 사용법

우선 저는 Gradle을 사용하기 때문에 Gradle 위주로 설명을 진행하도록 하겠습니다. Maven의 경우에도 사용법은 유사하기 때문에 별 어려움 없이 사용하실 수 있습니다. (macOS 기준)



1. `build.gradle` 파일 열기

2. `Control` + `Enter` 단축키 입력

3. Add maven artifact dependency 선택

   ![](/images/2018/0626_01_01.png)

4. 라이브러리 검색

   ![](/images/2018/0626_01_02.png)

보신것과 같이 사용법은 굉장히 심플하지만 개발자들에게 높은 편의성을 제공해주는 인텔리제이의 기능입니다. 이 기능을 사용하면 **라이브러리 명**으로 검색할 수 있을 뿐만 아니라, **클래스 명** 또한 검색할 수 있기 때문에  `FileUtils` 와 같은 유틸성 라이브러리를 쉽고 간편하게 검색할 수 있습니다.