---
title: "CentOS7에 HAProxy 설치하기"
categories: ETC
tags: etc
published: true
comments: true
---



Haproxy는 L4, L7 과 같은 하드웨어 로드밸런서를 대체하기 위한 오픈소스 소프트에어로 이름처럼 Reverse Proxy 를 기반으로 로드밸런싱을 하기에 매우 강력하고 또 가벼운 어플리케이션입니다. HA (High Availability) 라는 이름처럼 고가용성을 위하여 설계되었습니다.

### 1. 설치

yum(패키지) 관리자를 통한 설치 방법과 소스코드 컴파일을 이용한 설치방법 두가지가 존재합니다.

```bash
> yum -y update
> yum info haproxy
```

현재 시점(2018.01.31)일 기준으로 yum에 배포된 haproxy 버전은 1.5.18 버전이다. HTTP/2를 사용하기위해선 [haproxy 1.8 버전](https://www.haproxy.com/blog/whats-new-haproxy-1-8/) 이상을 사용해야 하기 때문에 소스코드 컴파일 방식으로 설치를 진행해 보겠습니다.



#### 1.1. 의존 패키지 설치

```bash
> sudo yum -y install wget gcc pcre-static pcre-devel
```



#### 1.2. 다운로드

HAProxy의 최신 버전은 http://www.haproxy.org에서 다운로드 받을 수 있습니다.

```bash
> wget https://www.haproxy.org/download/1.8/src/haproxy-1.8.3.tar.gz ~/haproxy-1.8.3.tar.gz
```

> **Note:** 현재 기준(2018.01.31) HAProxy 1.8.3버전이 안정화 버전 입니다. 



#### 1.3. 압축 해제

```bash
> tar xzvf ~/haproxy-1.8.3.tar.gz -C ~/
```



#### 1.4. 컴파일

압축을 해제 한 후 make를 진행해야 하는데 **주의할 점**은 make를 진행하기 전에 **HAProxy의 README를 필수적으로 확인**해 보고 진행해야 합니다. 중요한 부분은 make TARGET을 지정하는 부분입니다.

```bash
# 여기서 TARGET 옵션은 리눅스 종류에 맞춰 지정합니다.
- linux22     for Linux 2.2
- linux24     for Linux 2.4 and above (default)  
- linux24e    for Linux 2.4 with support for a working epoll (> 0.21)  
- linux26     for Linux 2.6 and above  
- linux2628   for Linux 2.6.28, 3.x, and above (enables splice and tproxy)  
- solaris     for Solaris 8 or 10 (others untested)  
- freebsd     for FreeBSD 5 to 10 (others untested)  
- netbsd      for NetBSD   - osx         for Mac OS/X  
- openbsd     for OpenBSD 3.1 and above  
- aix51       for AIX 5.1  
- aix52       for AIX 5.2  
- cygwin      for Cygwin  
- generic     for any other OS or version.  
- custom      to manually adjust every setting
```

사용중인 리눅스 커널 버전에 따라 TARGET을 지정해 줘야 합니다. **필자는 3.10.0 전을 사용 중이므로, linux2628을 TARGET으로 지정**하겠습니다.

```bash
> uname -a
Linux CentOS7-CICD 3.10.0-693.17.1.el7.x86_64 #1 SMP Thu Jan 25 20:13:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

이제 컴파일을 진행해 보겠습니다. https 사용이 필요하기때문에 USE_OPENSSL 옵션을 활성화 하도록 합니다.

```
> cd ~/haproxy-1.8.3
> make TARGET=linux2628 USE_OPENSSL=1
```

컴파일 진행도중 오류가 발생한다면, HAProxy과 의종성이 있는 라이브러리가 설치가 되어 있지 않아서 발생할 경우가 많습니다.

저의 경우는 openssl부분에서 오류가 발생하여 다음 명령으로 설치를 진행하였습니다.

```bash
> yum -y install openssl-devel
```



#### 1.5. 설치

정상적으로 컴파일이 완료되었다면 `make install`명령으로 HAProxy를 설치합니다.

```bash
> sudo make install
install -d "/usr/local/sbin"
install haproxy  "/usr/local/sbin"
install -d "/usr/local/share/man"/man1
install -m 644 doc/haproxy.1 "/usr/local/share/man"/man1
install -d "/usr/local/doc/haproxy"
for x in configuration proxy-protocol management architecture peers-v2.0 cookie-options lua WURFL-device-detection linux-syn-cookies network-namespaces DeviceAtlas-device-detection 51Degrees-device-detection netscaler-client-ip-insertion-protocol peers close-options SPOE intro; do \
	install -m 644 doc/$x.txt "/usr/local/doc/haproxy" ; \
done
```



#### 1.6.설정

정상적으로 설치가 완료되었다면 다음과 같이 HAProxy 설정을 진행해야 합니다.



##### 1.6.1. haproxy.cfg 파일 생성

examlple 디렉토리 내의 cfg파일을 참고하여, /etc/haproxy 디렉토리 내에 haproxy.cfg 파일을 생성해 주어야 합니다.

```bash
> mkdir -p /etc/haproxy
> mkdir -p /var/lib/haproxy
> touch /var/lib/haproxy/stats

> vi /etc/haproxy/haproxy.cfg

global
  log /dev/log local0
  log /dev/log local1 notice
  chroot /var/lib/haproxy
  stats timeout 30s
  user haproxy
  group haproxy
  daemon

defaults
  # 기본 설정
  log global
  mode http
  option httplog
  option dontlognull
  timeout connect 5000ms
  timeout client 50000ms
  timeout server 50000ms

listen stats
  # HAProxy 모니터링 설정
  bind :8888
  mode http
  log global

  maxconn 10

  clitimeout 100s
  srvtimeout 100s
  contimeout 100s
  timeout queue 100s

  stats enable
  stats hide-version
  stats refresh 30s
  stats show-node
  stats auth admin:1234
  stats uri /haproxy

frontend http_front
  bind *:80
  default_backend http_back

backend http_back
  # 헬스체크
  option httpchk GET /
  http-check expect status 200
  default-server inter 1s fall 3 rise 2
  # 로드 밸런싱
  balance roundrobin
  server api1 localhost:8081 check
  server api2 localhost:8082 check
```

- default-server inter 1s fall 3 rise 2 : **1초 마다 서버에 접속**하여 헬스체크를 행하되 **3번 실패하면 접속 불가**로 판단하고 **2번 성공하면 정상 상태**로 간주
- option httpchk GET / : http `GET` 으로 `[서버주소]/` 요청을 날려서 `http-check expect status 200` 에 따라 `200 OK` 에 해당하는 응답이 나올시 정상 상태라고 판단



##### 1.6.2. 서비스 등록

/etc/init.d 디렉토리는 서비스 데몬 제어 파일을 저장하는 디렉토리 입니다. service에 등록하기 위해 init.d 디렉토리 내에 example/haproxy.init 파일을 복사한 후 퍼미션을 변경해 주도록 합니다.

```bash
> cp ~/haproxy-1.8.3/examples/haproxy.init /etc/init.d/haproxy
> chmod 755 /etc/init.d/haproxy
```



##### 1.6.3. 명령어 등록

/user/sbin 디렉토리는 명령어가 저장되는 디렉토리 입니다. haproxy 명령어를 사용할 수 있도록 /usr/sbin 디렉토리에 링크를 설정합니다.

```bash
> ln -s /usr/local/sbin/haproxy /usr/sbin/haproxy
```



##### 1.6.4. 사용자 등록

```bash
> sudo useradd -r haproxy
```



### 2. 서비스 시작 및 등록

서비스 시작 및 등록을 하기 앞서 설정한 값들을 적용시키기 위해 재시작을 진행합니다.

```bash
> reboot
> systemctl start haproxy

# 부팅할때 자동 실행
> systemctl enable haproxy
```



### 3. 방화벽

방화벽에 의해 HAProxy 모니터링 사이트에 접속이 되지 않는다면 다음 명령어를 수행하여, 방화벽 제한을 풀어 주어야 합니다.

```bash
> firewall-cmd --permanent --zone=public --add-service=http
> firewall-cmd --permanent --zone=public --add-port=[PORT]/tcp
> firewall-cmd --reload
```



### 4. Reference

- http://d2.naver.com/helloworld/284659