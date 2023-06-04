---
title:  "Kubernetes 환경 변수 설정"
excerpt: "kubernetes의 pod에 환경 변수를 설정하는 방법에 대해 알아봅니다."

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-06-05
last_modified_at: 2023-06-05
---
# Pod - 컨테이너로 환경변수 전달

이번 포스트에서는 kubernetes를 통해 pod를 띄울 때 pod 내의 컨테이너에 환경변수를 설정하는 방법에 대해 알아봅니다.   
포스팅을 통해 수행해볼 작업은 아래와 같습니다.    
1. Pod YAML 파일에 컨테이너에서 사용할 환경변수를 선언
2. Pod 실행 후 컨테이너에 접속하여 선언한 환경변수가 잘 설정되었는 지 확인

## 환경변수 세팅 프로세스
환경변수를 세팅하기 위해 수행할 단계는 아래와 같습니다.  
1. Pod 선언과 환경변수 설정  
2. Pod 생성, 배포
3. 컨테이너에서 사용할 Image pull
4. Pod IP 할당 및 컨테이너 실행 확인  
5. 생성된 컨테이너 포트포워딩
6. Pod로 트래픽 전송
7. Http 서버 응답 확인  
8. 컨테이너 환경변수 목록 확인  

### 설정할 환경 변수  
환경변수는 아래 6가지를 설정할 예정입니다.  
- GREETING
- NODE_NAME
- NODE_IP
- NAMESPACE_NAME
- POD_NAME
- POD_IP

환경 변수 설정 후 아래와 같은 형태로 환경변수 값을 확인하기위한 응답을 받도록 하겠습니다.  
```json
{
  "greeting": "Welcome to Kubernetes",
  "host": {
    "name": "gke-node",
    "ip": "10.128.0.15"
  },
  "namespace": {
    "name": "default"
  },
  "pod": {
    "name": "hello-app",
    "ip": "10.76.0.143"
  }
}
```

## Pod 이름, 컨테이너 이름, 이미지, 포트 설정
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-app
spec:
  containers:
    name: hello-app
    image: devchloe/hello-app:1.0
    ports:
    - containerPort: 3000
    env: hello-app          # 환경 변수 설정
    - name: STUDENT_NAME
      value: 홍길동
    - name: GREETING
      value: $(STUDENT_NAME)님, 쿠버네티스 공부 화이팅!
    # 쿠버네티스 오브젝트로부터 환경변수 값을 설정
    - name: NODE_NAME       # Pod가 배포된 노드의 이름을 설정
      valueFrom:            # 쿠버네티스 오브젝트로부터 환경변수 값을 얻고자 함
        fieldRef:           # Pod spec, status의 field를 환경변수 값으로 참조
          fieldPath: spec.nodeName    # 참조할 field 경로 선택
    - name: NODE_IP
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
```

## Pod 배포를 위한 명령어  
- Pod 생성
  ```sh
  kubectl apply -f <yaml 파일 경로>
  ```
- Pod 실행 및 IP 확인
  ```sh
  kubectl get pod -o wide     # -o는 output 형태에 대한 옵션으로 wide를 선택하면 상세 정보를 얻을 수 있음
  ```
- Pod 종료
  ```sh
  kubectl delete pod --all or kubectl delete pod <pod-name>
  ```
- 컨테이너 IP 확인
  ```sh
  kubectl exec <pod-name> [-c <container-name>] --ifconfig eth0
  ```
- 컨테이너 환경변수 확인
  ```sh
  # 컨테이너에서 실행할 명령어를 줄 때는 꼭 double dash(--)를 주어야 함. dash를 하나만 주면 kubernetes 명령어로 인식하여 컨테이너로 명령어가 가지 않음
  kubectl exec <pod-name> --env     # 컨테이너 정보를 주지 않으면 가장 먼저 생성된 컨테이너로 명령을 실행하게 됨. (env 명령어를 실행)
  ```
- 포트 포워딩
  ```sh
  kubectl port-forward <pod-name> <host-port>:<container-port>    # pod 내의 컨테이너를 포트포워딩하는 명령어 (로컬의 호스트 포트를 pod의 컨테이너 포트로 포워딩)
  ```


## Pod 템플릿 선언 시 컨테이너에 환경변수 설정하기
가장 먼저 kubectl apply를 이용해 hello-app을 배포하도록 하겠습니다.  
이를 위해서는 hello-app이라는 yaml파일이 팔요합니다.  
hello-app.yaml파일의 내용은 아래와 같습니다.  
```yaml
# hello-app.yaml
# Pod API 버전: v1
# Pod 이름: hello-app
# Pod 네임스페이스: default
# 컨테이너 이름/포트: hello-app:8080
# 도커 이미지: yoonjeong/hello-app:1.0

