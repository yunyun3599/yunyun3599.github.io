---
title:  "Docker 및 Minikube 설치"
excerpt: "Docker를 설치하고 kubernetes를 사용할 수 있도록 minikube를 설치합니다. "

categories:
  - Docker
tags:
  - [Docker, Kubernetes, Devops, Minikube]

toc: true
toc_sticky: true
 
date: 2022-11-23
last_modified_at: 2022-11-23
---


# docker 
도커는 각종 서비스들을 프로세스 격리 기술들을 사용해 분리된 환경에서 컨테이너로 실행하고 관리하는 오픈 소스 프로젝트입니다.  

## Docker의 필요성
서버 내에 필요한 서비스를 그냥 올리면 편할 것 같은데 도커가 왜 필요할까요?  
다음과 같은 문제 상황을 생각해 봅시다.  
- 서비스가 의존하고 있는 패키지나 모듈 등이 여러가지 서비스가 올라가는 경우에 충돌이 일어남.
- 서버 자원을 충분히 활용하지 못하는 문제가 발생함  

하나의 서버에 여러가지 서비스를 올리려고 할 때 위와 같은 문제가 생긴다면, 격리된 환경을 제공하지 않고는 좀처럼 해결하기 어렵습니다.  

**이에 따라 격리된 환경을 제공하여 한 서버에 여러 서비스를 올릴 수 있는 가상 환경 도구인 docker의 필요성이 나타나게 된 것입니다.**  

## docker 설치
Mac에 Docker를 설치하는 방법은 간단합니다.  
Docker Desktop이라는 응용 프로그램을 설치하면 됩니다.  
[이 사이트](https://www.docker.com/products/docker-desktop/)에서 본인의 환경에 맞는 파일을 다운받아 주세요.

<br>

# kubernetes 
- 쿠버네티스는 docker 같은 가상 환경 단위의 컨테이너들을 운영할 수 있도록 도와주는 오케스트레이션 도구입니다.  
- 쿠버네티스는 클러스터의 관리/운영을 도와주는 역할을 수행합니다.  

## kubectl, kustomize
- kubectl은 kubernetes를 cli를 통해 관리할 수 있게 해주는 도구입니다.
- kustomize는 kubernetes 구성을 사용자 정의화하는 도구로 애플리케이션 구성 파일을 관리할 때 사용합니다. 

### kubectl 설치
```shell
brew install kubectl
```

### kustomize 설치
```shell
brew install kustomize
```

## minikube
minikube는 가상환경을 사용하여 쿠버네티스 클러스터를 구현하는 데 도움을 주는 도구 입니다.  
minikube에서 사용 가능한 드라이버 목록은 다음과 같습니다.  
`Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, VMware`
> minikube와 관련된 세부 내용은 [이 링크](https://minikube.sigs.k8s.io/docs/start)에서 추가적으로 확인 가능합니다. 

### minikube 설치
```sh
$ brew install minikube
```

### minikube를 이용해 클러스터 생성
- **minikube 시작**
```sh
minikube start --driver docker
```
![](/assets/img/2022-11/2022-11-23-docker_install/kubernetes_install.png)

- **minikube 상태 확인**
```sh
minikube status
```
위의 명령어를 통해 해당 minikube 위에서 클러스터가 잘 작동하고 있는 지 확인 가능합니다.  
![](/assets/img/2022-11/2022-11-23-docker_install/minkube_status.png)

- **minikube로 띄운 클러스터의 정보 확인**
```sh
kubectl cluster-info
```
kubernetes control plane이 어디에 떠있는 지 확인 가능합니다.  
구성요소로 coreDNS도 실행되고 있음을 확인할 수 있습니다.  
![](/assets/img/2022-11/2022-11-23-docker_install/kubectl_cluster-info.png)

- **kubectl 통신 설정**  
kubectl이 kubernetes cluster와 통신하려면 설정파일이 필요한데, 이 설정 파일은 ~/.kube/config 파일입니다. 
![](/assets/img/2022-11/2022-11-23-docker_install/config.png)

  - **clusters**
    - 관리할 클러스터 목록  
  - **context**   
    - 인증 리스트
    - 어떤 클러스터와 통신할 지 인증과 관련된 설정 진행  
    - 내부 세부 항목
        - cluster: 접속하게 될 클러스터 정보 기입
        - users: 인증 사용자 정보
    - context는 cluster와 user 정보를 조합해 어떤 클러스터에 어떤 user로 접속할지 정보를 적어둔 부분

- **노드 정보 확인**  
```sh
$ kubectl get nodes
```
kubectl이 접속하게되는 클러스터의 노드 정보를 확인할 수 있습니다. 
![](/assets/img/2022-11/2022-11-23-docker_install/kubectl_get_nodes.png)

### minikube 기본 사용법
0. 클러스터 시작
```sh
$ minikube start
```
1. 클러스터 상태 확인 
```sh
$ minikube status
```
2. 클러스터 중지
```sh
$ minikube stop
```
3. 클러스터 삭제
```sh
$ minikube delete
```
4. 클러스터 일시 중지
```sh
$ minikube pause
```
5. 클러스터 재개
```sh
$ minikube unpause
```

### minikube addons
minikube addons  
- 애드온이란 쿠버네티스 클러스터에 필요한 다른 요소들을 따로 설치할 수 있도록 해주는 것입니다  

**addons 확인**
```sh
$ minikube addons
```
![](/assets/img/2022-11/2022-11-23-docker_install/minikube_addons.png)

**addons 제공 리스트 확인**
```sh
$ minikube addons list
```
![](/assets/img/2022-11/2022-11-23-docker_install/addons_list.png)

**addons 이용해 ingress 활성화**
```sh
$ minikube addons enable ingress
```
![](/assets/img/2022-11//2022-11-23-docker_install/addons_enable_ingress.png)

### minikube 노드에 ssh로 접속
```sh
$ minikube ssh
```
![](/assets/img/2022-11/2022-11-23-docker_install/minikube_ssh.png)

### 로컬의 kubectl과 minikube의 kubectl 버전이 다른 경우
두 kubectl의 버전이 다르다면 이로 인해 문제가 날 수 있습니다.  
이런 일을 방지하기 위해 `minikube kubectl [수행할기능]` 이라고 명령어를 치면 로컬에 깔린 kubectl이 아닌 minikube의 kubectl 버전으로 kubectl을 사용할 수 있게 됩니다.  
```sh
$ kubectl version
```
![](/assets/img/2022-11/2022-11-23-docker_install/kubectl_version.png)
> Client Version과 Server Version이 다름  

```sh
$ minikube kubectl version
```
![](/assets/img/2022-11/2022-11-23-docker_install/minikube_kubectl_version.png)
> minikube kubectl version을 한 결과 Client Version과 Server Version이 같아짐
