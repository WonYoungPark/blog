---
title: "[Jenkins Blue Ocean] Activity View"
categories: DevOps
tags: [DevOps, Jenkins, Jenkins Pipeline, Blue Ocean]
published: true
comments: true
---



> 해당 글은 [Blue Ocean - Activity View](https://jenkins.io/doc/book/blueocean/activity/) 를 정리 및 요약한 글입니다.



Blue Ocean의 Activity 화면은 파이프 라인과 관련된 모든 활동을 확인할 수 있습니다.

![](/images/2018/0521_02_01.png)



### Navigation Bar

Activity 화면은 Navigation Bar를 화면 상단에 포함하고 있습니다. navigation 바는 다음과 같은 항목을 포함하고 있습니다.:

- 파이프라인 이름: 파이프라인이름을 확인할 수 있습니다.
- 즐겨찾기 토글: 즐겨찾기 여부를 동작/확인할 수 있습니다.
- 탭(Activity, Branches, Pull Request): 해당 탭 화면으로 이동합니다.



### Activity

Activity화면이 기본 화면입니다. Activity 화면은 최근에 완료 됐거나 진행 중인 리스트를 출력합니다. 이 목록의 각 행은 동작 상태, id number, git commit info, 소요시간, 완료여부를 표시합니다. 각 행을 클릭하면 파이프라인 동작 상세를 확인할 수 있습니다. **In Progress** 동작의 경우 **Stop** 버튼 클릭을 통해 **aborted** 할 수 있습니다.



### Branches

Branches 탭에서는 현재 파이프라인에서 완료 또는 진행 중인 모든 Branch 목록을 표시합니다.

![](/images/2018/0521_02_02.png)