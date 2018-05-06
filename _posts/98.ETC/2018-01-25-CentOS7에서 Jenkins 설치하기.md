---
title: "CentOS7에서 Jenkins 설치하기"
categories: ETC
tags: etc
published: false
comments: true
---



## JDK

### 설치가능 목록 확인

```bash
> yum list java*jdk-devel
```

### 설치

```bash
> yum -y install java-1.8.0-openjdk-devel.x86_64
```

#### 설치 확인

```bash
> java -version
```



## Git

### 설치

```bash
> yum -y install git
```

### 설치 확인

```bash
> git --version
```



## Wget

### 설치

```bash
> yum -y install wget
```



## Jenkins

### yum 레파지토리 설정

```bash
> wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
```

### 설치

```bash
> yum -y install jenkins
```

### 포트 설정

```bash
> sudo vi /etc/sysconfig/jenkins

//JENKINS_PORT와 JENKINS_AJP_PORT를 변경
JENKINS_PORT="9090" 
JENKINS_AJP_PORT="9091"
```

### 방화벽 설정

```bash
# sudo firewall-cmd --permanent --zone=public --add-port=[PORT]/tcp
> sudo firewall-cmd --permanent --zone=public --add-port=9090/tcp
> sudo firewall-cmd --reload
```

### 시작

```bash
> systemctl start jenkins
```

### 부팅 시 자동실행

```bash
> systemctl enable jenkins
```



## Docker

### yum-uils 설치

yum-utils는 패키지 관리자 yum에 추가적인 기능과 보완할 수 있는 유틸리티들을 모아 놓은 컬렉션입니다. 이 yum-utils를 이용해 레포지토리 추가를 손쉽게 하는 기능을 추가할 수 있습니다.

```bash
> yum -y install yum-utils
```

### 저장소 추가

`yum install docker` 로도 Docker를 설치할 수 있지만 최신 버전을 설치하기 위해서는 Docker에서 배포하는 저장소를 추가해야 합니다. 안정적인 버전이 필요하시면 저장소를 추가하지 마시고, `yum install docker` 를 이용해 설치하세요.

```bash
> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 설치

```bash
> yum -y install docker-ce
```

### 실행

```bash
> systemctl start docker
```

#### 부팅 시 자동실행

```bash
> systemctl enable docker
```

### 도커 그룹에 젠킨스 추가

jenkins 사용자를 docker 그룹에 추가하여, 권한문제를 해결할 수 있습니다.

```bash
# usermod -a -G docker [USER]
> usermod -a -G docker jenkins
```



```
{
  "insecure-registries" : ["myregistrydomain.com:5000"]
}
```





### Reference

- [CentOS 7 에 Jenkins CI 설치하기](http://hreeman.tistory.com/m/120)
- https://youngbin.xyz/blog/2015/11/16/install-and-configure-docker-on-ubuntu.html
- http://seujeum.tistory.com/12
- http://hreeman.tistory.com/category/?page=3
- http://www.kwangsiklee.com/ko/2017/05/%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95-solving-docker-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket/
- http://www.kwangsiklee.com/ko/2017/07/centos%EC%97%90%EC%84%9C-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/
- https://devnoff.wordpress.com/2016/01/23/%EC%9A%B0%EB%B6%84%ED%88%AC-%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%82%AC%EC%9A%A9%EC%9E%90%EB%A5%BC-%EA%B7%B8%EB%A3%B9%EC%97%90-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0/
- http://www.kwangsiklee.com/ko/2017/05/%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%95-solving-docker-permission-denied-while-trying-to-connect-to-the-docker-daemon-socket/