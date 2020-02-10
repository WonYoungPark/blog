---
title: "GitLab CI/CD 구축"
categories: DevOps
tags: [GitLab, CICD]
published: true
comments: true
---

> **Note**
>
> 해당 포스팅은 GitLab CI/CD 구축에 대해서만 다루고 있습니다. GitLab 설치 및 설정, Docker Engine 설치 관련 내용은 다루지 않습니다.


# 우리 개발팀의 기술 스택

저희 개발팀은 다음과 같은 기술스택을 갖고 있습니다.

| 구분          | 비고                  |
| ------------- | --------------------- |
| 언어          | JAVA 11+              |
| 프레임워크    | Spring Boot, Vert.x   |
| 의존성 관리   | Gradle                |
| 컨테이너 기술 | Docker                |
| Git 저장소    | GitLab (Slef-Hosting) |

위의 기술 스택을 전제로 GitLab CI/CD 환경을 구축하는 내용을 다루도록 하겠습니다. 

> 해당 포스팅은 Docker가 설치되어 있다는 전제로 진행됩니다. Docker가 설치되지 않으신 분들은 [해당 문서를 참고하십시오.](https://docs.docker.com/v17.12/install/#supported-platforms)



# CI/CD 환경이 왜 필요한가?

CI/CD 환경은 개발 생산성과 배포 안전성을 위해서 개발팀에서 필수적으로 구축해야하는 환경 중 하나입니다. 개발조직은 다음과 같은 상황을 대비하고 장애상황에서 신속하게 대처하기 위해 CI/CD 환경을 구축해야 합니다.

## 상황 1. 보안정책으로 인해 허가된 인원 이외에 운영서버에 접근이 불가능할 경우

저희 회사는 보안을 위해 허가된 인원 이외에는 운영 서버에 접근할 수 없도록 정책이 수립되어 있습니다. CI/CD가 구축되어 있지 않는 환경이라면 운영 서버에 허가된 인원이 공석인 상황에서 장애가 발생할 경우 신속한 대처가 불가능하게 됩니다.

## 상황 2. 잦은 배포를 하는 개발조직

`개발환경, 스테이징환경, 운영환경` 등 다수의 환경을 구축한 개발조직에서는 배포작업이 빈번하게 이루어집니다. 한번의 배포 작업만 볼 경우 배포에 소비되는 시간이 짧다고 느껴질 수 있으나, 장기적인 관점으로 볼때 불필요한 큰 손실입니다.

## 상황 3. 배포 실수

익숙하고 반복적인 배포 작업이더라도 사람은 종종 실수를 하곤합니다. 환경변수를 누락하거나, 최신버전이 아닌 이전 버전을 배포하는 등의 작은 실수를 사람은 종종하게 됩니다. 이런 작고 사소한 실수들이 실제 운영환경에서는 큰 장애로 발생할 수 있습니다.

## 상황 4. 배포 주기가 긴 레거시 프로젝트

사내에서만 사용하는 레거시 프로젝트의 경우 배포 주기가 굉장히 길 확률이 높습니다. 반년, 일년 주기로 hotfix만 처리하여 배포를 진행할 경우가 발생하게되면 개발자는 해당 프로젝트의 배포 프로세스를 작성해둔 문서를 부랴부랴 찾은 후에 배포를 진행하곤 합니다.



간단하게 몇 가지 사례를 작성해 보았습니다. 대부분의 상황에 해당하시지 않으신가요? 이미 많은 개발조직이 위의 사례와 비슷한 경우를 대비하기 위해 CI/CD 환경을 구축하고있습니다.



# 이런 개발조직에게 추천드립니다.

GitLab으로 CI/CD 환경을 구축하기 좋은 개발조직을 정리해 보았습니다.

- 형상관리를 Git으로 하고있는 개발조직
- 무료 Git 저장소를 찾는 개발조직 (Self Hosting, [가격 참조](https://about.gitlab.com/pricing/#self-managed))
- Jenkins, Docker Registry, Git Repository... 너무 많은 툴로 인해 고통받고 있는 개발조직 (툴 일원화)
- 컨테이너 오케스트레이션을 사용하지 않아도 되는 규모의 개발조직 (GitLab에서도 쿠버네티스와 연동을 할 수 있지만 내용을 다루지는 않습니다.)
- **쉽고 빠르게 CI/CD 환경을 구축하고 싶은 개발조직**



GitLab은 Git 저장소, 이슈 트래커, 위키 등의 기능을 제공할 뿐만아니라 DevOps 기능 등 다양한 기능을 제공하고 있습니다. 단일 애플리케이션으로 형상관리 부터 자동 배포 환경 까지 다양한 기능을 제공합니다. 쉽고 빠르게 CI/CD 환경을 구축하고 관리 툴을 일원화 하기를 원하는 개발조직해서 사용하시면 만족스러운 결과를 느끼실 수 있습니다.



# 시스템 구성도

GitLab CI/CD 환경 구성을 이해하기 쉽도록 다음과 같이 크게 세가지(`GitLab, CI Server, Application Server`)로 구분 짓도록 하겠습니다.

![GitLab_CI_CD_181203-시스템_구성도_1](/images/2018/1203_01_01.jpg)

- GitLab: 위키와 이슈 추적등 다양한 기능을 갖춘 웹 기반의 Git 저장소 관리자이며, CI/CD 기능을 제공합니다.
    - Repository: Git 저장소
    - GitLab Container Registry: GitLab에 통합 된 Docker Container Registry 입니다. 모든 프로젝트에서 Docker 이미지 저장공간을 사용할 수 있습니다. (Docker Registry에 대한 자세한 내용은 [Docker Registry 소개 문서를 참조하시기 바랍니다.](https://docs.docker.com/registry/introduction))
- CI Server: 빌드 전용 서버입니다. 빌드 작업은 서버 리소스를 많이 사용하므로 서버를 분리하여 구축하도록 합니다. (GitLab과 같은 서버에 구성할 수도 있으나 추천드리지 않습니다.)
    - GitLab Runner (Integration): 빌드를 수행하는 GitLab Runner입니다.
- Application Servers: 서비스하는 애플리케이션이 구동되는 `물리적 or 논리적` 서버
    - GitLab Runner (Deploy): 배포를 수행하는 GitLab Runner입니다.
    - Application: 서비스 애플리케이션



# CI/CD 구축 Hands-On

## Spring Boot Demo 애플리케이션 생성

[Spring Boot](https://spring.io/projects/spring-boot) 애플리케이션을 GitLab CI를 통해 배포하는 방법을 다루어보도록 하겠습니다. 데모 프로젝트를 생성하기 위해 [Spring Initializr](https://start.spring.io/)를 이용하여 프로젝트를 생성합니다.

```bash
# Demo Spring Boot Source 다운로드
curl https://start.spring.io/starter.tgz \
    -d type=gradle-project \
    -d language=java \
    -d bootVersion=2.1.1.RELEASE \
    -d baseDir=demo \
    -d groupId=com.example \
    -d artifactId=demo \
    -d name=demo \
    -d description=Demo+project+for+Spring+Boot \
    -d packageName=com.example.demo \
    -d packaging=jar \
    -d javaVersion=11 \
    -d style=web \
    | tar -xzvf -

# Git 설정 및 push
cd demo
git init
git remote add origin [GIT_REPOSITORY_URL]
git add .
git commit -m "Initial commit"
git push -u origin master
```



## GitLab Runner 구성

GitLab Runner는 `.gitlab-ci.yml` 에 정의한 작업을 실행하는 응용프로그램입니다. GitLab Runner는 Go 언어로 개발되어있기 때문에 가상 머신, VPS, 도커 컨테이너 등 크로스플랫폼을 지원합니다. GitLab Runner를 설치하는 방법은 다양합니다. 바이너리를 다운로드하여 설치하거나 패키지 저장소를 이용하여 설치하거나 Docker를 사용하는 방법 등이 있습니다. 우리는 도커에서 GitLab Runner 컨테이너를 구성하는 방식으로 진행하도록 하겠습니다.

> GitLab Runner에 대한 자세한 사항은 [해당 문서를 참조하십시오.](https://gitlab.mrblue.com/help/ci/runners/README.md)



### GitLab Runner 역할구분

![ci-cd-test-deploy](/images/2018/1203_01_02.png)

GitLab Runner는 크게 `빌드를 수행하는 GitLab Runner` 와 ` 배포를 담당하는 GitLab Runner`로 구분지을 수 있습니다.

- 빌드를 수행하는 GitLab Runner: 빌드만을 수행하는 GitLab Runner입니다. 해당 GitLab Runner는 모든 프로젝트에서 공통적으로 사용해야 하므로 Shared GitLab Runner로 등록해야 합니다. Shared GitLab Runner에 대해서는 아래에서 설명하도록 하겠습니다.
- 배포를 담당하는 GitLab Runner: 배포를 수행하는 GitLab Runner입니다. 해당 GitLab Runner는 배포 할 서버에 GitLab Runner를 설치하여 Specific GitLab Runner로 등록해야 합니다.



### 빌드 전용 GitLab Runner 구성

GitLab Runner는 세가지의 타입(`specific, group, shared`) 으로 구성할 수 있습니다. GitLab Runner는 GitLab에서 특정 프로젝트에만 등록되거나, 여러 혹은 모든 프로젝트에 등록될 수 있습니다. 모든 프로젝트에 등록되는 GitLab Runner를 Shared Runner라고 합니다.

[시스템 구성도](#시스템 구성도)에서 설명한 CI Server의 `GitLab Runner (Integration)`를 구성해보도록 하겠습니다. 해당 GitLab Runner는 CI를 수행하는 Agent이기 때문에 모든 프로젝트에 등록되는 Shared Runner로 구성하도록 하겠습니다.



### Shared Runner 등록 화면으로 이동

GitLab 관리자 계정으로 로그인하여 `Admin Area -> Runners` 화면으로 이동하면 아래와 같은 화면을 확인하실 수 있습니다. GitLab에서 친절하게 Shared Runner을 구성할때 필요한 `URL`, `Token` 데이터를 제공하고 있습니다. 
![Admin_Area___GitLab_2018-12-05_10-15-15](/images/2018/1203_01_03.png)


### 빌드 서버에 Shared Runner 구성

Shared Runner 등록 화면에서 제공해준 `URL`, `Token` 데이터를 가지고 GitLab Runner를 Docker 컨테이너로 구성해 보도록 하겠습니다.

```bash
# 컨네이너 생성
docker run --detach \
    --name gitlab-runner \
    --restart always \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner

# GitLab Runner를 GitLab에 등록
docker exec -it gitlab-runner gitlab-runner register \
    --non-interactive \
    --url https://gitlab.mrblue.com \
    --registration-token [PROJECT_REGISTRATION_TOKEN] \
    --executor docker \
    --description "빌드 서버" \
    --tag-list DevLab-CI \
    --docker-image "docker:latest" \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock
```

GitLab Runner register command 속성의 의미는 다음과 같습니다.

| Command            | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| non-interactive    | 비 대화식 설정을 정의합니다. (Default: interactive)          |
| url                | GitLab Runner가 등록될 GitLab의 URL를 정의합니다.            |
| registration-token | GitLab에서 제공하는 `registration token` 을 등록합니다.      |
| executor           | GitLab Runner는 다양한 Executor를 정의할 수 있습니다. 우리는 Docker 기반으로 환경을 구성할 것이기 때문에 docker로 정의합니다. 자세한 사항은 [해당 문서를 참조하십시오.](https://docs.gitlab.com/runner/executors/README.html) |
| description        | GitLab Runner에 대한 설명을 정의합니다.                      |
| tag-list           | 태그는 GitLab에서 GitLab Runner에게 요청을 보낼 때 사용되는 구분 값입니다. |
| docker-image       | Docker Executor가 실행될때 사용하는 기반 도커 이미지를 정의합니다. |
| docker-volumes     | Docker socket 바인딩 방식의 Docker in Docker 기술을 사용하기 위해 정의합니다. 자세한 사항은  [해당 문서를 참조하십시오.](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding) |

### Shared Runner 등록 확인

Shared Runner가 정상적으로 등록되면 다음과 같이 확인할 수 있습니다.
![Image_2018-12-05_11-15-15](/images/2018/1203_01_04.png)


## 프로젝트별 배포용 GitLab Runner 구성

`Setting -> CI/CD` 화면으로 이동하면 다음과 같은 화면을 확인하실 수 있습니다.
![CI___CD_Settings___CI___CD___One.0___cicd-demo___GitLab_2018-12-05_11-51-52](/images/2018/1203_01_05.png)


### 애플리케이션 서버에 Specific Runner 구성

Shared Runner를 등록했던 방법과 같은 방법으로 애플리케이션 서버에 GitLab Runner을 구성하도록 하겠습니다. Shared Runner를 등록했던과 방법은 같지만 정의 값이 달라졌습니다. 특히 `tag-list` 에서 선언한 `tag` 의 경우 GitLab Runner를 식별하는 용도로 사용되기 때문에 GitLab Runner의 용도에 따라 구별가능한 `tag` 를 정의해 주셔야 합니다.

```bash
# 컨네이너 생성
docker run --detach \
    --name gitlab-runner \
    --restart always \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    gitlab/gitlab-runner

# GitLab Runner를 GitLab에 등록
docker exec -it gitlab-runner gitlab-runner register \
    --non-interactive \
    --url https://gitlab.mrblue.com \
    --registration-token [PROJECT_REGISTRATION_TOKEN] \
    --executor docker \
    --description "Demo 서버" \
    --tag-list DevLab-Demo \
    --docker-image "docker:latest" \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock
```



> GitLab Runner 구성에 대한 자세항 사항은 [해당 문서를 참조하십시오.](https://gitlab.mrblue.com/help/ci/runners/README.md#registering-a-specific-runner)

> Shared Runner는 GitLab의 관리자인 경우에만 등록할 수 있습니다.





## Spring Boot 애플리케이션 도커라이징

Docker 컨테이너를 패키징 하는것을 도커라이징(Dockerizing) 라고 합니다. 생성한 Demo Spring Boot 애플리케이션을 Docker 컨테이너로 패키징하도록 하겠습니다. 도커라이징을 하기 위해선 `Dockerfile` 을 루트 디렉토리에 만들어야 합니다.

```dockerfile
FROM centos:7

MAINTAINER wyparks2 <wyparks2@mrblue.com>

ENV JDK_VERSION 11.0.1
ENV JDK_DIR jdk-${JDK_VERSION}
ENV JDK_FILE jdk-${JDK_VERSION}.tar.gz
ENV JAVA_HOME /usr/local/${JDK_DIR}

# Install Packages
RUN yum update -y && \
    yum install -y wget && \
    yum clean all

# JDK 설치
RUN cd /usr/local/src && \
    wget https://download.java.net/java/GA/jdk11/13/GPL/openjdk-11.0.1_linux-x64_bin.tar.gz -O ${JDK_FILE} && \
    tar xzf ${JDK_FILE} && \
    rm -f ${JDK_FILE} && \
    mv ${JDK_DIR} ${JAVA_HOME}

# JAVA_HOME PATH 등록
ENV PATH="$PATH:$JAVA_HOME/bin"

# 언어 설정
RUN localedef -f UTF-8 -i ko_KR ko_KR.UTF-8
ENV LC_ALL ko_KR.UTF-8
ENV LANGUAGE ko_KR.UTF-8

# 타임존 설정
ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo ${TZ} > /etc/timezone

ADD build/libs/*.jar app.jar

VOLUME ["/tmp"]

EXPOSE 8080

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

이해를 돕기 위해 위의 코드를 분리하여 설명하도록 하겠습니다.



### 기본 도커 이미지 정의

`FROM` 키워드는 도커의 기본 이미지를 정의합니다. 도커의 기본 이지미로 `centos 7` 버전을 정의합니다. 경량 Linux 배포판인 [Alpine Linux](https://alpinelinux.org/) 를 사용할수도 있으나, 저희 개발팀은 내부적으로 `centos 7` 버전으로 사용하도록 정책을 수립하였습니다.

```dockerfile
FROM centos:7
```



### 생성자 정의

`MAINTAINER` 키워드는 Dockerfile 생성자 정보를 정의합니다.

```dockerfile
MAINTAINER wyparks2 <wyparks2@mrblue.com>
```



### 환경변수 정의

`Dockerfile` 에서 사용할 변수를 정의합니다.

```dockerfile
ENV JDK_VERSION 11.0.1
ENV JDK_DIR jdk-${JDK_VERSION}
ENV JDK_FILE jdk-${JDK_VERSION}.tar.gz
ENV JAVA_HOME /usr/local/${JDK_DIR}
```



### Package 설치

패키지 업데이트를 진행하고 필요한 패키지를 추가 설치합니다.

```dockerfile
# Install Packages
RUN yum update -y && \
    yum install -y wget && \
    yum clean all
```



### JDK 바이너리 다운로드 및 설정

OpenJDK 버전 11 바이너리를 다운로드 하고 압축해제합니다.

```dockerfile
RUN cd /usr/local/src && \
    wget https://download.java.net/java/GA/jdk11/13/GPL/openjdk-11.0.1_linux-x64_bin.tar.gz -O ${JDK_FILE} && \
    tar xzf ${JDK_FILE} && \
    rm -f ${JDK_FILE} && \
    mv ${JDK_DIR} ${JAVA_HOME}
```



### 환경변수 등록

환경변수 `JAVA_HOME`을 정의합니다.

```dockerfile
ENV PATH="$PATH:$JAVA_HOME/bin"
```



### 타임존 설정

서버 시간대를 한국표준 시간대로 정의합니다.

```dockerfile
ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && \
    echo ${TZ} > /etc/timezone
```



### ADD

`ADD` 키워드는 파일을 이미지에 추가할 때 정의합니다.  `Gradle Build`가 정상적으로 수행될 경우 모든 출력 파일은 `build` 디렉토리 하위에 생성되며, JAR 파일은 `build/libs` 디렉토리 하위에 생성됩니다. Docker 이미지에 생성된 JAR 파일을 추가합니다.

```dockerfile
ADD build/libs/*.jar app.jar
```



### VOLUME

`VOLUME` 키워드는 디렉토리를 컨테이너 내부가 아닌 Host에 저장해야 할때 정의합니다. 단, `VOLUME` 을 정의하였더라도 Host의 특정 디렉토리와 연결되지는 않습니다. Host의 특정 디렉토리와 연결하기 위해서는 `docker run` 명령 시 `-v ` 옵션으로 연결해야 합니다.

```dockerfile
VOLUME ["/tmp"]
```



### EXPOSE

`ExPOSE` 키워드는 컨테이너가 호스트와 연결할 포트를 정의합니다. 단,  `EXPOSE` 을 정의하였더라도 Host와 연결이 되었을 뿐 외부에 포트가 공개되지는 않습니다. 외부에 포트를 공개하기 위해서는 `docker run` 명령 시 `-p` 옵션으로 공개해야 합니다.

```dockerfile
EXPOSE 8080
```



### ENTRYPOINT

`ENTRYPOINT` 명령어는 컨테이너가 실행되었을때 수행하는 명령을 정의합니다.

```dockerfile
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```



생성한 `Dockerfile` 또한 Git 저장소에 Push 합니다.

```bash
git add Dockerfile
git commit -m "Add Dockerfile"
git push origin master
```



> `Dockerfile` 에 대한 자세한 사항은 [해당 문서를 참조하십시오.](https://docs.docker.com/engine/reference/builder/#expose)



## GitLab CI Pipeline 만들기

Git 저장소 루트 디렉토리에 GitLab CI/CD config 파일(`.gitlab.yml`)을 추가해야 합니다. 이 파일은 GitLab Runner가 프로젝트의 빌드 및 배포를 관리하는데 사용됩니다. `.gitlab.yml` 파일을 저장소의 루트 디렉토리에 추가하면 GitLab은 코드를 자동으로 감지하고 GitLab에 새로운 코드가 Push 될때 마다 정의 된 단계를 실행합니다.

```yaml
image: docker:latest

stages:
  - build
  - package
  - deploy

gradle-build:
  stage: build
  tags:
    - DevLab-CI
  image: gradle:4.10.2-jdk11
  variables:
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  before_script:
    - export GRADLE_USER_HOME=`pwd`/.gradle
  script:
    - gradle build
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
      - .gradle
  artifacts:
    paths:
      - build/libs/*.jar
  only:
    - master

docker-build:
  stage: package
  tags:
    - DevLab-CI
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID .
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID
  when: on_success
  only:
    - master
deploy-devel:
  stage: deploy
  tags:
    - DevLab-Demo
  script:
    - ID=$(docker ps -a --filter="name=devel-$CI_PROJECT_NAME" -q) && [[ -n $ID ]] && docker stop $ID && docker rm $ID
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
    - docker run --detach --name devel-$CI_PROJECT_NAME --restart always -v /tmp:/tmp --publish 8080:8080 -e "SPRING_PROFILES_ACTIVE=devel" $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID
  environment:
    name: devel
    url: http://127.0.0.1:8080
  when: on_success
  only:
    - master
```

이해를 돕기 위해 위의 코드를 분리하여 설명하도록 하겠습니다.

### Image

```yaml
image: docker:latest
```

`image` 키워드는 우리가 사용하고자 하는 도커 이미지의 이름을 정의합니다. 유효한 이미지는 Docker Engine 또는 [Docker Hub]() 에서 호스팅되는 이미지입니다. `.gitlab-ci.yml` 파일에  `image` keyword를 정의하지 않으시면 GitLab Runner를 등록할때 `--docker-image` 로 정의한 이미지가 적용됩니다.

### Stages

```yaml
stages:
  - build
  - package
  - deploy
```

`stages` 키워드는 빌드의 생명주기를 정의힙니다. 또한 stage는 Job의 묶음입니다. stage내의 모든 Job은 병렬로 실행되고, stage는 정의된 순서대로 순차적으로 수행됩니다.

### gradle-build Job

```yaml
gradle-build:
  stage: build
  tags:
    - DevLab-CI
  image: gradle:4.10.2-jdk11
  variables:
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  before_script:
    - export GRADLE_USER_HOME=`pwd`/.gradle
  script:
    - gradle build
  artifacts:
    paths:
      - build/libs/*.jar
  only:
    - master
```

위의 코드는 Job을 정의한 것입니다. 해당 Job은 Gradle 빌드를 수행합니다. Gradle 빌드를 도커 컨테이너를 생성해서 진행하는 이유는 프로젝트마다 의존성 관리 툴과 사용하는 언어 또는 언어의 버전이 다르기 때문입니다.

`artifacts` 키워드는 실행 가능한 JAR 파일을 유지하고, 여러 Job에 결과물을 공유하기 위해 정의힙니다. 빌드가 성공하게 되면 파이프 라인의 화면 UI에서 다운로드도 할 수 있습니다.


![artifacts](/images/2018/1203_01_06.png)

> job artifacts의 default 만료기간은 `30일` 입니다. 자세한 내용은 [해당 문서를 참조하십시오.](https://docs.gitlab.com/ee/user/admin_area/settings/continuous_integration.html#default-artifacts-expiration-core-only)



### docker-build Job

```yaml
docker-build:
  stage: package
  # 해당 tag가 지정된 GitLab Runner들에게 작업을 요청
  tags:
    - DevLab-CI
  script:
  	# Dockerizing
    - docker build -t $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID .
    # Docker Registry 인증 (Docker Registry에 이미지를 Push 하기 위해서 반드시 수행해야 합니다.)
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
    # 생성된 Docker Image를 Docker Registry로 Push
    - docker push $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID
  # package stage에 포함된 이전 Job이 성공할 경우에만 해당 Job을 수행  
  when: on_success
  # Master 브랜치일 경우에만 해당 Job을 수행
  only:
    - master
```

해당 Job은 애플리케이션을 Dockerizing을 정의한 것입니다. `script` 키워드는 GitLab Runner에 의해 실행되는 쉘 스크립트를 정의할 수 있습니다. 해당 `script` 키워드는 Dockerizing을 진행하고, GitLab Container Registry에 로그인을 한 다음 이미지를 Push합니다.

`CI_REGISTRY_IMAGE, CI_PIPELINE_ID, CI_REGISTRY, CI_BUILD_TOKEN ` 변수는 GitLab CI가 빌드환경에 자동으로 주입하는 **미리 정의 된 변수** 입니다.

> 사전 정의 된 변수에 대한 자세한 사항은 [해당 문서를 참조하십시오.](https://docs.gitlab.com/ee/ci/variables/README.html#variables)



### deploy-devel

`devel` 환경에 배포를 수행하도록 정의한 Job입니다.

```yaml
deploy-devel:
  stage: deploy
  # 해당 tag가 지정된 GitLab Runner들에게 작업을 요청
  tags:
    - DevLab-Demo
  script:
    # devel-* 로 시작하는 도커 컨테이너가 있다면 멈추고 제거
    - ID=$(docker ps -a --filter="name=devel-$CI_PROJECT_NAME" -q) && [[ -n $ID ]] && docker stop $ID && docker rm $ID
    # Docker Registry 인증 (Docker Registry에 이미지를 Pull 하기 위해서 반드시 수행해야 합니다.)
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN $CI_REGISTRY
    # Docker 컨테이너 실행
    - docker run --detach --name devel-$CI_PROJECT_NAME --restart always -v /tmp:/tmp --publish 8080:8080 -e "SPRING_PROFILES_ACTIVE=devel" $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID
  environment:
    name: devel
    url: http://127.0.0.1:8080
  # deploy stage에 포함된 이전 Job이 성공할 경우에만 해당 Job을 수행
  when: on_success
  # Master 브랜치일 경우에만 해당 Job을 수행
  only:
    - master
```



# 파이프라인 상태 보기

포스팅 내용을 순차적으로 진행하셨다면 GitLab 으로 CI/CD 환경 구축이 완성되셨을 겁니다. GitLab 에서는 Pipeline 화면을 통해 이전에 수행했던 모든 Pipeline의 결과를 확인할 수 있습니다. 

![pipeline-list](/images/2018/1203_01_07.png)

## 파이프라인 도식화

`CI/CD -> Pipelines` 화면에서 Pipeline의 `Status` 를 선택하면 다음과 같은 Pipeline을 시각적으로 도식화하여  배포 프로세스를 이해하기 쉽도록 UI를 제공해 줍니다.

![pipeline-ui](/images/2018/1203_01_08.png)

## Job Log 확인

작업을 클릭하면 해당 작업의 로그를 확인할 수 있습니다. 해당 Job이 성공 또는 실패한 원인을 판단하는 중요한 판단 기준이 됩니다.

![Job_Log](/images/2018/1203_01_09.png)


# 정리

GitLab을 이용하여 빠르고 간단하게 CI/CD 환경을 구축해 보았습니다. 이번 포스팅은 **쉽고 빠르게 CI/CD 환경을 구축**하는 방법을 설명하는 주제이기 때문에 생략하고 넘어간 GitLab의 기능과 Docker의 기능들이 많습니다. (해당 기능들은 최대한 문서에 참조 링크를 함께 남겨놓았으나 아쉬운 부분이 많습니다.)

컨테이너 기술에 익숙하지 않은 분들에게는 포스팅의 주제처럼 쉽지만은 않으셨을 수도 있을거 같습니다. 하지만 최근에는 많은 개발조직에서 컨테이너 기술을 기반으로 개발, 스테이징, 운영 환경을 구축을 하기 때문에 개발자들이 필수적으로 익혀야 하는 기술 중 하나인거 같습니다.

해당 포스팅을 통해 아직 CI/CD 환경을 구축하지 못하셨거나 관리 툴의 일원화를 원하시는 개발조직에게 도움이 되셨으면 합니다.

감사합니다.

# Reference

- https://about.gitlab.com/2016/12/14/continuous-delivery-of-a-spring-boot-application-with-gitlab-ci-and-kubernetes/
- https://docs.gitlab.com/ee/ci/yaml/README.html





