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
$ kubectl apply -f deplyoment.yml
```

그리고 난 후 다음 명령어로 이벤트를 조회해봅니다.  
```sh
$ kubectl describe deployment/my-app
```

replicaset의 상태 확인은 아래 명령어를 통해 가능합니다.  
```sh
$ kubectl get rs -w
```

### Recreate 배포 전략 동작 방법  
Recreate를 배포 전략으로 삼은 후 deployment에 업데이트 사항이 있을 때 pod와 replicaset은 다음 그림과 같이 동작합니다.  
![](/assets/img/2023/08/2023-08-17-kubernetes_deployment_rollout_strategies/recreate_strategy_work_process.png)

