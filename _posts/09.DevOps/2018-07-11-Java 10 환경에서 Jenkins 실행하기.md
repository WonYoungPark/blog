---
title: "Java 10 환경에서 Jenkins 실행하기"
categories: DevOps
tags: [DevOps, Jenkis]
published: true
comments: true
---



최근에 Oracle에서 Java 유료화를 발표하면서 사내에서 OpenJDK로 개발환경을 변경하는 작업을 진행하고 있습니다. 현재 사내 개발서버에서는 다양한 애플리케이션이 가동 중인데 다행히도 모든 애플리케이션이 Java 10을 지원해 주고 있었습니다. 그 중에서 Jenkins를 Java 10 버전에도 실행시키는 간단한 방법을 정리해 보겠습니다.



[Jenkins 2.127](https://jenkins.io/changelog/#v2.127) 버전부터 Java 10 및 Java 11 버전을 지원해 주고 있습니다. 이는 안정화된 버전은 아니지만 사용하기에 무리가 없을 것으로 판단하여서 작업을 진행하였습니다.



## 설치 및 실행

1. 2.17 버전 이상의 [Jenkins WAR](http://mirrors.jenkins.io/war/) 파일을 다운로드 합니다.

2. 다음과 명령을 사용하여 WAR를 실행합니다.

   ```bash
   ${JAVA10_HOME}/bin/java --add-modules java.xml.bind -jar jenkins.war \
       --enable-future-java --httpPort=8080 --prefix=/jenkins
   ```

