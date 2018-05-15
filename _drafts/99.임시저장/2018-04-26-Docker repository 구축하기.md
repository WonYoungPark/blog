



## Private docker registry 구축

docker registry의 기본포트는 5000입니다. 기본설정으로 진행해보도록 하겠습니다.

```bash
# docker registry image를 가져온다.
> docker pull registry
# docker registry를 실행한다.
> docker run -dit --name docker-registry -p 5000:5000 registry
```



### docker run 명령어 및 옵션

```bash
docker run <옵션> <이미지 이름, ID> <명령> <매개 변수>
```

- --name : 컨테이너에 이름을 설정
- -d : Detached 모드, 보통 데몬 모드라고 부르며 컨테이너가 백그라운드로 실행
- -i : 표준 입력(stdin)을 활성화하며 컨테이너와 연결(attach)되어 있지 않더라도 표준 입력을 유지합니다. 보통 이 옵션을 사용하여 Bash에 명령을 입력합니다.
- -t : TTY 모드(pseudo-TTY)를 사용합니다. Bash를 사용하려면 이 옵션을 설정해야 합니다. 이 옵션을 설정하지 않으면 명령을 입력할 수는 있지만 셸이 표시되지 않습니다.
- -p : 호스트에 연결된 컨테이너의 특정 포트를 외부에 노출합니다.



## Docker image push

도커 이미지를 만들어서 도커 레파지토리에 푸쉬하는 절차를 진행해 보도록 하겠습니다.

```bash
# docker image pull
> docker pull hello-world

# cicd.mrblue.com:5000/hello-world 이미지 생성 
> docker tag hello-world cicd.mrblue.com:5000/hello-world

# 이미지 생성여부 확인
> docker images
```



이미지를 생성하고 이미지에 태그를 설정하였습니다. 도커 레파지토리에 푸쉬를 진행해 보도록 하겠습니다.

```bash
# docker image push
> docker push cicd.mrblue.com:5000/hello-world

# 이미지 확인
> curl -X GET http://cicd.mrblue.com:5000/v2/_catalog

# 이미지 태크 정보 확인
> curl -X GET http://cicd.mrblue.com:5000/v2/hello-world/tags/list
```



### docker tag 명령어 및 옵션

```bash
docker tag <옵션> <이미지 이름>:<태그> <저장소 주소, 사용자명>/<이미지 이름>:<태그>
```





## Dockerfile

```dockerfile
FROM openjdk:8-jre-alpine
MAINTAINER One.0 <wyparks2@mrblue.com>

VOLUME ["/tmp"]
ADD ${JAR_FILE} app.jar
ENV JAVA_OPTS="-Dspring.profiles.active=production"

ENTRYPOINT ["java", "$JAVA_OPTS", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app.jar"]
EXPOSE 8100
```

### 명령어 및 옵션

- FROM : 어떤 이미지를 기반으로 이미지를 생성할지 설정합니다. `FROM <이미지> 또는 FROM <이미지>:<태그>`형식 입니다.
- MAINTAINER : 이미지를 생성한 사람의 정보를 설정합니다. `MAINTAINER <작성자 정보>`형식 입니다.
- VOLUME : 디렉터리의 내용을 컨테이너에 저장하지 않고 호스트에 저장하도록 설정합니다.
- ADD : 파일을 이미지에 추가합니다.
- ENV : 환경 변수를 설정합니다. ENV로 설정한 환경 변수는 RUN, CMD, ENTRYPOINT에 적용됩니다.
- ENTRYPOINT : 컨테이너가 시작되었을 때 스크립트 혹은 명령을 실행합니다.
- EXPOSE : 호스트와 연결할 포트 번호를 설정합니다. `docker run` 명령의 `--expose` 옵션과 동일합니다.



## Dokcer compose

Docker compose는 여러 컨테이너를 **쉽게 실행할 수 있도록** 도와주는 도커 실행 툴입니다. 도커 이미지를 실행하게되면 애플리케이션의 성격에 따라 옵션 설정들을 많이 해야되는 경우가 발생하는데, Docker compose는 이와 같은 옵션등을 yml파일 형태로 미리 선언해 놓고 사용할 수 있기때문에 애플리케이션을 쉽고, 실수로 인한 장애를 예방하는데 도움을 줍니다.



```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "8100:8080"
    volumes:
      - .:/logs
    restart: always

```

### 명령어 및 옵션

