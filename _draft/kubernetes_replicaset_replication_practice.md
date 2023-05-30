# ReplicaSet을 통한 Pod 배포 실습  
## ReplicaSet 오브젝트를 이용해 Pod 생성하기   
### ReplicaSet 선언 및 생성
ReplicaSet을 선언할 때 사용하기 위해 replicaset.yaml파일을 정의하였습니다.  
```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
  name: blue-app
spec:
  selector:
    matchLabels:
      app: blue-app
  replicas: 3
  template:
    metadata:
      labels:
        app: blue-app
    spec:
      containers:
        - name: blue-app
          image: 'yoonjeong/blue-app:1.0'
          ports:
            - containerPort: 8080
```

위 yaml명세를 이용해 replicaset을 생성해보도록 하겠습니다.   
```sh
$ kubectl apply -f replicaset.yaml
```

생성된 replicaset의 목록을 조회하고 싶을 때는 아래 명령어를 사용합니다.  
```sh
$ kubectl get rs blue-replicaset -o wide
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubect_get_rs_-o_wide.png)

생성된 pod들을 조회하고 싶다면 아래 명령어를 사용할 수 있습니다.  
```sh
$ kubectl get pod -o wide
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_get_pod_-o_wide.png)
replicaset을 통해 생성된 pod들은 pod 이름 앞에 prefix로 레플리카셋의 이름이 붙게 됩니다.  

ReplicaSet에 의해 생성된 pod에 대한 생성 과정을 더 자세히 조회하기 위해서는 `describe` 명령어를 사용할 수 있습니다.  
```sh
$ kubectl describe rs blue-replicaset
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_describe_rs_blue_replicaset.png)

kubernetes 클러스터에서 발생한 모든 이벤트에 대해 확인해보고 싶은 경우에는 kubectl get events 라는 명령어를 사용할 수 있습니다.   
```sh
$ kubectl get events --sort-by=metadata.creationTimestamp
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_get_events.png)

### 생성된 Pod으로 요청 / 응답 확인
생성한 pod로 요청을 보내기 위해 pod의 포트를 포트포워딩하도록 하겠습니다.  
```sh
$ kubectl port-forward rs/blue-replicaset 8080:8080
```
위 명령어를 통해 pod내 컨테이너의 8080 포트가 로컬 호스트의 8080 포트로 포워딩이 되면 앞서 생성한 blue-replicaset이라는 레플리카셋에 의해 생성된 파드로 트래픽이 전달됩니다.  
이 때 주의해야할 점은 첫번째 생성된 파드로만 요청이 전달되며 로드밸런싱이 일어나지 않아 트래픽이 분산되지 않는다는 점입니다.  
포트포워딩 후 브라우저에서 제공되는 엔드포인트를를 이용해 컨테이너에 접근하면 다음과 같은 결과를 얻을 수 있습니다.  
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_port_forward.png)


### 기존에 생성한 Pod를 ReplicaSet으로 관리하려는 경우
만약 미리 생성되어있던 Pod들을 Replicaset을 이용해 관리하고자하는 추가 요구사항이 발생한 경우에는 어떻게 해야할까요?  
이런 경우에는 Selector를 이용해 손쉽게 pod들을 replicaset으로 관리하도록 전환할 수 있습니다.  

먼저 replicaset을 사용하지 않고 단순히 pod를 배포해보기 위해 `blue-app.yaml` 파일을 작성해보도록 하겠습니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-app
  labels:
    app: blue-app
spec:
  containers:
  - name: blue-app
    image: yoonjeong/blue-app:1.0
    ports:
      - containerPort: 8080
    env:
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
```

아래 명령어로 pod를 배포합니다.  
```sh
$ kubectl apply -f blue-app.yaml
```
배포 후 목록을 확인해보면 pod가 하나 생성된 것을 볼 수 있습니다.  
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_get_pod_-o_wide_blue-app.png)

그 후에는 `blue-app` 배포시 사용했던 selector와 동일한 selector를 갖도록 설정하여 `blue-replicaset`이라는 ReplicaSet을 생성합니다.  
```sh
$ kubectl apply -f replicaset.yaml
```

ReplicaSet이 잘 생성되었는지 조회해보도록 하겠습니다.
```sh
$ kubectl get rs blue-replicaset -o wide
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubect_get_replicaset_-o_wide.png)

pod 목록도 조회해보도록 하겠습니다.  
```sh
$ kubectl get pod
```
pod 목록을 조회해본 결과 replicaset에 의해 생성된 pod가 2개 생겼음을 볼 수 있습니다.   
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/get_pod_after_set_replicaset.png)

추가적으로 `kubectl describe` 명령어를 사용해 ReplicaSet이 몇 개의 pod를 생성했는지 확인해보도록 하겠습니다.  
```sh
$ kubectl describe rs blue-replicaset
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_describe_rs.png)
기존에 있던 1개의 pod를 제외하고 나머지 2개의 pod를 생성했음을 확인할 수 있습니다.  

> ReplicaSet은 자신이 관리하는 Pod의 수를 replicas 값을 넘지 않게 관리합니다.  
> 따라서 이미 생성되어있던 Pod 레이블이 ReplicaSet의 Pod Selector와 같다면 관리 범주에 들어오므로 Pod Selector 설계 시에 주의해야합니다.   

마지막으로 `delete` 명령어를 통해 지금까지 생성했던 ReplicaSet을 삭제하도록 하겠습니다.   
```sh
$ kubectl delete rs/blue-replicaset
```
