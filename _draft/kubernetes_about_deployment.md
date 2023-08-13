# 쿠버네티스 Deployment

## ReplicaSet과의 비교
이전 포스트들에서 ReplicaSet을 통해 Pod를 배포하는 방식에 대해 알아보았습니다.  
ReplicaSet을 이용해서 Pod를 배포하면 다음과 같은 이점이 있습니다.  
1. ReplicaSet의 Pod 복제 기능을 통해 여러개의 Pod를 한 번에 실행할 수 있습니다.  
2. 선언한 replicas 수만큼 Pod의 실행을 보장합니다.  
3. ReplicaSet이 Pod 상태를 항상 감시합니다.  
4. Pod 실행 중에도 replicas 조정이 자유롭습니다.  

ReplicaSet을 통해 pod를 배포하면 pod에 문제가 있어 이전 버전으로 롤백을 하려고 하면 아래 과정을 거쳐야합니다.  
1. 새로운 ReplicaSet을 만들어 Pod 재배포하거나 Pod Template을 변경 후 적용
2. 사용하지 않는 ReplicaSet과 Pod를 제거
3. 롤백 or 새 버전 배포 시마다 위의 과정을 반복

따라서 ReplicaSet을 통해 신규 버전을 배포하거나 이전 버전으로의 롤백을 진행하려고 한다면 관리자가 직접 새로운 ReplicaSet의 배포 및 이전 ReplicaSet과 Pod를 삭제해주는 작업을 진행해야합니다.  
그렇다면 이런 과정을 쿠버네티스에 위임하는 방법은 없을까요?  
쿠버네티스에서는 더 간편한 배포를 위해 Deployment라는 오브젝트를 제공하고 있습니다.  

Pod 배포 시에 필수적으로 정의해야 하는 값들은 아래와 같습니다.  
- selector: 어떤 Pod 집합을 대상으로 Replication 해야하는 지
- replicas: Pod를 몇 개나 생성할 지
- Pod Template Image: Pod에서 어떤 컨테이너를 실행할지

신규 배포를 진행하거나 롤백 할 때 바뀌는 부분은 Pod Template Image로, 기존에 ReplicaSet을 이용할 때는 변경된 이미지가 반영된 Pod를 띄우기 위해 직접 Pod 신규 배포와 이전 Pod 삭제롤 관리했습니다.   
이에 대해 만약 쿠버네티스가 알아서 ReplicaSet을 생성하고 이전 Pod를 제거해주는 기능을 제공받는다면 더 편한 배포가 가능할 것이고, 이러한 기능을 제공해 주는 것이 바로 Deployment 입니다.  


## Deployment 개념과 특징
Deployment는 Pod 배포 자동화를 위한 오브젝트로 `ReplicaSet 오브젝트`에 `배포 전략`이 추가된 오브젝트라고 볼 수 있습니다.  

Deployment가 수행해주는 주요 역할은 아래와 같습니다.  
- 새로운 Pod를 롤아웃/롤백할 때 ReplicaSet 생성을 대신해줍니다.  
- 다양한 배포 전략을 제공하고 이전 파드에서 새로운 파드로의 전환 속도를 제어할 수 있습니다.  

Deployment가 제공해 주는 기능은 ReplicaSet을 통해 pod를 배포했을 때 겪는 어려운 부분들을 보완해주므로 Pod를 배포할 떄 ReplicaSet이 아닌 Deployment를 사용해 배포하면 Pod를 더 손쉽게 관리할 수 있습니다.  

Deployment 오브젝트를 생성할 때도 ReplicaSet과 유사한 항목들을 정의해야하는데요, 주요 항목을 꼽아보면 아래와 같습니다.  
- Replicas
- Pod Selector
- Pod Template

### Deployment 생성 명세
Deployment 오브젝트를 배포할 떄 작성해야하는 yml 파일의 기본 구조에 대해 알아보도록 하겠습니다.  
yml 파일의 주요 항목을 살펴보면 다음과 같은 형태를 갖습니다.  
```yml
apiVersion: apps/v1   # kubernetes API 버전
kind: Deployment      # 오브젝트 타입
metadata:             # 오브젝트 식별을 위한 정보
  name: my-app        # 오브젝트 이름
  labels:
        app: my-app
spec:                 # 배포하려는 Pod 정보
  selector:           # ReplicaSet을 통해 관리할 Pod를 선택하기 위한 Label query
    matchLabels:
      app: my-app
  replicas: 3         # pod 복제본 개수
  template:           # pod template (pod 실행 정보)
    metadata:
      labels:
        app: my-app   # selector에 정의한 label을 포함해야 deployment에 의해 관리됨
    spec:
      containers:
      - name: my-app
        image: my-app:1.0
```