- build : `Dockerfile`에 이미지 구성을 작성하고, 이를 자동으로 build하여 이미지로 지정합니다. (docker-compose.yml` 파일 경로 기준으로 `Dockerfile` 디렉토리 경로를 지정합니다.)
- port : 호스트와 게스트 port를 매핑합니다.
- expose : 포트 번호를 호스트에 공개하지 않고 링크로 연계된 컨테이너에만 공개합니다. (데이터베이스 서버와 같이 호스트 머신에 직접 액세스 하지 않고 컨테이너 끼리 커넥션되어 사용할 경우 주로 사용합니다.)
- volumes
  - 컨테이너에 볼륨을 마운트합니다.
  - `호스트 디렉토리 경로:컨테이너의 디렉토리 경로`형식
  - 볼륨을 읽기 전용(read only)로 마운트하고자 할 경우는 마지막에 `:ro` 접미사를 붙입니다.
- enviroment : 컨테이너 내의 환경변수를 지정합니다.
- container_name : Docker Compose에서 생성된 컨테이너 이름을 명시합니다.



### 커맨드

| 옵션    | 비고                        |
| ------- | --------------------------- |
| up      | 컨테이너 생성 및 구동       |
| scale   | 생성할 컨테이너 개수 지정   |
| ps      | 컨테이너 목록 확인          |
| logs    | 컨테이너 로그 출력          |
| run     | 컨테이너 실행               |
| start   | 컨테이너 구동               |
| stop    | 컨테이너 중지               |
| restart | 컨테이너 재가동             |
| kill    | 실행중인 컨테이너 강제 종료 |
| rm      | 컨테이너 삭제               |



## Webhook

현재 사내 배포 프로세스로 `git`, `jenkins`, `docker`, `docker registry`를 사용하고 있습니다. yona에 webhook을 등록하여, commit 이벤트를 젠킨스에서 감지하고, 젠킨스에서 `테스트`, `빌드`, `도커 이미지 빌드`, `도커 레지스트리 푸쉬` 등의 프로세스를 진행하기 위해 webhook을 등록해보겠습니다.

현재 사내에서 형상관리 도구로 [Yona(git)](https://github.com/yona-projects/yona)를 도입하여 사용하고 있기 때문에 Yona 기준으로 설명하겠습니다.



### Jenkins 빌드 유발 설정

![0430_01_03](/Users/mrblue-one0/Dev/01.Project/02.GitPage/WonYoungPark.github.io/images/2018/0430_01_03.png)

- 빌드를 원격으로 유발을 선택합니다.
- Authentication Token 입력란에 토큰값을 입력합니다.(아무 값이나 넣어줍니다.)



### Jenkins API Token 발급

![0430_01_01](/Users/mrblue-one0/Dev/01.Project/02.GitPage/WonYoungPark.github.io/images/2018/0430_01_01.png)

1. 사용자 클릭
2. 설정 클릭
3. API Token 발급



### Webhook 등록

![0430_01_02](/Users/mrblue-one0/Dev/01.Project/02.GitPage/WonYoungPark.github.io/images/2018/0430_01_02.png)

- 전송할 주소에 `[아이디][API_Token]@[JENKINS_URI]/job/[JOB_NAME]/build?token=[JENKINS_AUTHENTICATION_TOKEN]` 형태로 값을 입력합니다.
- `Git push hook` **콤보박스를 체크**합니다.
- 웹후크 추가 버튼을 클릭합니다.



이제 commit 이벤트가 발생하면 Jenkins Job이 Build 되는것을 확인할 수 있습니다.





## Docker regisrey Http 배포

인증서를 등록하지 않고, 도커 레지스트리에 이미지를 배포를 시도하면 아래와 같은 오류가 발생합니다.

```bash
x509: certificate signed by unknown authority
```

인증서를 등록하여 사용해도 되지만, 간단하게 사용하기 위해 레지스트리 보안을 무시하는 설정을 구성해 보겠습니다. `/etc/docker/daemon.json` 파일을 아래와 같이 수정합니다.

```bash
{
  "insecure-registries" : ["myregistrydomain.com:5000"]
}
```



### Docker Remote API

도커 데몬이 수신할 수 있는 도커 엔진API는 세가지 소켓 유형을 가지고 있습니다.(`unix`, `tcp`, `fd`).

도커는 기본적으로 unix 소켓이 활성화 되어 있습니다. 도커 데몬에 원격으로 액세스하고 싶을 경우 `tcp` 소켓을 활성화 해야합니다.



### 도커 데몬

- conf 설정

```shell
> vi /etc/systemd/system/docker.service.d/hosts.conf

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 

> systemctl daemon-reload
> systemctl restart docker
```



### 도커 클라이언트

- `-H` 옵션을 사용

```shell
> dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```



- 도커 클라이언트는 `DOCKER_HOST` 환경변수 또는 `-H` 옵션을 사용하여 도커 데몬과 통신할 수 있습니다.

```shell
# 옵션을 사용한 방법
> docker -H tcp://0.0.0.0:2375 ps

#or

# 환경변수를 사용한 방법
> export DOCKER_HOST="tcp://0.0.0.0:2375"
> docker ps

#or
> DOCKER_HOST=tcp://0.0.0.0:2375 docker ps
```