apiVersion: v1
kind: Pod
metadata:
  name: hello-app
  labels:
    name: hello-app
  namespace: default     # namespace는 클러스터 노드를 논리적인 단위로 구분하기 위한 용도. 설정하지 않으면 default가 기본으로 설정됨
spec:
  containers:
  - name: hello-app
    image: yoonjeong/hello-app:1.0
    ports:
    - containerPort: 8080   # 사용할 이미지에 사전적으로 8080 포트가 노출되어있음
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NAMESPACE_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: NODE_IP
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
    - name: STUDENT_NAME
      value: 최윤재
    - name: GREETING
      value: 안녕하세요, $(STUDENT_NAME)님!
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"

```

위와 같이 hello-app.yaml 파일을 작성 후 `kubectl apply -f <파일명>` 명령어를 통해 pod를 생성합니다.   
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/create_pod.png)

`kubectl get pod -o wide` 명령어를 통해 생성한 Pod 정보를 확인해보도록 하겠습니다.  
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/kubectl_get.png)
결과에서 Ready는 컨테이너 수를 의미하며, 총 1개의 컨테이너 중 1개가 사용자의 요청을 받을 준비가 되었다는 것을 알 수 있습니다.  
또한 Pod의 IP와 pod가 배포된 node도 알 수 있습니다.   
만약 더 상세한 정보를 json 형식으로 보고싶다면 `kubectl get pod -o json` 명령어를 이용할 수 있습니다.  

### 생성된 컨테이너 이용
다음으로는 pod에 생성된 컨테이너 내부로 들어가보도록 하겠습니다.   
컨테이너 내부에 어떠한 명령어를 실행시켜보기 위해서는 double dash(--) 뒤에 원하는 명령어를 치면 됩니다.  

컨테이너의 /etc/hosts 내용을 보고싶다면 다음 명령어를 치면 됩니다.  
```sh
$ kubectl exec hello-app -- cat /etc/hosts
```
이렇게 되면 hello-app 이라는 pod 내에 있는 컨테이너 내에서 `--` 뒤의 명령어가 수행됩니다.  
만약 하나의 pod에 여러 컨테이너가 떠 있는 경우라면 `-c <컨테이너명>` 옵션을 추가하여 컨테이너에 명령어를 수행시킬 수 있습니다.  

![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/kubectl_exec_cat.png)
`/etc/hosts` 파일 출력 결과 앞에서 `kubectl get pod -o wide` 명령어를 통해 확인했던 ip와 동일한 ip로 설정되어있음을 확인할 수 있습니다.  

또한 앞서 설정한 환경변수가 잘 적용되었는지 보기 위해 `env` 명령어를 수행시킨 결과 원하던 대로 환경변수가 잘 세팅되어있음을 확인할 수 있습니다.  
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/kubectl_exec_env.png)

컨테이너가 listening하고있는 포트를 확인하기 위해 `netstat` 명령어도 보내보도록 하겠습니다.  
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/kubectl_exec_netstat.png)
사용할 포트로 지정했던 8080 포트를 listening 중임을 확인할 수 있습니다.  

여기서 8080포트를 포트포워딩하여 트래픽을 연결해 외부에서 컨테이너의 8080 포트를 통해 서비스에 접근할 수 있도록 하는 작업이 필요합니다.  
포트포워딩을 하기 위한 명령어는 다음과 같습니다.  
```sh
$ kubectl port-forward hello-app 8080:8080
```
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/kubectl_port-forward.png)

포워딩된 포트를 통해 curl을 이용하여 서비스에 접근해보도록 하겠습니다.  
![](/assets/img/2023/04/2023-04-09-kubernetes_pod_env_setting/curl_get_response.png)

컨테이너에서 띄운 서비스에서 미리 요청이 온다면 세팅한 환경변수를 포함하여 응답을 주도록 정의해두었기 때문에, 앞서 설정한 환경변수들이 포함된 리스폰스가 잘 오는 것을 확인할 수 있습니다.  

지금까지 생성한 리소스들을 삭제함으로써 더이상 쓰지 않는 pod를 제거하기 위해서는 아래 명령어를 사용할 수 있습니다.  
```sh
$ kubectl delete pod --all      # kubernetes 클러스터상의 모든 pod를 삭제
$ kubectl delete pod hello-app  # hello-app pod를 삭제
```