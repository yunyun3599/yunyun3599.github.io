---
title:  "Docker 및 Minikube 설치"
excerpt: "Docker를 설치하고 kubernetes를 사용할 수 있도록 minikube를 설치합니다. "

categories:
  - Docker
tags:
  - [Docker, Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2022-11-16
last_modified_at: 2022-11-16
---

# Docker 구성 요소
## Client
- docker build
- docker pull
- docker run 
- ...
도커 관련 명령어 질의

## Docker Host
Docker daemon (=docker engine)
도커 호스트에서 Container와 Image를 관리
- image는 직접 빌드 or pull을 통해 레지스트리에서 가져오는 방법 2가지로 얻을 수 있음  
- image를 실행하면 컨테이너가 되는 것

## Docker Registry
- docker image들이 저장되어 있는 곳
- pull을 통해 local로 가지고 와서 사용할 수 있음  

<br>

# 도커 이미지와 컨테이너 
## 이미지
컨테이너를 생성할 때 필요한 요소  
컨테이너에서 필요로 하는 바이너리와 의존성이 설치되어 있음.  
여러 계층 구조로 이루어져 있음

### 이미지 명명 규칙
`저장소이름/이미지이름:태그` 로 표기 
여기서 저장소 이름과 태그는 생략 가능  
태그가 생략된 경우는 가장 최근 버전을 가져옴  
저장소 이름이 생략된 경우는 기본 저장소인 도커 허브로 인식

이미지 이름 예시  
> bitnami/python:3.9.15

### 도커 이미지 저장소  
서버 어플리케이션으로 도커 이미지를 관리/공유할 수 있게 해줌  
- public registry: 누구나 사용할 수 있는 public 이미지를 저장 (e.g dockerhub)
- private registry: 비공개형 도커 이미지 저장소 (e.g. AWS ECR - Elastic Container Registry)

## 컨테이너
이미지를 통해 만들어진 프로세스  
다른 컨테이너나 호스트로부터 격리된 환경  
컨테이너 내에서 작업한 내용은 컨테이너를 생성한 이미지에 반영되지 않음  
이미지를 여러 방법으로 실행하여 여러 컨테이너를 띄울 수 있음  

## Dockerfile, 이미지, 컨테이너  
Dockerfile을 작성해 build하면 **이미지** 생성  
이미지를 run하면 **container** 생성  