이처럼 배포된 Deployment 오브젝트에 대해 Pod template을 변경하여 재배포를 하게 되면 어떻게 될까요?
Pod 개수를 조정 및 삭제하여 신규 이미지의 Pod를 배포했던 ReplicaSet과 달리 Deployment는 새로운 ReplicaSet을 생성해 새로운 버전의 이미지를 통해 Pod를 띄우고, 기존에 존재했던 ReplicaSet의 scale을 0으로 조정해 원래 사용했던 pod의 개수를 0개로 줄입니다.  
ReplicaSet을 이용해 배포할 때는 사용자가 직접 진행했던 작업을 쿠버네티스에서 책임지고 진행해주는 것입니다.  

## Deployment 배포(롤아웃) 전략
Deployment를 통해 신규 버전을 배포할 때 사용되는 전략에는 여러 가지가 있습니다.  
몇 가지를 하나씩 알아보도록 하겠습니다.  

### Recreate 배포 전략
Recreate 전략으로 신규 버전을 배포하면 Deployment는 이전 Pod를 모두 종료하고 새로운 Pod를 replicas만큼 생성합니다.  
이 방식을 통해 배포를 진행하면 이전 버전의 Pod를 모두 삭제한 후에 새로운 Pod들을 생성하므로 필연적으로 Pod가 하나도 존재하지 않는 시점이 생깁니다.  
이는 서비스 다운타임이 발생한다는 의미가 되기 때문에, 개발 단계에서는 시도해볼만한 방식이지만 운영 시에는 사용하기 적합하지 않습니다.  

**replicas=3일 때 `Recreate` 배포 전략으로 배포시 pod의 상태**   

|단계|pod1|pod2|pod3|상태|
|---|---|---|---|---|
|롤아웃 시작|v1|v1|v1|이전 버전의 pod 3개 존재|
|롤아웃 진행중|-|-|v1|이전 버전의 pod 삭제중|
|롤아웃 진행중|-|-|-|이전 버전의 pod 모두 삭제됨, 신규 버전의 Pod 배포 전 <-- pod가 하나도 존재 X|
|롤아웃 완료|v2|v2|v2|신규 버전의 pod 배포 완료|  


### RollingUpdate 배포 전략
RollingUpdate 전략으로 신규 버전을 배포하면 Deployment는 새로운 Pod 생성과 이전 Pod 종료를 동시에 진행하며 배포를 수행합니다.  
이 방식을 통해 배포하면 이전 버전의 Pod가 삭제된 개수만큼 신규 pod가 생성됩니다.  
따라서 서비스 다운타임이 따로 발생하지는 않지만, 이전 버전의 Pod와 새로운 버전의 Pod가 함께 존재하므로 이전 버전의 Pod쪽으로 요청이 들어오면 예전 버전의 응답을 주게 됩니다.  

**replicas=3일 때 `RollingUpdate` 배포 전략으로 배포시 pod의 상태**   
|단계|pod1|pod2|pod3|상태|
|---|---|---|---|---|
|롤아웃 시작|v1|v1|v1|이전 버전의 pod 3개 존재|
|롤아웃 진행중|v2|v1|v1|이전 버전의 pod 삭제 & 신규 버전 pod 생성|
|롤아웃 진행중|v2|v2|v1|이전 버전의 pod 삭제 & 신규 버전 pod 생성|
|롤아웃 완료|v2|v2|v2|신규 버전의 pod 배포 완료|  


### Recreate와 RollingUpdate 비교
- Recreate (재생성)
  - 새로운 버전을 배포하기 전에 이전 버전이 즉시 종료됨
  - 컨테이너가 정상적으로 시작되기 전까지 서비스하지 못함
  - replicas 수만큼 컴퓨팅 리소스 필요
  - 개발 단계에서 유용
