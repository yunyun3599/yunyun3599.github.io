---
title:  "Jenkins를 사용한 CI/CD (2)"
excerpt: "Jenkins를 이용해 스프링 프로젝트애 CI/CD를 적용해봅니다."

categories:
  - Devops
tags:
  - [Devops, Jenkins]

toc: true
toc_sticky: true
 
date: 2023-01-03
last_modified_at: 2023-01-03
---
# Jenkins
젠킨스는 소프트웨어 개발 시 CI(continuous integration, 지속적 통합) 서비스를 제공하는 도구입니다.  

젠킨스는 지속적 통합을 할 수 있는 빌드 도구로 사용될 수 있으며, 커스텀 된 잡들을 생성하여 코드를 깃이나 버전 컨트롤 시스템과 연동하여 빌드, 컴파일, 단위 테스트, 통합테스트 등을 진행할 수 있습니다.  

젠킨스에서 제공하는 다양한 플러그인들을 추가적으로 이용하여 손쉽게 많은 기능들을 붙여 사용할 수도 있습니다.  

이 포스팅에서는 코드 작업을 할 로컬 환경과 젠킨스를 구동할 서버 환경을 구축하고, 깃허브를 이용하여 코드를 통합한 후 배포하는 실습을 진행해보력 합니다.  

## 인증 설정
로컬 서버, 깃허브, 젠킨스를 구동하는 서버를 이용하므로 아래와 같은 인증 설정을 해 두어야 합니다.  
- SSH key 생성 (로컬)
- Github 인증 설정 (ssh-key)
- 배포 서버 인증 설정 (deploy-key)

### 로컬에서 ssh 키 생성
로컬에서 pem 키를 저장할 디렉토리를 생성하고, 키를 생성합니다.  
```sh
$ mkdir -p ~/pems/jenkins
$ cd ~/pems/jenkins
```
키를 생성하기 위해 아래 명령어를 수행한 후 키 생성이 완료될 때까지 enter를 눌러줍니다.  
```sh
$ ssh-keygen -t ed25519 -a 100 -f ssh-key
```
결과로 ssh-key, ssh-key.pub 파일이 생성됩니다.  
ssh-key 키는 private 키로, jenkins에 인증설정을 할 때 사용할 것입니다.  
ssh-key.pub는 public 키로, github에 등록을 하게 됩니다.  

### 깃허브 ssh key 설정
키가 생성된 후에는 배포할 코드를 올려두는 github 레포지토리에 들어가 deploy key를 설정해야합니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/add_deploy_key.png)
레포지토리 페이지에서 settings에 들어가 Deploy keys 메뉴 선택 후 우측 상단의 Add deploy key를 선택합니다.  

![](/assets/img/2022-12/2022-12-20-jenkins/add_deploy_key_2.png)
그리고 난 후 위의 사진처럼 적당한 Title을 짓고, 앞서 발급한 ssh-key.pub 파일의 내용을 복사해 붙여넣고, allow write access를 체크한 후 키를 등록해 줍니다.  

깃허브 레포지토리에는 public key를 등록하고, 추후 설정할 jenkins의 credential에는 private key를 입력할 것입니다.  


## 서버 설정
저는 집에서 놀고 있는 노트북에 ubuntu를 깔아 진행하였습니다.  
만약 적당한 노트북이 없거나, 설정이 귀찮다면 aws에서 ec2 서버를 발급받아 사용하는 것을 추천합니다.  

