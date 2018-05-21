---
title: "[Jenkins Blue Ocean] DashBoard"
categories: DevOps
tags: [DevOps, Jenkins, Jenkins Pipeline, Blue Ocean]
published: true
comments: true
---



> 해당 글은 [Blue Ocean - Dashboard](https://jenkins.io/doc/book/blueocean/dashboard/) 글을 요약 정리할 글입니다.



### Dashboard

Blue Ocean의 **Dashboard** 는 Blue Ocean의 인덱스 화면이며, 젠킨스에 구성된 모든 파이프라인 프로젝트의 개요를 확인할 수 있습니다.

Dashboard는 네비게이션 바, 파이프라인 목록, 즐겨찾기 목록 으로 구성되어 있습니다.

![](/images/2018/0521_01_01.png)



### Navigation bar

네비게이션 바는 대시보드 상단의 파란색 부분에 해당합니다. 네비게이션 바는 다음과 같이 구성되어 있습니다.

- Search pipelinse: 파이프라인 목록을 필터링할 수 있습니다.
- New Pipeline: 새로운 파이프라인 프로세스를 생성할 수 있습니다.



### Pipelinse list

파이프라인 목록은 대시 보드의 기본 목록이며 처음으로 Blue Ocean에 액세스하면 이것이 대시 보드에 표시된 유일한 목록입니다.

이 목록은 Jenkins 인스턴스에 구성된 각 파이프 라인의 전체 상태를 보여줍니다. 다음 정보가 표시됩니다.

- 이름
- 상태
- git 저장소에서 **BRANCHS**, **PR(Pull Request)** 의 passing 또는 failing 횟수
- 즐겨찾기 활성화 버튼(별)



### Health icons

Blue Ocean은 프로세스가 정상적으로 통과 한 최근 빌드의 수에 따라 변경되는 날씨 아이콘을 사용하여 파이프 라인/item/branch의 전반적인 상태를 나타냅니다.

| Icon                             | health                                      |
| -------------------------------- | ------------------------------------------- |
| ![](/images/2018/0521_01_02.svg) | **Sunny**, Runs passing 80% 이상            |
| ![](/images/2018/0521_01_03.svg) | **Partially Sunny**, Runs passing 61% ~ 80% |
| ![](/images/2018/0521_01_04.svg) | **Cloudy**, Runs passing 41% ~ 60%          |
| ![](/images/2018/0521_01_05.svg) | **Rainning**, Runs passing 21% ~ 40%        |
| ![](/images/2018/0521_01_06.svg) | **Storm**, Runs passing 21% 이하            |

### Run status

Blue Ocean은 아이콘 세트를 사용하여 파이프 라인/item/branch 중 하나의 실행 상태를 나타냅니다.

| Icon                             | Status      |
| -------------------------------- | ----------- |
| ![](/images/2018/0521_01_07.png) | In Progress |
| ![](/images/2018/0521_01_08.png) | Passed      |
| ![](/images/2018/0521_01_09.png) | Unstable    |
| ![](/images/2018/0521_01_10.png) | Failed      |
| ![](/images/2018/0521_01_11.png) | Aborted     |