- RollingUpdate (롤링 업데이트)
  - 새로운 버전을 배포하면서 이전 버전을 종료
  - 서비스 다운 타임 최소화
  - 동시에 실행되는 Pod의 개수가 replicas를 넘게 되므로 컴퓨팅 리소스 더 많이 필요


### RollingUpdate 방식의 배포 속도 조절
RollingUpdate 방식은 점진적으로 배포하는 개수에 따라서 배포 속도를 조절할 수 있습니다.  
속도에 관련된 옵션 2가지를 살펴보도록 하겠습니다.  

**maxUnavailable**   
maxUnavailable은 최대로 이용하지 못하는 Pod의 개수를 지정하는 것입니다.  
즉 Deployment를 다시 배포했을 때 최대 몇 개까지를 즉시 종료할 수 있을 지를 정의하는 것입니다.  

maxUnavailble 옵션을 이용하면 롤링 업데이트를 수행하는 동안 유지하고자 하는 최소 Pod 비율/개수를 지정할 수 있습니다.  
최소 pod 유지 비율은 100 - maxUnavailable 값으로 예를 들어 replicas가 10일 때 maxUnavailable을 30%로 지정하면 최소 Pod 유지 비율은 70%가 되는 것입니다.  
위의 상황은 아래와 서로 다른 표현법으로 아래와 같이 표현할 수 있으며 각각이 의미하는 바는 동일합니다.  
- 이전 버전의 Pod를 replicas 수의 최대 30%까지 즉시 Scale Down 할 수 있다
- replicas를 10으로 선언했을 때 이전 버전의 pod를 3개까지 즉시 종료할 수 있다
- 새로운 버전의 Pod 생성과 이전 버전의 Pod 종료를 진행하면서 replicas 수의 70% 이상의 Pod를 항상 Running 상태로 유지해야 한다  

**maxSurge**
새로운 버전의 Pod를 한 번에 최대 몇 개까지 생성할 수 있는 지에 대한 옵션입니다.  
이 옵션을 이용하면 롤링 업데이트를 수행하는 동안 허용할 수 있는 최대 Pod의 비율/개수를 지정할 수 있습니다.  

최대 Pod 허용 비율 = maxSurge 값이므로 replicas가 10개일 떄 maxSurge=30%라면 아래와 같은 동작들이 가능합니다.  
- 새로운 버전의 Pod를 replicas 수의 최대 30%까지 즉시 Scale Up 할 수 있다
- 새로운 버전의 Pod를 3개까지 즉시 생성할 수 있다  
- 새로운 버전의 Pod 생성과 이전 버전의 Pod 종료를 진행하면서 총 Pod 수가 replicas 수의 130%를 넘지 않도록 유지해야한다  


## Deployment 롤백 전략
Deployment는 롤아웃 히스토리를 Revision # 으로 관리합니다.  

Deployment Revision이 Revision 1, Revision 2, Revision 3 이 있다면 Revision 1이 가장 오래된 롤아웃 히스토리이고, Revision 3이 가장 최신 Revision 히스토리입니다.  
Deployment를 이용해 롤백을 하는 경우에는 현재 상태에서 특정 Revision N으로 바로 이동이 가능합니다.  
또한 특정 Revision에 대해 조회를 하면 해당 Revision 배포 시 사용했던 Pod Template의 상세 정보를 조회할 수 있습니다.  
```yml
# Pod Template 조회 결과
Pod Template:
  Labels: app=my-app
          version=v1
          pod-template-hash=ca76d9fd83    # hash값이 같다면 같은 pod template으로 배포한 것임  
  Annotations: kubernetes.io/change-cause: v1 배포
  Containers:
    my-app:
      Image: nginx:1.16.1
      Port: 80/TCP
```

Revision을 이용해 롤백할 때 사용하는 명령어는 아래와 같습니다.  
```sh
# Revision 1 버전으로 롤백 수행
$ kubectl rollout undo deployment <deployment-name> --to-revision=1
```


## Deployment 이용 예시  
Deployment의 동작을 확인해보기 위해 다음 4가지 항목에 대한 수행 결과를 확인해보도록 하겠습니다.   
1. Deployment를 통한 Relicaset 생성과 Pod 복제 결과 확인  
2. Deployemnt Pod replicas 변경 후 결과 확인  
3. Deployment Pod Template 이미지 변경 후 결과 확인
4. Deploymetn Pod Template 레이블 변경 후 결과 확인  

