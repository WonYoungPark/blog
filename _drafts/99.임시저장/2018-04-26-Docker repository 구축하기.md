



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
$ vi /etc/systemd/system/docker.service.d/hosts.conf

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 

$ systemctl daemon-reload
$ systemctl restart docker
```



### 도커 클라이언트

- `-H` 옵션을 사용

```shell
$ dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```



- 도커 클라이언트는 `DOCKER_HOST` 환경변수 또는 `-H` 옵션을 사용하여 도커 데몬과 통신할 수 있습니다.

```shell
# 옵션을 사용한 방법
$ docker -H tcp://0.0.0.0:2375 ps

#or

# 환경변수를 사용한 방법
$ export DOCKER_HOST="tcp://0.0.0.0:2375"
$ docker ps

#or
$ DOCKER_HOST=tcp://0.0.0.0:2375 docker ps
```




## Docker-machine

docker 에서는 docker 머신을 통해 다수의 호스트에 쉽게 도커를 엔진을 생성할 수 있도록 도와 줍니다.

### 호스트 추가

`docker-machine create <머신 명>` 명령을 사용합니다.

```shell
$ docker-machine create manager1
```

- `--driver virtualbox`: 버추얼박스 사용
- `--virtualbox-cpu-count "2"` : CPU 코어의 수를 2 개로 지정
- `--virtualbox-memory "2048"`: 메모리 크기를 2 GB로 지정
- `--virtualbox-disk-size "50000"`: 메모리 크기를 50 GB로 지정



output:

```shell
Creating CA: /Users/mrblue-one0/.docker/machine/certs/ca.pem
Creating client certificate: /Users/mrblue-one0/.docker/machine/certs/cert.pem
Running pre-create checks...
(manager1) Image cache directory does not exist, creating it at /Users/mrblue-one0/.docker/machine/cache...
(manager1) No default Boot2Docker ISO found locally, downloading the latest release...
(manager1) Latest release for github.com/boot2docker/boot2docker is v18.05.0-ce
(manager1) Downloading /Users/mrblue-one0/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.05.0-ce/boot2docker.iso...
(manager1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(manager1) Copying /Users/mrblue-one0/.docker/machine/cache/boot2docker.iso to /Users/mrblue-one0/.docker/machine/machines/manager1/boot2docker.iso...
(manager1) Creating VirtualBox VM...
(manager1) Creating SSH key...
(manager1) Starting the VM...
(manager1) Check network to re-create if needed...
(manager1) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env manager1
```



### 호스트 접속

docker-machine으로 생성한 호스트는 ssh를 사용하여 접속할 수 있습니다.

```shell
$ docker-machine ssh manager1
```



## 도커 스웜

### 도커 스웜에서 사용하는 용어

**스웜swarm**: 직역하면 떼, 군중이라는 뜻을 가지고 있습니다. 도커 1.12버전에서 스웜이 스웜 모드로 바뀌었지만 그냥 스웜이라고도 하는듯 합니다. 스웜 클러스터 자체도 스웜이라고 합니다. (스웜을 만들다. 스웜에 가입하다. = 클러스터를 만들다. 클러스터에 가입하다)

**노드node**: 스웜 클러스터에 속한 도커 서버의 단위입니다. 보통 한 서버에 하나의 도커데몬만 실행하므로 서버가 곧 노드라고 이해하면 됩니다. 매니저 노드와 워커 노드가 있습니다.

**매니저노드manager node**: 스웜 클러스터 상태를 관리하는 노드입니다. 매니저 노드는 곧 워커노드가 될 수 있고 스웜 명령어는 매니저 노드에서만 실행됩니다.

**워커노드worker node**: 매니저 노드의 명령을 받아 컨테이너를 생성하고 상태를 체크하는 노드입니다.

**서비스service**: 기본적인 배포 단위입니다. 하나의 서비스는 하나의 이미지를 기반으로 생성하고 동일한 컨테이너를 한개 이상 실행할 수 있습니다.

**테스크task**: 컨테이너 배포 단위입니다. 하나의 서비스는 여러개의 테스크를 실행할 수 있고 각각의 테스크가 컨테이너를 관리합니다.

### 도커 스웜이 제공하는 기능

Docker 1.13을 기준으로 어떤 기능을 제공하는지 하나하나 살펴봅니다.

**스케줄링 scheduling**: 서비스를 만들면 컨테이너를 워커노드에 배포합니다. 현재는 균등하게 배포spread하는 방식만 지원하며 추후 다른 배포 전략이 추가될 예정입니다. 노드에 라벨을 지정하여 특정 노드에만 배포할 수 있고 모든 서버에 한 대씩 배포하는 기능(Global)도 제공합니다. 서비스별로 CPU, Memory 사용량을 미리 설정할 수 있습니다.

**고가용성 High Available**: [Raft 알고리즘](https://raft.github.io/)을 이용하여 여러 개의 매니저노드를 운영할 수 있습니다. 3대를 사용하면 1대가 죽어도 클러스터는 정상적으로 동작하며 매니저 노드를 지정하는 방법은 간단하므로 쉽게 관리할 수 있습니다.

**멀티 호스트 네트워크 Multi Host Network**: Overlay network로 불리는 SDN(Software defined networks)를 지원하여 여러 노드에 분산된 컨테이너를 하나의 네트워크로 묶을수 있습니다. 컨테이너마다 독립된 IP가 생기고 서로 다른 노드에 있어도 할당된 IP로 통신할 수 있습니다. (호스트 IP를 몰라도 됩니다!)

**서비스 디스커버리 Service Discovery**: 서비스 디스커버리를 위한 자체 DNS 서버를 가지고 있습니다. 컨테이너를 생성하면 서비스명과 동일한 도메인을 등록하고 컨테이너가 멈추면 도메인을 제거합니다. 멀티 호스트 네트워크를 사용하면 여러 노드에 분산된 컨테이너가 같은 네트워크로 묶이므로 서비스 이름으로 바로 접근할 수 있습니다. Consul이나 etcd, zookeeper와 같은 외부 서비스를 사용하지 않고 어떠한 추가 작업도 필요 없습니다. 스웜이 알아서 다 해 줍니다.

**순차적 업데이트 Rolling Update**: 서비스를 새로운 이미지로 업데이트하는 경우 하나 하나 차례대로 업데이트합니다. 동시에 업데이트하는 작업의 개수와 업데이트 간격 시간을 조정할 수 있습니다.

**상태 체크 Health Check**: 서비스가 정상적으로 실행되었는지 확인하기 위해 컨테이너 실행 여부 뿐 특정 쉘 스크립크가 정상으로 실행됐는지 여부를 추가로 체크할 수 있습니다. 컨테이너 실행 여부로만 체크할 경우 아직 서버가 실행 되지 않아 접속시 오류가 날 수 있는 미묘한 타이밍을 잡는데 효과적입니다.

**비밀값 저장 Secret Management**: 비밀번호를 스웜 어딘가에 생성하고 컨테이너에서 읽을 수 있습니다. 비밀 값을 관리하기 위한 외부 서비스를 설치하지 않고 쉽게 사용할 수 있습니다.

**로깅 Logging**: 같은 노드에서 실행 중인 컨테이너뿐 아니라 다른 노드에서 실행 중인 서비스의 로그를 한곳에서 볼 수 있습니다. 특정 서비스의 로그를 보려면 어느 노드에서 실행 중인지 알필요도 없고 일일이 접속하지 않아도 됩니다.

**모니터링 Monitoring**: 리소스 모니터링 기능은 제공하지 않습니다. 3rd party 서비스([prometheus](https://prometheus.io/), [grafana](http://grafana.org/))를 사용해야 합니다. 설치는 쉽지만 은근 귀츈…

**크론, 반복작업 Cron**: 크론, 반복작업도 알아서 구현해야 합니다. 여러 가지 방식으로 해결할 수 있겠지만 직접 제공하지 않아 아쉬운 부분입니다.



### 스웜 클러스터 생성

스웜 클러스터를 생성해 보도록 하겠습니다. 스웜 클러스터는 우선 매니저 노드를 생성하고, 매니저 노드가 생성한 토큰을 사용하여, 워커 노드에서 매니저 노드로 접속하는 방식입니다.

매니저 노드를 설정하기 위해서  `docker swarm init` 명령을 사용하면 됩니다.

```shell
$ docker swarm init --advertise-addr <MANAGER-IP>
```

output:

```shell
Swarm initialized: current node (isshrd2f0ffvh3hjfvs8rfs7p) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4lkznaw651fjl5fe4n8qv76wh3nwhji1jz821d0gydzf4laykb-0658mrxgscb8fygtnfpk1n6vn 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

매니저 노드가 생성되면서 워커 노드에서 매니저 노드로 접속하는 명령어를 알려줍니다. 해당 명령어를 복사하였다가 추후에 워커노드에서 사용하도록 하겠습니다.

> **Tip **
>
> - ip는 `docer-machine ls` 커맨드를 입력하면 URL 컬럼에 출력됩니다.
> - 해당 토큰을 복사하지 못하였다면 매니저 노드에서 `docker swarm join-token manager` 명령를 실행하시면 매니저 노드 접속 명령어를 반환합니다.



### Node 조회

매니저 노드에서 `docker node ls` 커맨드를 수행하여 스웜에서 노드를 확인할 수 있습니다.

```shell
$ docker node ls
```



### Worker 노드 추가

```shell
$ docker-machine create worker1
$ docker-machine ssh worker1

$ docker swarm join --token SWMTKN-1-4lkznaw651fjl5fe4n8qv76wh3nwhji1jz821d0gydzf4laykb-0658mrxgscb8fygtnfpk1n6vn 192.168.99.100:2377
```



```shell
$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
isshrd2f0ffvh3hjfvs8rfs7p *   manager1            Ready               Active              Leader              18.05.0-ce
tsgko4ffpuhzz14tfogoozogv     worker1             Ready               Active                                  18.05.0-ce
1a5gy8nbd8zlb4upz1my1v4uz     worker2             Ready               Active                                  18.05.0-ce
```



### 서비스 생성

클러스터를 구성하였으니 간단한 서비스를 생성해 보도록 하겠습니다.

서비스를 생성하려면 **매니저 노드**에서 `docker service create` 명령을 사용합니다.

```bash
$ docker service create --replicas 1 --name helloworld alpine ping docker.com
```



### 서비스 목록

생성된 서비스가 정상적으로 생성되었는지 확인해 보도록 하겠습니다.

매니저 노드에서 `docker service ls` 명령을 통해 현재 실행 중인 서비스들의 목록을 확인 할 수 있습니다.

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
c8t242k2azjn        helloworld          replicated          1/1                 alpine:latest
```



### 서비스 상세 상태

좀 더 상세한 서비스 정보를 확인하기 위해서 `docker service ps <서비스 명>` 을 사용합니다.

해당 서비스가 어떠한 worker 노드에서 구동 중인지 확인할 수 있습니다.

```bash
$ docker service ps helloworld

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
kxr96fj59yj3        helloworld.1        alpine:latest       manager1            Running             Running 3 minutes ago
```



```bash
$ docker service inspect helloworld --pretty

ID:		c8t242k2azjnruvosorm4rwpp
Name:		helloworld
Service Mode:	Replicated
 Replicas:	1
Placement:
UpdateConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:	1
 On failure:	pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:		alpine:latest@sha256:7df6db5aa61ae9480f52f0b3a06a140ab98d427f86d8d5de0bedab9b8df6b1c0
 Args:		ping docker.com
Resources:
Endpoint Mode:	vip
```

> **Tip:** 스웜 명령어는 매니저 노드에서만 실행됩니다.



### Scale 조정

서비스를 운영하다 보면 트래픽에 따라서 서버를 증설해야할 경우가 발생합니다. 

도커 스웜은 간편하게 서비스를 증설할 수 있는 명령어를 지원합니다.

`docker service scale <SERVICE_NAME>=<NUMBER>` 명령을 이용하면 됩니다.

```bash
$ docker service scale helloworld=5
helloworld scaled to 5
overall progress: 2 out of 5 tasks
1/5: preparing [=================================>                 ]
2/5: preparing [=================================>                 ]
3/5: running   [==================================================>]
4/5: preparing [=================================>                 ]
5/5: running   [==================================================>]
```



`docker service ps <SERVICE_NAME>` 명령을 사용하여 상태를 확인합니다.

```
docker service ps helloworld
```



### 서비스 삭제

```bash
$ docker service rm helloworld
```



## Portainer

- Portainer는 다른 docker 환경을 쉽게 관리할 수 있는 경량 관리 UI 입니다.
- Portainer를 사용하면 쉽게 배포할 수 있습니다. docker 엔진에서 실행할 수 있는 단일 컨테이너로 구성되어있습니다.
- Portainer를 사용하면 docker 컨테이너, 이미지, 볼륨, 네트워크 등을 관리할 수 있습니다. 독립 실행형 docker 엔진 및 docker 스웜모드와 호환됩니다.



### 설치

도커 스웜 환경에서 Portainer를 사용하려면 아래 명령어를 수행하면 됩니다.

```bash
$ docker service create \
--name portainer \
--publish 9000:9000 \
--replicas=1 \
--constraint 'node.role == manager' \
--mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
portainer/portainer \
-H unix:///var/run/docker.sock
```

