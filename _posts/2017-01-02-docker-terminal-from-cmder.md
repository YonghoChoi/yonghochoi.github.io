---
layout: post
title:  "Windows에서 cmder로 docker terminal 사용"
date:   2017-01-02 19:16:00 +0900
tags: [docker]
---

## 사전 준비

* [docker toolbox 설치](https://docs.docker.com/toolbox/toolbox_install_windows/)
* [git 설치](https://git-scm.com/downloads)
* [cmder 다운로드](http://cmder.net/)

## 설치 경로

* docker toolbox : "C:\Program Files\Docker Toolbox"
* git : C:\Program Files\Git

## cmder 설정

* cmder 실행 후 **윈도우키 + Alt + t**로 Settings 진입
* + 버튼 클릭 후 아래와 같이 설정
  ![](http://yonghochoi.github.io/images/docker/cmder_for_docker_1.png)
```
task명 : docker (임의로 지정 가능)
Task parameters : /icon "C:\Program Files\Docker Toolbox\docker-quickstart-terminal.ico"
Commands : "C:\Program Files\Git\bin\bash.exe" --login -i "C:\Program Files\Docker Toolbox\start.sh" -new_console:d:"C:\Program Files\Docker Toolbox""
```
* **Save settings**를 클릭하여 설정 저장
* **New console** 메뉴를 선택하거나 **왼쪽 하단 탭** 더블클릭
  ![](http://yonghochoi.github.io/images/docker/cmder_for_docker_2.png)
* console을 선택하는 드롭다운 메뉴에서 **docker** 선택 후 **Start**
