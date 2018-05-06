---
title: "gitlab - SSH 공개키 등록하기"
categories: ETC
tags: etc
published: true
comments: true
---





### 1. SSH 공개키 등록하기

gitlab project를 clone하기위해서 SSH 공개키를 등록하도록 하자.

#### 1.1. 로그인

브라우저 URL창에 `https://192.168.4.236/gitlab` 입력하여 접속한다. 로그인 화면이 출력되었으면 할당받은 아이디와 패스워드를 입력하여 로그인하도록 한다.

![](/images/2017/0902_01_01.png)

#### 1.2. SSH 공개키 등록화면 이동

`Setting -> SSH Keys`으로 접근한다.

![](/images/2017/0902_01_02.png)



#### 1.3. SSH 공개키 가져오기 

아래 명령어를 통해서 공개키를 조회한다.

```
cd /Users/one0/.ssh
cat id_rsa.pub
```



아래와 같은 공개키를 확인 할 수 있다. 공개키를 복사(`Ctrl + c`하도록 한다.)

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDB3yL4gIY98QJ8FPRf6ROzQL2F+gVPuR5PPfx/BIKp5Sck/RVnWMrl4tdSRSo1KaSRKHwZ+IoNcuUsZW/ft6j07AnXLnJhhuYgoGBYeJb5UGbCEFctjA4xAxg0nnJD113AfIh9wRWZnyvON7oDkF+p1Wf6RevyPSqRirKehgbvMKM93lkiYzzFvmtd7tnm3G0/GXay7AzctD2/XCyV0W/dRYYsHMOFjJwV8/kla/CPmorhj6D45+XbTe7FPDszC1gY7unUD8GFwhyVOSK/v1IjDqvFUFzJnPqMCIsB4z/Sxmd3ScvGTKm4zkM2na7gt+NQMHkYkOpkYxxbt4Go49 one0@One0-MacBook
```



#### 1.4. SSH 공개키 등록하기

아래와 같이 복사한 공개키를 Key란에 붙여넣기 하도록한다. Title란은 임의의 값을 넣도록 한다. 값을 모두 입력하였으면 `Add key`를 눌러 SSH 공개키 등록을 완료하도록 한다.

![](/images/2017/0902_01_03.png)

