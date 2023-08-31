# 쿠버네티스 Deployment Pod Rollback 

## Deployment rollback
쿠버네티스를 이용해 deployment 오브젝트를 배포하고, 버전을 업그레이드 했을 때 예기치 못한 문제로 인해 서비스에 오류가 발생할 수 있습니다.  
이럴 때는 이전 버전으로의 롤백이 필요한데요, 이 때 `Revision`을 이용해 Deployment의 롤백을 시도할 수 있습니다.  
또한 롤백에 대한 사유를 `Annotation`을 통해 남겨둘 수도 있습니다.  

## Deployment Rollback 실습
Deployment를 rollback 해보기 위해 다음 과정을 거쳐 실습을 진행해보도록 하겠습니다.  
1. `1.0` 버전의 이미지를 이용해 deployment를 배포  
2. 이미지 버전을 `2.0`으로 변경해 재배포
3. 배포기록 Revision 조회
4. Revision을 이용해 이미지 1.0으로 롤백
5. 롤백 사유 남기기   

### deployment 배포
아래 내용으로 `deployment.yml` 파일을 작성하여 deployment 오브젝트를 배포하도록 하겠습니다.   
`metadata.annotations.kubernetes.io/change-cause` 항목을 추가하여 배포에 대한 설명을 추가하였습니다.  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
  annotations:
    kubernetes.io/change-cause: "initial image 1.0"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
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

위처럼 작성된 yml 파일을 가지고 아래 명령어를 통해 deployment 오브젝트를 배포합니다.  
```sh
$ kubectl apply -f deployment.yml
```
배포 후에는 아래 명령어를 통해 rollout 상태를 확인합니다.  
```sh
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/rollout_status.png)

### 이미지 변경  
my-app deployment의 pod의 컨테이너에서 사용하고 있는 이미지를 `2.0` 버전으로 업그레이드 해보겠습니다.  
```sh
$ kubectl set image deployment/my-app my-app=yoonjeong/my-app:2.0
```

이미지 업데이트 후 deployment를 조회해보면 잘 적용되었음을 확인할 수 있습니다.  
```sh
$ kubectl get deployment my-app -o wide
```
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/get_deployment_-o_wide.png)

이미지를 변경했으니 `annotate` 명령어를 이용해 변경 사유를 남겨보도록 하겠습니다.  
```sh
$ kubectl annotate deployment/my-app kubernetes.io/change-cause="image updated to 2.0"
```

### Revision 목록 조회  
Revision은 deployment가 지금까지 배포된 pod template들을 저장해두는 기능입니다.  
Revision 기록을 확인하는 명령어는 다음과 같습니다.  
```sh
$ kubectl rollout history deployment/my-app
```
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/rollout_history_deployment.png)

위의 결과에서 `CHANGE-CAUSE`는 Deployment 오브젝트 내용을 변경하여 배포하게 된 사유를 기록한 것입니다.  
`Revision`은 1부터 순차적으로 숫자가 증가하며, 숫자가 커질수록 최근 배포된 기록입니다.  

만약 배포 히스토리 중 특정 revision의 상세 정보를 보고 싶다면 `--revision=N` 옵션을 사용해서 다음 명령어를 통해 확인할 수 있습니다.  
```sh
$ kubectl rollout history deployment/my-app --revision=2
```
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/rollout_history_with_revision_option.png)

### Rollback 진행  
이전 혹은 특정 Revision으로 Pod를 재배포하고 싶다면 아래 명령어를 사용할 수 있습니다.  
```sh
# 이전 버전으로 롤백
$ kubectl rollout undo deployment/my-app

# 특정 revision 버전으로 롤백
$ kubectl rollout undo deployment/my-app --to-revision=1
```
`revision=1`로 롤백을 진행하고 rollout history를 다시 확인해본 결과 이전 버전으로 롤백이 잘 이루어졌고, annotation도 잘 적용되었음을 확인할 수 있습니다.  
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/rollout_undo_to_revision.png)

만약 롤백 사유를 추가적으로 따로 남기고 싶다면 다음과 같은 명령어를 사용할 수 있습니다.  
```sh
$ kubectl annotate deployment/my-app kubernetes.io/change-cause="image reverted to 1.0 for a few bugs"
```
위의 명령어를 적용한 후에 다시 Rollout history를 확인해보면 변경된 annotation이 잘 적용되었음을 확인할 수 있습니다.   
![](/assets/img/2023/08/2023-08-27-kubernetes_deployment_rollback/kubectl_annotate.png)