### Deployment 생성 및 replicas 변경  
Deployment 오브젝트를 생성하고 `kubectl sacel -- replicas` 명령어를 통해 Pod replicas를 변경해보도록 하겠습니다.  
 
사용할 deployment 오브젝트를 다음과 같이 `deployment.yaml` 파일로 정의합니다.  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
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

그리고 다음 명령어를 통해 deployment 오브젝트를 생성합니다.  
```sh
$ kubectl apply -f deployment.yaml
```

생성된 오브젝트를 다음 명령어를 통해 조회할 수 있습니다.  
```sh
$ kubectl get deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_get_deployment.png)

상세 정보 조회는 `describe` 명령어를 통해 가능합니다.  
```sh
$ kubectl describe deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_describe_deployment.png)

<br/>

> Deployment를 통해 생성되는 ReplicaSet과 Pod 이름의 특성은 아래와 같습니다.     
> **ReplicaSet 이름:** `<deployment-name>-<pod-template-hash>`   
> **Pod 이름:** `<deployment-name>-<pod-template-hash>-<임의의 문자열>`   
![](2023-08-06-15-23-46.png)
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_get_pod_get_replicaset.png)
>> **pod-template-hash:** Pod Template을 해싱한 값  
>  
> ReplicaSet 이름 생성 방식을 통해 Pod Template이 변경되면 pod-template-hash도 변경되므로 Pod Template이 변경되면 새로운 ReplicaSet이 생성됨을 알 수 있습니다.   
> 또한 pod-template-hash가 ReplicaSet과 Pod의 Labels에 추가된 것도 확인할 수 있습니다.  

Deployment가 생성될 때 Replicaset의 상태 변화를 확인해보면 다음과 같습니다.   
```sh
$ kubectl get rs -w
```
```
NAME                DESIRED   CURRENT   READY   AGE
my-app-65cb545f45   2         0         0       0s
my-app-65cb545f45   2         0         0       0s
my-app-65cb545f45   2         2         0       0s
my-app-65cb545f45   2         2         1       3s
my-app-65cb545f45   2         2         2       3s
```  

Deploy가 생성될 때 Pod의 상태 변화를 확인해보면 다음과 같습니다.   
```sh
$ kubectl get deployment -w
```
```
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   0/2     0            0           0s
my-app   0/2     0            0           0s
my-app   0/2     0            0           0s
my-app   0/2     2            0           0s
my-app   1/2     2            1           3s
my-app   2/2     2            2           3s
```

또한 Deployment의 배포 진행 및 완료 상태를 확인할 때 사용할 수 있는 명령어로는 아래 명령어가 있습니다.
```
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/rollout_status_deployment.png)
명령어의 결과로 위의 메세지를 보게된다면, 모든 pod가 잘 생성되었다고 볼 수 있습니다.  

**Deployment의 Pod Replicas 변경 (spec.replicas)**  
pod replicas를 변경해 deployment에서 관리하는 pod의 개수를 변경해보도록 하겠습니다.  
```sh
$ kubectl scale deployment/my-app --replicas=5
```  

replicas 개수를 변경한 후에 Deployment의 이벤트를 확인해보면 아래와 같습니다.  
```sh
$ kubectl describe deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/scale_replicas_and_describe_event.png)


replicaset의 이벤트를 다음 명령어를 통해 조회해보면 아래와 같습니다.  
```sh
$ kubectl describe rs/my-app-<replicaset hash 값>
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_describe_rs.png)


**Deployment를 통해 생성한 Pod로 요청 전달 & 응답 확인**  
pod에 연결하기 위해 pod의 8080번 포트를 로컬의 8080번 포트로 포트포워딩 해보도록 하겠습니다.  
```sh
$ kubectl port-forward deployment/my-app 8080:8080
```

작업 후에 `localhost:8080` 주소로 접근하면 다음과 같은 결과를 확인할 수 있습니다.  
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/port-forward_and_check_result.png)


**Deployment의 replicas 변경 결론**  
위의 명령어들을 통해 상태를 조회해본 결과 deployment의 replicas를 변경한다고 해서 새로운 ReplicaSet이 생성되지는 않는 것을 알 수 있습니다.  
그러나 이미 생성한 ReplicaSet이 새로운 Pod를 필요한 개수만큼 추가적으로 생성합니다.  


**리소스 삭제**  
위에서 생성한 deployment 오브젝트를 아래 명령어를 통해 삭제합니다.  
```sh
# deployment를 삭제하는 방식
$ kubectl delete deployment/my-app

