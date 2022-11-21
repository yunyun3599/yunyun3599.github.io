# docker 강의 - chapter 1
### docker 
- 서비스가 의존하고 있는 패키지나 모듈 등이 여러가지 서비스가 올라가는 경우에 충돌이 일어날 수 있음  
- 서버 자원을 충분히 활용하지 못하는 문제가 생길 수 있음

=> 격리된 환경을 제공하여 한 서버에 여러 서비스를 올릴 수 있는 가상 환경 도구인 docker 필요 

### kubernetes 
- docker 같은 가상 환경 단위의 컨테이너들을 운영할 수 있도록 도와주는 오케스트레이션 도구. 
- 콜러스터의 관리/운영을 도와줌  

### docker 설치
- mac은 docker desktop 설치하면 됨

### kubectl, kustomize 설치
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