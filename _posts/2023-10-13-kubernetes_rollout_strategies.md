---
title:  "Kubernetes Deployment 배포전략"
excerpt: "Kubernetes의 Deployment의 배포 전략에 대해 알아봅니다."

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-10-13
last_modified_at: 2023-10-13
---

# Deployment 배포 전략  

## Recreate 배포
Recreate 배포는 개발할 때 주로 사용하는 배포 전략입니다.  
Recreate 전략을 개발 시에 사용하는 이유는 Recreate 전략을 사용하여 신규 버전을 배포하면 Deployment는 이전 Pod를 모두 종료하고 새로운 Pod를 replicas만큼 생성하기 때문입니다.  
즉, 이전 버전의 Pod를 모두 삭제한 후에 새로운 Pod들을 생성하므로 필연적으로 Pod가 하나도 존재하지 않는 시점이 생겨 서비스 다운타임이 발생하기 때문에 개발 단계에서 주로 사용하며 운영 환경에서는 사용하기 적합하지 않습니다.   

![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/recreate_strategy.png)

Recreate 배포의 동작 방식을 알아보기 위해 `my-app`이라는 deployment를 배포했다가 Image와 Label을 업데이트 하도록 하겠습니다.  

### Deployment 생성  
Deployment를 배포하기 위해 아래 스펙대로 `deployment.yml` 파일을 작성합니다.  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
        app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: my-app
        project: deployment_sample
        env: local
    spec:
      containers:
      - name: my-app
        image: yoonjeong/my-app:1.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"

```  

이 때 rollout 전략을 적용하기 위해 다음 부분이 추가되었음을 알 수 있습니다.  
```yml
strategy:
  type: Recreate
```

deployment를 아래 명령어를 통해 생성합니다.  
```sh
$ kubectl apply -f deployment.yml
```

deployment의 배포 진행 / 완료 상태를 확인하기 위해 아래 명령어를 사용합니다.  
```sh
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollout_status_deployment.png)

app=my-app 레이블을 갖는 모든 리소스를 조회하기 위해서는 아래 명령어를 사용할 수 있습니다.  
```sh
$ kubectl get all -l app=my-app -o wide --show-labels
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/get_all_-l_app=my-app.png)

### 컨테이너 이미지와 레이블 변경  
위에서 생성한 deployment의 컨테이너 이미지와 레이블을 변경해서 재배포해보도록 하겠습니다.  
앞서 작성한 `deployment.yml` 파일을 아래와 같이 변경해줍니다.  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
        app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: my-app
        project: deployment_sample
        env: local
        version: v2
    spec:
      containers:
      - name: my-app
        image: yoonjeong/my-app:2.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```
`labels`에 `version: v2`를 추가했으며, `image`의 태그를 `2.0`으로 변경했습니다.  

변경 후에 apply 명령어를 이용해 deployment를 재배포합니다.  
```sh
$ kubectl apply -f deployment.yml
```

그리고 난 후 다음 명령어로 이벤트를 조회해봅니다.  
```sh
$ kubectl describe deployment/my-app
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/describe_deployment_after_updating_image.png)
기존에 생성되었던 replicaset의 pod 수를 0개로 줄인 후에 새로운 replicaset을 이용해 3개의 pod를 다시 띄우는 것을 확인할 수 있습니다.  

replicaset의 상태 확인은 아래 명령어를 통해 가능합니다.  
```sh
$ kubectl get rs -w
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/kubectl_get_rs_-w_after_update_image.png)
deployment 초기 배포시 생성되었던 replicaset의 개수를 0개까지 줄이고 새로운 replicaset을 통해 다시 pod를 3개 띄우는 것을 확인할 수 있습니다.  