집에 있는 노트북을 외부에서 접근 가능하도록 하기 위해 진행한 설정들은 [이 포스팅](https://yunyun3599.github.io/server/connect_remote_server/)에서 확인할 수 있습니다.   

### 서버 도커 설정
서버에 도커를 깔아야 하므로 아래 명령어를 하나씩 수행해 줍니다.  
```shell
$ sudo apt update
$ sudo apt install -y docker.io
$ sudo chmod 666 /var/run/docker.sock
```
설정이 완료되면 docker 관련 명령어가 잘 동작함을 알 수 있습니다.   


### Jenkins 설치
jenkins 관련 컨테이너가 볼륨으로 사용할 수 있도록 jenkins라는 이름의 디렉터리를 홈 디렉터리에 생성하도록 하겠습니다.  
```sh
$ mkdir jenkins
```
그리고 나서 젠킨스 컨테이너를 실행시키는 명령어를 수행합니다.  
```sh
$ docker run --name jenkins -d -p 8080:8080 -v ~/jenkins:/var/jenkins_home -u root jenkins/jenkins:latest
```
컨테이너가 뜨면 서버의 8080 포트를 통해 jenkins ui로 접근할 수 있습니다.  

jenkins 페이지에 로그인 하기 위해 Admin password를 확인할 때는 다음 명령어를 실행시키면 됩니다.  
```sh
$ docker exec -ti jenkins bash -c "cat /var/jenkins_home/secrets/initialAdminPassword"
```

위 명령어로 나온 결과를 `<서버ip>:8080`을 통해 들어간 jenkins ui에 입력하면 로그인을 할 수 있습니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/jenkins_ui.png)

로그인이 완료된 후 나오는 페이지에서 install suggested plugins를 선택하여 플러그인 설치를 진행합니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/jenkins_install_plugin.png)

플러그인 설치가 완료되면 admin user를 생성하는 페이지가 뜨게 됩니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/jenkins_set_admin_user.png)
원하는 값으로 admin user를 생성 후 나오는 url 설정까지 완료하면 jenkins 메인 화면이 나오게 됩니다.  
참고로 저는 url은 미리 생성되어 있는 값을 수정하지 않고 그대로 사용하였습니다.  


## 플러그인 설치
다음으로 추가로 사용할 플러그인을 설치하도록 하겠습니다.  
메인 화면에서 Jenkins 관리 > 플러그인 관리에 들어갑니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/plugin_setting.png)
플러그인 화면의 좌측 탭에서 Available plugins에 들어간 후 추가 플러그인을 설치하도록 합니다.  

새로 설치할 플러그인은 목록은 다음과 같습니다.  
- Jenkins DSL 플러그인: 젠킨스 파일을 작성하고, Groovy 언어 기반으로 사용하기 위한 플러그인 
    - Job DSL
- Jenkins Pipeline 플러그인: 각각의 job task를 연결
    - Pipeline: Deprecated Groovy Libraries
    - Pipeline: Declarative Agent API
    - Pipeline Utility Steps
    - Build Pipeline
- Github 플러그인: 개발 코드를 github 레포지토리에 연결하기 위함
    - GitHub Integration
    - GitHub Authentication
    - Pipeline: GitHub
    - Gradle Repo
- Docker 플러그인: micro 서비스 운영을 위해 특정 워커 노드에 코드를 배포할 수 있도록 이미지 생성 및 레지스트리 푸쉬를 돕는 플러그인
    - Docker
    - Docker Commons
    - Docker Pipeline
    - Docker API
    - docker-build-step
- SSH 플러그인: 젠킨스에서 명령어를 배포 서버에 원격으로 날리기 위함  
    - SSH
    - Publish Over SSH
    - SSH Pipeline Steps
    - SSH Agent

위의 목록들을 선택하고, install without restart 버튼을 통해 재시작 없이 플러그인들을 설치해줍니다.  

## 젠킨스 인증 설정
젠킨스 메인 페이지에서 Jenkins 관리 > Manage Credentials 메뉴로 들어가 인증 설정을 진행합니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/jenkins_credentials.png)
위 페이지에서 System 클릭 후 다음 페이지에서 Global credentials를 클릭합니다.  
그 결과 아래 페이지로 접근할 수 있게 됩니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/jenkins_credentials_add.png)

이 페이지에서 jenkins에서 github 레포지토리와 연결할 수 있는 ssh key를 credential로 추가해야 합니다.  

우측 상단의 파란색 +Add Credentials 버튼을 눌러 다음 화면과 같이 작성합니다.  
![](/assets/img/2022-12/2022-12-20-jenkins/add_github_credential.png)

밑에서 Add 버튼을 눌러 key값을 입력해 주도록 하겠습니다.  
여기에 들어가는 key는 앞서서 github 레포지토리의 deploy key에 등록했던 ssh-key.pub와 매칭되는 ssh-key 내용을 등록하면 됩니다.  
```sh
$ cat ~/pems/jenkins/ssh-key
```
위 명령어의 결과로 나온 값을 등록해 줍니다.  