# 특정 label을 갖는 모든 리소스를 삭제하는 명령어: kubectl delete all -l <label-key>=<label-value>
$ kubectl delete all -l app=my-app
```


## Deployment Pod Template 이미지 변경  
다음 실습으로는 `my-app:1.0` 이미지로 생성한 deployment의 컨테이너들을 `my-app:2.0` 이미지로 변경하고 싶을 때 어떻게 하면 되는 지에 대해서 알아보도록 하겠습니다.  

이번 실습에서 확인해볼 사항은 아래와 같습니다.  
1. Deployment 생성
2. Deployment의 Pod Template 이미지 변경
3. Deployment, ReplicaSet, Pod 변화 확인  

deployment의 이미지를 변경하면 template의 hash값이 변함에 따라 pod들은 아래 그림처럼 새로 생성됩니다.  
새로운 ReplicaSet을 생성하고 기존에 존재하던 Pod를 제거하는 과정을 거쳐 모든 Pod를 새로운 Replicaset에 의해 생성된 Pod로 변경하게 됩니다.  
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/deployment_template_image_update.png)

### Deployment 생성
새로 생성할 Deployment 오브젝트는 앞에서 생성한 Deployment와 거의 유사하나 replicas만 개수를 3개로 늘려 배포하도록 하겠습니다.  
deployment 생성 정보를 작성한 `deployment2.yml` 파일을 아래와 같이 작성합니다.  
```yaml
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
  template:
    metadata:
      labels:
        app: my-app
        project: deployment_sample2
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

아래 명령어를 통해 deployment를 생성합니다.  
```sh
$ kubectl apply -f deployment2.yaml
```

Deployment의 ReplicaSet 이벤트를 아래 명령어를 통해 확인해볼 수 있습니다.  
```sh
$ kubectl describe deployment/my-app
```

ReplicaSet의 pod 개수 변화를 확인하기 위해서는 다음 명령어룰 사용할 수 있습니다.  
```sh
$ kubectl get rs -w
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_get_rs_-w.png)

Deployment를 통해 생성한 Pod의 상태를 확인하기 위해서는 다음 명령어를 사용할 수 있습니다.  
```sh
$ kubectl get deployment -w
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_get_deployment_-w.png)

또한 deployment의 배포 진행/완료 상태를 확인하고 싶을 때는 다음 명령어를 사용할 수 있습니다.  
```sh
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_rollout_status_deployment.png)


### Deployment의 Pod Template 이미지 변경
Deploymentdml my-app 컨테이너 imagefmf 2.0으로 변경하도록 하겠습니다.   
Pod 이미지를 변경하기 위해 사용해야 하는 명령어는 다음과 같습니다.  
```sh
$ kubectl set image deployment/my-app my-app=yoonjeong/my-app:2.0
```

이미지를 변경한 후 각종 명령어를 통해 pod 및 replicaset의 변경 상태를 추적해보도록 하겠습니다.   
Deployment의 ReplicaSet 이벤트를 확인해보면, 이전 버전의 pod들은 점점 감소하고 새로운 pod들이 총 3개까지 새로 생성되는 것을 확인할 수 있습니다.  
```sh
$ kubectl describe deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_describe_after_set_image.png)

ReplicaSet의 pod 개수 변화를 확인하기 위해서는 다음 명령어룰 사용할 수 있습니다.  
```sh
$ kubectl get rs -w
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/get_rs_-w_after_set_image.png)

Deployment를 통해 생성한 Pod의 상태를 확인하기 위해서는 다음 명령어를 사용할 수 있습니다.  
```sh
$ kubectl get deployment -w
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/kubectl_get_deployment_-w_after_set_image.png)

또한 deployment의 배포 진행/완료 상태를 확인하고 싶을 때는 다음 명령어를 사용할 수 있습니다.  
```sh
$ kubectl rollout status deployment/my-app
```
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/rollout_status_after_set_image.png)

위의 과정을 그림으로 확인해보면 아래 그림과 같이 pod의 변경이 일어납니다.  
![](/assets/img/2023/07/2023-08-06-kubernetes_deployment/pod_changes.png)
