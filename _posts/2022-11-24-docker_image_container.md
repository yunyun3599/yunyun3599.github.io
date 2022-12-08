---
title:  "Docker Image, Container"
excerpt: "Docker 이미지와 컨테이너 관련 기본적인 명령어와 개념을 알아봅니다.  "

categories:
  - Docker
tags:
  - [Docker, Devops]

toc: true
toc_sticky: true
 
date: 2022-11-24
last_modified_at: 2022-11-24
---

# Docker 구성 요소
## 1. Client
도커 관련 명령어 질의하는 역할을 합니다.  
Client에서 많이 사용되는 명령어는 다음과 같습니다.  
- docker build
- docker pull
- docker run 
- ...

## 2. Docker Host
Docker daemon (=docker engine)을 실행하는 역할을 합니다.  
도커 호스트에서 Container와 Image를 관리합니다.  
- Docker Host에 위치한 image는 직접 빌드하거나 pull을 통해 레지스트리에서 가져오는 방법 2가지로 얻을 수 있습니다.  
- image를 실행하면 컨테이너가 되어 실제적인 작동을 수행합니다.  

## 3. Docker Registry
docker image들이 저장되어 있는 곳입니다.  
pull을 통해 local로 가지고 와서 사용할 수 있습니다.  

<br>

# 도커 이미지와 컨테이너 
## 이미지
**이미지:** `컨테이너를 생성할 때 필요한 요소`    
이미지에는 컨테이너에서 필요로 하는 바이너리와 의존성이 설치되어 있습니다.  
이미지는 여러 계층 구조(layer)로 이루어져 있습니다.  

### 이미지 명명 규칙
이미지의 이름은 `저장소이름/이미지이름:태그` 로 표기합니다.  
여기서 저장소 이름과 태그는 생략이 가능 합니다.
태그가 생략된 경우는 가장 최근 버전(latest)을 가져옵니다.  
저장소 이름이 생략된 경우는 기본 저장소인 도커 허브(dockerhub)로 인식합니다.  

**이미지 이름 예시**  
> bitnami/python:3.9.15

### 도커 이미지 저장소  
도커 이미지 저장소(도커 이미지 레지스트리)는 서버 어플리케이션으로 도커 이미지를 관리/공유할 수 있게 해줍니다.

도커 이미지 저장소에는 다음과 같이 2가지 종류가 있습니다. 
- public registry: 누구나 사용할 수 있는 public 이미지를 저장 (e.g dockerhub)
- private registry: 비공개형 도커 이미지 저장소 (e.g. AWS ECR - Elastic Container Registry)

## 컨테이너
컨테이너는 이미지를 통해 만들어진 프로세스라고 볼 수 있습니다.    
컨테이너는 다른 컨테이너나 호스트로부터 격리된 환경을 갖습니다.  
컨테이너 내에서 작업한 내용은 이미지에 반영되지 않기 때문에 컨테이너 중지 시에는 모든 변경 사항을 잃게 됩니다.   

### 컨테이너 라이프 사이클  
도커 컨테이너 라이프 사이클 종류는 다음과 같습니다.  
1. Created: 컨테이너 생성
2. Running
    - Create한 컨테이너를 start함으로써 실행
    - image를 run 함으로써 실행
3. Paused: 실행 컨테이너를 일시 중지
4. Stopped: 컨테이너 중지
5. Deleted: 컨테이너가 created나 stopped 상태일 때 컨테이너를 삭제

### 컨테이너 관련 명령어
1. 컨테이너 생성  
    - docker run: 컨테이너 생성 및 시작을 한번에 함
        ```sh
        $ docker run [image]
        ```
    - dodcker create & start: 컨테이너 생성 후 시작
        ```sh
        $ docker create [image]
        ```
        ```sh
        $ docker start [container_이름 or container_id]
        ```
    - docker container run 옵션   

        |옵션|내용|
        |:---:|:---:|
        |-i|호스트 입력과 컨테이너 입력을 연결 (interactive)|
        |-t|tty 할당. 터미널 명령을 정상 수행하도록 함|
        |--rm|컨테이너 실행 종료되면 자동 삭제|
        |-d|컨테이너 백그라운드 동작|
        |--name [컨테이너이름]|컨테이너 이름 지정|
        |-p|호스트  -컨테이너 포트 바인딩|
        |-v|호스트 - 컨테이너 볼륨 바인딩|

        가장 많이 사용되는 -i -t옵션에 대해 좀 더 알아보도록 하겠습니다.  
        ```sh
        $ docker run ubuntu
        ```
        위의 명령을 실행하면 바로 컨테이너가 종료됩니다.  
        그 이유는 ubuntu의 기본 명령은 bash이므로 입력받을 것이 따로 없어서 바로 종료된 것이라고 볼 수 있습니다.  
        이 때 호스트의 입력과 컨테이너 입력을 연결해주는 -i 옵션과 터미널 명령을 수행하도록 하는 -t 옵션을 함께 줘서 아래와 같이 컨테이너를 실행시키면 컨테이너 실행과 동시에 컨테이너의 bash에 연결되어 컨테이너가 종료되지 않습니다.    
        ```sh
        $ docker run -ti ubuntu 
        ```
        ![](/assets/img/2022-11/2022-11-24-docker_image_container/docker_run_ti_ubuntu.png)
        위의 상태에서 exit 을 통해서 빠져나오면 빠져나오자 마자 컨테이너가 종료됩니다.  
        컨테이너 종료 없이 빠져나오려면 `ctrl + p + q`를 누르면 됩니다.  
                
2. 컨테이너 조회
    - 실행중인 컨테이너 조회
        ```sh
        $ docker ps
        ```
    - 전체 컨테이너 조회
        ```sh
        $ docker ps -a
        ```
    - 특정 컨테이너 상세 정보 조회
        ```sh
        $ docker inspect [container_name]
        ```
3. 컨테이너 일시 정지
    - 컨테이너 일시 중지
        ```sh
        $ docker pause [container_name]
        ```
    - 컨테이너 재개
        ```sh
        $ docker unpause [container_name]
        ```
4. 컨테이너 종료
    - 컨테이너 종료
        ```sh
        $ docker stop [container]
        ```
    - 컨테이너 강제 종료
        ```sh
        $ docker kill [container]
        ```
5. 컨테이너 삭제
    - 컨테이너 삭제 (실행중인 컨테이너는 삭제되지 않음)
        ```sh
        $ docker rm [container]
        ```
    - 컨테이너 강제 종료 후 삭제
        ```sh
        $ docker rm -f [container]
        ```
    - 컨테이너 실행 완료 후 삭제
        ```sh
        $ docker run --rm [image]
        ```
    - 중지된 모든 컨테이너 삭제
        ```sh
        $ docker container prune
        ```


## Dockerfile, 이미지, 컨테이너  
Dockerfile을 작성해 build하면 **이미지**가 생성 됩니다.  
이미지를 run하면 **container**가 생성됩니다.   
