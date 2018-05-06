---
title: "gitlab - SSH 공개키 만들기"
categories: ETC
tags: etc
published: true
comments: true
---



### 1. SSH 공개키 만들기

많은 Git 서버들은 SSH 공개키로 인증을 한다. 공개키(public key)를 사용하려면 일단 공개키를 생성해야 한다. (아래의 내용은 Lunux OS 기준으로 작성되었다.)

#### 1.1. 생성

`ssh-keygen`이라는 프로그램으로 키를 생성해야 한다. `ssh-keygen`프로그램은 리눅스나 Mac의 SSH 패키지에 포함되어 있기 때문에 별도의 설치를 필요하지 않다.

아래의 명령어 입력을 통해서 public key와 private key 한쌍을 생성한다.

```
ssh-keygen
```



key가 생성될 경로를 입력하고 암호를 두번 입력한다. 이때 암호를 비워두면 키를 사용할 때 암호를 묻지 않는다. (보안상 입력을 권장한다.)

```
Enter file in which to save the key (/Users/one0/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
```



아래의 이미지가 출력된다면 public key가 정상적으로 생성된것을 의미한다.

```
Your identification has been saved in /Users/one0/.ssh/id_rsa.
Your public key has been saved in /Users/one0/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:WNlpwC+rDpXGezeKvxidU1fJ24nlHhpFS+5Cdmx7EZA one0@One0-MacBook
The key's randomart image is:
+---[RSA 2048]----+
|       ..    .o+ |
|        .+ . E=.o|
|        o.+  o+X |
|     . +... o.Oo+|
|      * So. .+.*o|
|     o o.o .  = o|
|    . o.= o  . . |
|     ..= + .     |
|     .+.+.       |
+----[SHA256]-----+
```



#### 1.2. ssh-key 생성확인

명령어 입력을 통해서 public key와 private key가 생성되었는지 확인하도록 한다.

```
cd /Users/one0/.ssh/
ls -al
```



아래와 같이 id_rsa, id_rsa.pub 파일이 생성되었다면 정상적으로 ssh-key가 생성된 것이다. 이중에 .pub파일이 공개키 이고, 다른 파일은 개인키이다.

![](/images/2017/0901_01_01.png)