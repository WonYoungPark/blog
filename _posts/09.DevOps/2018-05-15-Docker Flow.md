---
title: "Docker Flow"
categories: DevOps
tags: [DevOps, Docker, Docker Flow, Service Discovery, Service Registry, Consul]
published: true
comments: true
---



>  해당 글은 [데브옵스 2.0 툴킷](http://www.aladin.co.kr/shop/wproduct.aspx?ItemId=115894773) 부록의 내용인 도커 플로우를 실습하고 정리한 글입니다.



## 도커 플로우란?

도커 플로우란 사용하기 쉬운 지속적인 배포 플로우를 생성하기 위한 프로젝트 입니다. 도커 플로우는 [도커 엔진](https://docs.docker.com/engine/), [도커 스웜 모드](https://docs.docker.com/glossary/?term=Docker%20Swarm), [Consul](https://www.consul.io/), [Registrator](https://github.com/gliderlabs/registrator)에 의존성이 있습니다.

도커 플로우 프로젝트의 목표는 현재 도커 생태게에는 없는 기능과 프로세스를 추가하는 것입니다. Blue-Green 배포, 자동 확장, Service Discovery, Service Registry 등을 다룹니다.



## 표준 설정

먼저 일반적으로 도커 스웜 클러스터를 설정을 살펴보고 오케스트레이터를 사용할 때 발생할 수 있는 문제점에 대해서 논의해 보겠습니다.

스웜 클러스트 내의 각 노드에는 도커 엔진과 스웜 컨테이너가 실행 되고 있어야 합니다. 또한 클러스트에는 적어도 하나 이상의 매니저 노드가 존재해야 하며, 모든 노드는 자신의 존재를 매니저 노드에 게시해야 합니다.

지금 까지 도커 스웜 클러스터에 관해서 논의 했다면 도커 스웜 클러스터가 자기고 있는 부족한 점에 대해서 논의해 보도록 하겠습니다. 기본적인 도커 스웜 클러스터를 구성할 경우에는 각 노드간에 통신하기란 쉽지 않습니다. 각 서비스는 네트워크 주소(IP와 포트)가 동적으로 생성되어 할당되어 지기 때문에 서비스의 네트워크 주소를 알 수 없습니다. 고정적인 네트워크 주소(IP와 포트)를 사용할 경우 자동 확장 기능을 저하시키고, 포트 충돌을 야기시킬 수 잇기 때문입니다. 그렇기 때문에 도커 스웜 클러스터를 구성할 경우 Service Registry에 서비스 인스턴스의 네트워크 주소를 등록할 필요가 있습니다.

[Registrator](https://github.com/gliderlabs/registrator) 는 **도커 엔진 이벤트를 모니터링 하여 배포 되거나 중지된 컨테이너에 대한 정보를 Service Registry에 보내는 역할**을 수행하는 도구 중 하나입니다.

![](/images/2018/0515_02_01.png)



작은 규모의 도커 스웜 클러스트 구성을 제외하고 여러 개의 매니저 노드(Swarm master)와 Consul 을 가지므로 이 중 하나가 실패하거나 중단 되더다고 서비스를 안정적으로 운영할 수 있습니다.

위와 같은 구성에서 컨테이너 배포 프로세스는 아래와 같습니다.

1. 매니저 노드(Swarm Master)에게 요청을 보냅니다. 이 요청은 `DOCKER_HOST=<매니저노드 IP>:<매니저노드 Port> docker ps` 명령으로 요청할 수 있습니다.
2. 요청이 전송된 기준(CPU, 메모리, 선호도 등)에 따라 매니저노드는 컨테이너를 배포할 워커노드를 결정하고 요청을 합니다.
3. 워커노드는 컨테이너를 실행(또는 중지)하라는 요청을 받으면 로컬 도커 엔진을 호출합니다. 그러면 로컨 도커엔진이 원하는 컨테이너를 실행(또는 중지)하고 결과를 이벤트로 게시합니다.
4. Registrator는 도커 엔진을 모니터링하고 있다가 새로운 이벤트를 감지하면 정보를 Consul에게 전송합니다.
5. Consul을 통해서 도커 스웜 클러스터에서 동작하는 컨테이너를 확인할 수 있습니다.

이 프로세스는 겉으로 보기에 문제가 없어보이지만 실제로 운영 환경에서는 다소 부족한 점이 있습니다.



### 문제

#### 무중단 배포



#### 상대적인 번호를 사용한 컨테이너 확장



#### 배포 후 프록시 재설정



## 도커 플로우 둘러보기

예제 프로젝트를 복제하여 도커 플로우를 직접 구성해보고, 실제로 어떻게 동작하는지 확인해 보도록 하겠습니다.



### 설정

아래의 명령을 통해 프로젝트를 복제하도록 하겠습니다.

```bash
$ git clone https://github.com/vfarcic/docker-flow.git
 
$ cd docker-flow
```



도커 스웜 클러스터를 시뮬레이션하기 위해 Vagrant를 사용합니다. 코드가 복제되면 Vagrant를 실행하여 도커 스웜 노드를 구성할 수 있습니다.

```bash
$ vagrant plugin install vagrant-cachier

$ vagrant up master node-1 node-2 proxy
```

![](/images/2018/0515_02_02.png)

- Consul: 도커 스웜 클러스터와 그 안에 실행되는 서비스들에 대한 모든 정보를 가지고 있습니다.
- Swarm Master: 요청 전송 기준에 따라 노드들에게 컨테이너를 실행(또는 중지) 하라는 요청을 보냅니다.
- Swarm Node: Swarm Master에게 받은 정보를 로컬 도커 엔진에게 전달합니다.
- Registrator: 도커 엔진 이벤트를 모니터링하여 실행(또는 중지) 되는 컨테이너 정보를 Consul에게 전송합니다.



`vagrant up` 명령이 완료되면 proxy VM에 접속하여 도커 플로우가 작동하는 모습을 볼 수 있습니다.

```bash
vagrant ssh proxy
```



### 배포 후 프록시 재설정

도커 플로우는 Consul 인스턴스의 주소와 프록시가 실행 중인 노드에 대한 정보가 필요합니다. 필요한 정보를 제공하는 세가지 방법이 있습니다.



```
.docker-compose \
    --blue-green \
    --target=app \
    --service-path="/demo" \
    --side-target=db \
    --flow=deploy --flow=proxy
```