deployment의 상태 확인은 아래 명령어를 통해 가능합니다.  
```sh
$ kubectl get deployment -w
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/kubectl_get_deployment_-w_after_updating_image.png)

### Recreate 배포 전략 동작 방법  
Recreate를 배포 전략으로 삼은 후 deployment에 업데이트 사항이 있을 때 pod와 replicaset은 다음 그림과 같이 동작합니다.  
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/recreate_strategy_work_process.png)

위의 그림에서 보면 알 수 있듯이 Recreate 전략 사용 시에는 기존 pod가 모두 제거되고 난 후에 새로운 pod가 생성됩니다.  
따라서 Recreate 방식은 다운 타임이 발생하므로 운영에는 적합하지 않고 적은 리소스로 서비스를 확인해보는 개발 환경에 적합합니다.  


## RollingUpdate 배포
RollingUpdate는 운영 환경에서 사용 가능한 배포 전략입니다.  
RollingUpdate 방식을 통해 배포를 하면 기존 pod를 다 삭제한 후 신규 pod를 띄우지 않기 때문에 기존 pod와 신규 pod가 동시에 존재하는 시점이 존재합니다.   

RollingUpdate 방식으로 배포할 때 속도 제어 옵션을 사용할 수 있는데요, 옵션으로는 maxUnavailable과 maxSurge가 있습니다.  
- maxUnavailable: 기존의 Pod를 새로운 Pod로 전환하는 과정에서 Pod 제거와 생성을 반복할 때 최소로 유지해야하는 Pod의 개수
  - `desired replicas - maxUnavailable`
- maxSurge: 기존 Pod를 새로운 Pod로 전환하는 과정에서 Pod 제거와 생성을 반복할 때 동시에 존재할 수 있는 최대 Pod의 개수
  - `desired replicas + maxSurge`


RollingUpdate는 배포가 점진적으로 일어나며, 아래 그림과 같은 과정을 거쳐 신규 Pod로 모든 Pod들이 대체됩니다.   
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_process.png) . 
배포 과정 중 아래와 같이 기존 pod가 1개 더 종료되고있을 때 `replicas = 3, maxSurge=0`임을 감안하여, 한번에 정상 동작중인 pod가 최대 3개까지 가능하므로 2개의 신규 pod를 동시에 생성할 수 있습니다.  
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollinpudate_process2.png)

추가적으로 `maxSurge=1`인 상황에 대해서 알아보도록 하겠습니다.   
아래와 같은 상황에서 동시에 존재할 수 있는 pod 개수의 최소값은 2이고, 최대값은 4가 됩니다.   
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingUpdate_maxSurge=1.png)  
배포 과정 중 아래와 같이 기존 pod가 1개 더 종료되고있을 때 `replicas = 3, maxSurge=1`임을 감안하여, 한번에 정상 동작중인 pod가 최대 4개까지 가능하므로 3개의 신규 pod를 동시에 생성할 수 있습니다.  
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingUpdate_maxSurge=1_v2.png)  


### RollingUpdate 배포 전략 동작 방법  
RollingUpdate를 배포 전략으로 삼은 후 deployment에 업데이트 사항이 있을 때 pod와 replicaset은 다음 그림과 같이 동작합니다.  
아래 그림은 `relicas=3, maxUnavailable=1, maxSurge=1`인 경우 재배포가 있을 때 동작 흐름입니다.
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_strategy_work_process.png)  
`maxSurge=1`이기 때문에 재배포 시작 즉시 신규 pod를 하나 생성할 수 있습니다.  
그 후부터는 기존 pod가 하나 제거될 때마다 새로운 pod 배포가 가능합니다.  
따라서 Ready 상태로 유지해야 하는 최소 ~ 최대 pod replicas 구간은 <2개 ~ 4개> 임을 알 수 있습니다.


### Deployment 생성  
Deployment를 배포하기 위해 아래 스펙대로 `deployment.yml` 파일을 작성합니다.  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
        app: my-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
        project: deployment_sample
        env: local
    spec:
      containers:
      - name: my-app
        image: yoonjeong/my-app:1.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```
strategy 항목을 보면 type을 RollingUpdate로 정의했으며, `maxUnavailable`을 2로, `maxSurge`를 1로 설정했음을 확인할 수 있습니다.  

deployment를 아래 명령어를 통해 생성합니다.  
```sh
$ kubectl apply -f deployment.yml
```

deployment의 배포 진행 / 완료 상태를 확인하기 위해 아래 명령어를 사용합니다.  
```sh
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_rollout_status_deployment.png)

app=my-app 레이블을 갖는 모든 리소스를 조회하기 위해서는 아래 명령어를 사용할 수 있습니다.  
```sh
$ kubectl get all -l app=my-app -o wide --show-labels
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_get_all_-l_app=my-app.png)


### 컨테이너 이미지와 레이블 변경  
위에서 생성한 deployment의 컨테이너 이미지와 레이블을 변경해서 재배포해보도록 하겠습니다.  
앞서 작성한 `deployment.yml` 파일을 아래와 같이 변경해줍니다.  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
        app: my-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
        project: deployment_sample
        env: local
        version: v2
    spec:
      containers:
      - name: my-app
        image: yoonjeong/my-app:2.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

변경 후에 apply 명령어를 이용해 deployment를 재배포합니다.  
```sh
$ kubectl apply -f deployment.yml
```

그리고 난 후 다음 명령어로 이벤트를 조회해봅니다.  
```sh
$ kubectl describe deployment/my-app
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_describe_deployment_after_updating_image.png)   
기존에 생성되었던 replicaset의 pod 수를 0개로 줄인 후에 새로운 replicaset을 이용해 5개의 pod를 다시 띄우는 것을 확인할 수 있습니다.  

replicaset의 상태 확인은 아래 명령어를 통해 가능합니다.  
```sh
$ kubectl get rs -w
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_kubectl_get_rs_-w_after_update_image.png)   
deployment 초기 배포시 생성되었던 replicaset의 개수를 0개까지 줄이고 새로운 replicaset을 통해 다시 pod를 5개 띄우는 것을 확인할 수 있습니다.  


deployment의 상태 확인은 아래 명령어를 통해 가능합니다.  
```sh
$ kubectl get deployment -w
```
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_kubectl_get_deployment_-w_after_updating_image.png)


### RollingUpdate 배포 전략 동작 방법  
위의 실습에서 RollingUpdate를 배포 전략으로 삼아 deployment를 업데이트해보았습니다.  
이 때 각 단계별로 replicaset과 pod는 다음과 같이 동작합니다.  
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/rollingupdate_strategy_work_process_replicas_5.png)  
`maxUnavailabe=2`이므로 재배포 직후 기존 pod를 2개까지 즉시 삭제할 수 있으며, `maxSurge=1`이기 때문에 최대 6개까지 동시에 pod를 띄울 수 있어 신규 pod 3개를 즉시 생성 가능합니다.   
모든 pod가 신규 pod로 배포될 때까지 주어진 `maxUnavailable`과 `maxSurge`값에서 벗어나지 않는 선에서 신규 pod로 변경되고 있음을 확인할 수 있습니다.  
`rollingupdate`는 이와 같이 서비스 다운타임이 발생하지 않으며, `maxUnavailable`과 `maxSurge`값을 통해 서버 자원 사용량을 조절 가능하다는 점에서 운영 환경에서 쓰이기 적합합니다.  
