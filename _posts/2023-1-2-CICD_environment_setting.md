---
title:  "Jenkins를 사용한 CI/CD (1)"
excerpt: "Jenkins를 이용해 CI/CD를 적용하기 위해 서버 환경을 구축합니다."

categories:
  - Devops
tags:
  - [Devops, Jenkins]

toc: true
toc_sticky: true
 
date: 2023-01-02
last_modified_at: 2023-01-02
---
# CI/CD를 위한 서버 환경 세팅
*아래의 모든 설정들은 ubuntu 환경을 기반으로 진행하였습니다.*  

## 패키지 설치
### apt 
필요한 패키지를 설치하기에 앞서 apt upgrade를 수행합니다.  
```shell
$ sudo apt update
```

### 자바 설치
프로그래밍에 필요한 자바를 설치합니다.  
```shell
$ sudo apt install -y default-jdk
```

## 도커 설치
### 설치 명령
가상 환경을 통해 배포할 수 있도록 도커를 설치합니다.  
```shell
$ sudo apt install docker.io
```

### 동작 확인 
도커가 정상 동작하는 지 확인해봅니다.  
```shell
$ sudo systemctl status docker
```

### 권한 부여
도커를 사용할 수 있도록 권한을 부여해 줍니다.  
```shell 
$ sudo chmod 666 /var/run/docker.sock
```

## Gradle
Gradle은 빌드 자동화 시스템으로, Java, C, C++, 파이썬 등 여러 언어를 지원합니다.  
Gradle을 이용해 다양한 속성과 설정을 빌드할 때 자동화 할 수 있습니다.  

### Gradle을 통한 프로젝트 빌드
gradle을 통해 프로젝트를 빌드할 때 아래와 같은 과정을 거치게 됩니다.  
1. 초기화: Gradle에 빌드할 프로젝트를 설정 / 생성
2. 구성: 프로젝트 객체 구성. 프로젝트 코드를 작성하고 여러 라이브러리 의존성 관리 및 속성을 스크립트로 관리. 빌드 스크립트 및 Task 작성
3. 실행: Gradle의 모든 Task를 통합하고 빌드 실행  

### Gradle 기반 프로젝트 구조
gradle을 기반으로 한 프로젝트는 아래 디렉터리 구조를 갖습니다.  
```
├── app
|  ├── build.gradle
|  └── src
|      ├── main
|      |   ├── java
|      |   |   └── com
|      |   |       └── example
|      |   |           └── App.java
|      |   └── resources
|      └── test
|          ├── java
|          |   └── com
|          |       └── example
|          |           └── AppTest.java 
|          └── resources
├── gradle          # wrapper 실행을 위한 jar파일과 properties 파일이 있음
|   └── wrapper
|       └── gradle-wrapper.jar
|       └── gradle-wrapper.properties
├── gradlew         # gradle을 실행하고자하는 os에 gradle이 설치되어있지 않아도 동작할 수 있도록 해주는 파일
├── gradlew.bat
└── settings.gradle
```

### Gradle 설정 파일 종류
- `build.gradle`: 각종 플러그인, 의존성, 라이브러리 저장소 등을 설정할 수 있도록 해주는 파일  
- `settings.gradle`: 프로젝트의 구성 정보 파일
- `gredlew`: 리눅스나 맥os에서 사용되는 gradle 실행 쉘 파일
- `gradlew.bat`: 윈도우에서 사용되는 실행 배치 스크립트 파일
- `gradle 디렉토리`: gradle-wrapper 관련 파일들이 존재. 해당 파일들은 gradlew 파일을 실행시킬 때 사용되는 파일  

### gradle 설치
mac os의 경우 brew를 이용해 설치할 수 있습니다.  
```sh
$ brew install gradle
```

ubuntu의 경우 아래 명령어들을 통해 gradle을 설치할 수 있습니다.  
1. Ubuntu에 PPA 저장소 추가
    ```sh
    $ sudo apt -y install vim apt-transport-https dirmngr wget software-properties-common
    $ sudo add-apt-repository ppa:cwchien/gradle
    ```
2. Ubuntu 리눅스 패키지 업데이트 및 Gradle 설치
```sh
$ sudo apt-get update
$ sudo apt -y install gradle
```

### gradle 프로젝트 생성 및 빌드
gradle을 이용한 springboot 프로젝트 생성하기 위한 명령어는 다음과 같습니다.  
```sh
$ gradle init --dsl=groovy --type=java-application --test-framework=junit --package=com.test --project-name=test-docker-spring-boot
```

gradle 빌드 명령어는 아래와 같습니다.  
```sh
$ gradle build --info
```
빌드 명령의 결과로 생성되는 jar파일은 `프로젝트디렉터리/app/build/libs` 디렉터리 하에 있습니다. 
