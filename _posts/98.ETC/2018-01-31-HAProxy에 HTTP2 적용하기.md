---
title: "HAProxy에 HTTP/2 적용하기"
categories: ETC
tags: etc
published: true
comments: true
---



현재 HTTP/2 스펙을 지원하는 브라우저들의 **HTTP/2 프로토콜 이용 조건으로 TLS을 전제**하고 있기때문에 SSL 인증서가 필요합니다. 공인인증기관(CA)로 부터 인증을 받는것이 제일 좋겠지만, 개인 혹은 테스트 용도로 사용하는 것이라면 비용이 발생하므로 부담이 될 수 있습니다.

다행스럽게도 인증서를 만드는 방법이 몇가지 있습니다.

- 자가 서명 인증서(Self Signed Certificate)
  - 인증 서명 요청(CSR) 명시적 생성 -> 인증서에 self-sign -> 인증서 발급
  - key와 부가정보들을 입력하여 직접 self-sign하여 인증서 발급
- 인증 서명 요청(CSR)을 만들어 공인 인증기관(CA)에 요청해서 발급 : **유료**



위의 방법 모두 공개키 기반(PKI)이므로 개인키(Private key)와 공개키(Public key)를 만들어야 합니다.



### 1. 디지털 인증서

디지털 인증서는 공인 인증 기관(CA)에서 발급하며, 제한된 시간동안에만 사용할 수 있습니다. 만료일이 지나면 인증서를 교체해야 합니다. 인정서는 키 교환, 서버 인증, 클라이언트 인증에 사용됩니다.



인증서에는 인증서 소유자의 신분과 인증서 기구에 대한 정보가 포함되어 있어야 한다.

- 소유자 이름
- 소유자 퍼블릭 키
- 디지털 인증서가 발행된 날짜
- 인증서의 종료 날짜
- 발행 기구 이름 (CA의 기구명)
- 발행 기구의 디지털 서명




> **Note:** [생활코딩 - HTTPS와 SSL 인증서](https://opentutorials.org/course/228/4894) 에 SSL 인증서 원리가 이해하기 쉽게 잘 설명하고 있으므로, SSL 개념과 원리를 정확히 모르시는 분들은 참고하시면 좋을거 같습니다.



### 2. OpenSSL

#### 2.1. 다운로드

```bash
> wget https://www.openssl.org/source/openssl-1.0.2n.tar.gz ~/openssl-1.0.2n.tar.gz
```



#### 2.2. 압축 해제

```bash
> tar xzvf ~/openssl-1.0.2n.tar.gz -C ~/
```



#### 2.3. 옵션 설정

```bash
> ./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib no-shared zlib-dynamic
```



#### 2.4. 컴파일 & 설치

```bash
> make && make install
```



#### 2.5. 개인키 생성

```bash
# 암호 부여하지 않고 개인키 생성 방법
# openssl genrsa -out <키이름> <비트 수>
> openssl genrsa -out /etc/ssl/cert.key 2048

Generating RSA private key, 2048 bit long modulus
.+++
........................................+++
e is 65537 (0x10001)
```



#### 2.6. 공개키 생성

공개키는 개인키와 한 쌍을 이루기 때문에, 공개키를 생성할 때 개인키가 사용됩니다.

```bash
# openssl rsa -in <개인키 파일> -pubout -out <공개키 파일>
> openssl rsa -in /etc/ssl/cert.key -pubout -out /etc/ssl/cert_pub.key

writing RSA key
```



#### 2.7. 인증서명요청(CSR) 생성

아래와 같이 사용자의 정보를 알맞게 입력해 주도록 합니다.

```bash
# openssl req -new -key <개인키> -out <CSR 파일>
> openssl req -new -key /etc/ssl/cert.key -out /etc/ssl/cert.csr

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:KR
State or Province Name (full name) [Some-State]:Seoul
Locality Name (eg, city) []:Mapo
Organization Name (eg, company) [Internet Widgits Pty Ltd]:MrBlue
Organizational Unit Name (eg, section) []:WebPlatform
Common Name (e.g. server FQDN or YOUR name) []:www.mrblue.com
Email Address []:wyparks2@mrblue.com
```



`enter` 를 눌러줍니다.

```bash
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```



#### 2.8. Slef-Sigend 인증서 생성

```bash
# 인증서명요청(CSR)을 삽입하여 인증서를 생성하는 방법
# openssl x509 -req -days <유효날수> -in <인증사인요청파일> -signkey <개인키> -out <인증서 파일명>
> openssl x509 -req -days 3650 -in /etc/ssl/cert.csr -signkey /etc/ssl/cert.key -out /etc/ssl/cert.crt

signkey /etc/ssl/cert.key -out /etc/ssl/cert.crt
Signature ok
subject=/C=KR/ST=Seoul/L=Mapo/O=MrBlue/OU=WebPlatform/CN=www.mrblue.com/emailAddress=wyparks2@mrblue.com
Getting Private key
```



#### 2.9. 인증서 정보 보기

```bash
> openssl x509 -text -in /etc/ssl/cert.crt -noout
```



#### 2.10. 인증서 포맷 변환

HAProxy와 호환되도록 인증서와 개인키를 .pem 형식으로 변환합니다.

```bash
> cat /etc/ssl/cert.crt /etc/ssl/cert.key > /etc/ssl/cert.pem
```



### 3. HAProxy

HAProxy 에서 HTTP/2를 사용하기 위해선 별도의 설정값을 입력해 주어야 합니다. 아래의 몇 줄의 변경을 통해 우리는 간단하게 HTTP/2 프로토콜을 사용할 수 있습니다.



##### 3.1. H2 설정

앞서 [CentOS7에 HAProxy 설치하기]()에서 설정파일에서 frontend http_front 부분만 변경해 주도록 합니다.

```bash
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
  # HTTP/2 - see https://www.haproxy.com/blog/whats-new-haproxy-1-8/
  # h2 is HTTP2 with TLS - see https://http2.github.io/faq/
  # Order matters, so h2 before http1.1
  bind *:443 ssl crt /etc/ssl/cert.pem alpn h2,http/1.1
  
  # Redirect http -> https
  bind *:80
  redirect scheme https code 301 if ! { ssl_fc }
  
  # Everything else goes to the http_back servers
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



##### 3.2. 재시작

재시작을 통해 HTTP2 프로토콜이 적용되었는지 확인하도록 합니다.

```bash
systemctl restart haproxy
```



### 4. Reference

- [HTTP/2 에서 TLS 지원은 필수 사항일까?](http://ondemand.tistory.com/224)
- [[홈서버 구축기] SSL 인증서 만들기 (연습)](https://blog.hangadac.com/2017/07/31/%ED%99%88%EC%84%9C%EB%B2%84-%EA%B5%AC%EC%B6%95%EA%B8%B0-ssl-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EC%97%B0%EC%8A%B5/)
- [OpenSSL 로 ROOT CA 생성 및 SSL 인증서 발급](https://www.lesstif.com/pages/viewpage.action?pageId=6979614)
- [openssl 로 자가 서명 인증서 만들기](http://ohgyun.com/429)
- [The complete guide to HTTP/2 with HAProxy and Nginx](http://m12.io/blog/http-2-with-haproxy-and-nginx-guide)
- [HAProxy in 2018 - A load balancer with HTTP/2 and dynamic reconfig.](https://certsimple.com/blog/haproxy-http2-dynamic-load-balancing-ssl)
- [HTTP/2 Checker 구현하기](https://brunch.co.kr/@sangjinkang/5)

