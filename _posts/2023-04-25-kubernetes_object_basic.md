---
title:  "Kubernetes 오브젝트"
excerpt: "쿠버네티스의 오브젝트 개념에 대해 알아봅니다."

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-04-25
last_modified_at: 2023-04-25
---
# 쿠버네티스 오브젝트 

## 쿠버네티스 오브젝트란
쿠버네티스를 통해 사용자는 어떤 애플리케이션을 어느 위치에 얼마나 어떤 방식으로 배포할 지를 설정할 수 있습니다.  

이러한 정보들을 `쿠버네티스 오브젝트`로 작성하게 되는데, 이 때 형식은 yaml 형식을 이용하며 설정된 정보는 REST API를 통해 API server에 전달됩니다.  
이 때 어떤 오브젝트를 생성하고자 하는 지에 따라 정의할 수 있는 속성은 달라집니다.  

또한 오브젝트는 사용자가 의도한 바를 작성한 것이기 떄문에 결과적으로는 클러스터의 상태를 결정하게 됩니다.  
즉, 사용자가 어떻게 쿠버네티스 오브젝트를 정의하느냐에 따라 쿠버네티스의 상태가 결정되는 것입니다.  

### 쿠버네티스 오브젝트
쿠버네티스 오브젝트는 `쿠버네티스 클러스터를 이용해 애플리케이션을 배포하고 운영하기 위해 필요한 모든 쿠버네티스 리소스`입니다.  

쿠버네티스 오브젝트가 될 수 있는 것들의 목록은 아래와 같습니다.  
1. Pod: 어떤 애플리케이션을 배포할 지
2. ReplicaSet: 얼마나 많이 배포할 지
3. Node, Namespace: 어디에 배포할 지 (Namespace: 클러스터 내에서 리소스를 논리적으로 그룹핑하기 위해 설정해 놓은 단위)
4. Deployment: 어떤 방식으로 배포할 지
5. Service, Endpoints: 트래픽을 어떻게 로드밸런싱할 것인지

### 쿠버네티스 오브젝트 작성
쿠버네티스 오브젝트의 기본적인 모양은 아래와 같습니다.  

```yaml
apiVersion: apps/v1     # 오브젝트 생성 시 사용하는 API 버전
kind: Deployment        # 생성하고자 하는 오브젝트 종류
metadata:               # 오브젝트의 이름이나 기본적인 메타 성격의 정보 작성 (e.g. name, resourceVersion, labels, namespace 등)
    name: nginx-deployment
spec:                   # 사용자가 원하는 오브젝트 상태
    selector:
        matchLabels:
            app: nginx
    replicas: 2
    template:
        metadata:
            labels:
                app: nginx
        spec:
            containers:
            - name: nignx
              image: nginx:1.14.2
              ports:
              - containerPort: 80
```

쿠버네티스 오브젝트의 spec을 작성하여 쿠버네티스에게 요청을 보내면 쿠버네티스는 여러 컴포넌트의 협력으로 애플리케이션을 배포하고, 컨테이너 상태에 대한 health check를 진행하며 pod 및 리소스의 상태를 계속해서 체크합니다.   
그리고 이렇게 저장된 상태를 사용자들이 조회할 수 있도록 status라는 속성을 추가해줍니다.  


status는 시간대별로 쿠버네티스에서 일어난 일에 대한 메세지나 일어난 이유 등에 대한 내용 등을 담고 있습니다.  
오브젝트가 쿠버네티스 클러스터에 생성되면 쿠버네티스는 오브젝트 정보에 status 필드를 추가하고 아래와 같이 현재 실행 중인 오브젝트의 상태 정보를 알려주는 것 입니다.  
```yaml
status:
    availableReplicas: 2
    conditions: 
    - lastTransitionTime: '2023-02-25T12:34:56Z'
      lastUpdateTime: '2023-02-25T12:34:56Z'
      message: Deployment has minimum availability
      reason: MinimumReplicasAvailable
      status: 'True'
      type: Available
    - lastTransitionTime: '2023-02-25T12:12:34Z'
      lastUpdateTime: '2023-02-25T12:12:34Z'
      message: ReplicaSet "~~~" has successfully progressed
      reason: NewReplicasAvailable
      status: 'True'
      type: Processing
observedGeneration: 1
readyReplicas: 2
replicas: 2
updatedReplicas: 2
```

결과적으로 쿠버네티스는 사용자가 작성한 spec과 현재 클러스터 상태를 담은 status 값을 일치시키기 위한 방향으로 동작합니다.  

오브젝트 작성부터 쿠버네티스 동작까지의 과정을 간단히 나타내면 아래와 같습니다.  
1. 배포하기 원하는 상태를 반영하여 쿠버네티스 오브젝트 YAML 파일을 작성하고, 그 안에 spec을 정의합니다.
2. 쿠버네티스 API를 이용해 쿠버네티스에 생성을 요청합니다.  
3. 쿠버네티스 API server가 오브젝트 파일의 spec 파일을 읽고 오브젝트를 생성합니다. 
4. 쿠버네티스 ControllerManager가 spec과 status를 비교하면서 계속해서 조정하고 상태를 업데이트합니다.  
