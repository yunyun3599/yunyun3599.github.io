# docker 강의 - chapter 1
## docker 
- 서비스가 의존하고 있는 패키지나 모듈 등이 여러가지 서비스가 올라가는 경우에 충돌이 일어날 수 있음  
- 서버 자원을 충분히 활용하지 못하는 문제가 생길 수 있음

=> 격리된 환경을 제공하여 한 서버에 여러 서비스를 올릴 수 있는 가상 환경 도구인 docker 필요 

## kubernetes 
- docker 같은 가상 환경 단위의 컨테이너들을 운영할 수 있도록 도와주는 오케스트레이션 도구. 
- 콜러스터의 관리/운영을 도와줌  

## docker 설치
- mac은 docker desktop 설치하면 됨

## kubectl, kustomize 설치
- kubectl은 kubernetes를 cli를 통해 관리할 수 있게 해주는 도구
- kustomize는 kubernetes 구성을 사용자 정의화하는 도구. (애플리케이션 구성 파일을 관리할 때 사용)

### 설치
kubectl 설치
```shell
brew install kubectl
```

kustomize 설치
```shell
brew install kustomize
```

## minikube
가상환경을 사용하여 쿠버네티스 클러스터를 구현  
사용 가능 드라이버: Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, VMware
> https://minikube.sigs.k8s.io/docs/start

### 설치
```sh
$ brew install minikube
```

### 클러스터 생성
```sh
minikube start --driver docker
```
![](/_draft/%08kubernetes_install.png)
```sh
minikube status
```
해당 minikube 위에서 클러스터가 잘 작동하고 있는 지 확인 가능 
![](/_draft/minkube_status.png)

```sh
kubectl cluster-info
```
kubernetes control plane이 어디에 떠있는 지 확인 가능
구성요소로 coreDNS도 실행되고 있음을 확인 가능
![](/_draft/kubectl_cluster-info.png)

kubectl이 kubernetes cluster와 통신하려면 설정파일이 필요한데, 이 설정 파일은 ~/.kube/config 파일임
![](/_draft/config.png)

**clusters**
- 관리할 클러스터 목록  
**context**   
- 인증 리스트
- 어떤 클러스터와 통신할 지 인증과 관련된 설정 진행  
- 내부 세부 힝목
    - cluster: 접속하게 될 클러스터 정보 기입
    - users: 인증 사용자 정보
- context는 cluster와 user 정보를 조합해 어떤 클러스터에 어떤 user로 접속할지 정보를 적어둔 부분

```sh
$ kubectl get nodes
```
kubectl이 접속하게되는 클러스터의 노드 정보 확인 가능
![](/_draft/kubectl_get_nodes.png)

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

### 추가내용
minikube addons  
- 애드온이란 쿠버네티스 클러스터에 필요한 다른 요소들을 따로 설치할 수 있도록 해주는 것

**addons 확인**
```sh
$ minikube addons
```
![](/_draft/minikube_addons.png)

**addons 제공 리스트 확인**
```sh
$ minikube addons list
```
![](/_draft/addons_list.png)

**addons 이용해 ingress 활성화**
```sh
$ minikube addons enable ingress
```
![](2022-11-21-23-51-54.png)

**minikube 노드에 ssh로 접속**
```sh
$ minikube ssl
```
![](/_draft/minikube_ssh.png)


```sh
$ minikube kubectl
```


a. 쿠버네티스 클러스터 상태 확인 
```sh
$ minikube
```