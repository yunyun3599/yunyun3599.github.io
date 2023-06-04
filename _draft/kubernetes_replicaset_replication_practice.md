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

## Pod 종료 시 ReplicaSet 동작 확인 실습  
### ReplicaSet의 내결함성 확인  
ReplicaSet이 관리하는 Pod를 삭제하여 Pod 장애 상황을 만들고, 그에 대해 ReplicaSet이 관리하는 Pod 목록에 어떤 변화가 생기는 지 확인해보도록 하겠습니다.  

먼저 위에서 작성한 `replicaset.yaml` 파일을 이용해 ReplicaSet을 배포해보도록 하겠습니다.  
```sh
$ kubectl apply -f replicaset.yaml 
```

ReplicaSet을 확인해보면 다음과 같이 잘 생성되었음을 알 수 있습니다.  
```sh
$ kubectl get rs blue-replicasete -o wide
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_get_replicaset_blue_replicaset.png)

pod 목록도 조회해보도록 하겠습니다.  
```sh
$ kubectl get pod -o wide
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/get_pod_created_by_replicaset.png)

목록에 조회된 Pod 들 중 하나를 다음 명령어를 통해 삭제해보도록 하겠습니다.  
```sh
$ kubectl delete pod blue-replicaset-4mcv2
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_delete_pod.png)   

pod 삭제 후 다시 pod 목록을 조회해보도록 하겠습니다.  
```sh
$ kubectl get pod 
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_get_pod_after_deleting_pod.png)
pod 개수는 3개로 유지되면서 새로운 이름의 pod이 하나 생성되었음을 확인할 수 있습니다.  

ReplicaSet이 수행한 일을 확인해보기위해 describe 명령어를 이용해 이벤트를 확인해보도록 하겠습니다.  
```sh
$ kubectl describe rs blue-replicaset
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_describe_rs_blue_replicaset_check_auto_creation.png)    
새로운 pod가 생성된 것을 이벤트의 마지막줄을 통해 확인할 수 있습니다.  

이러한 결과를 통해 ReplicaSet을 이용해 Pod의 생성과 복구를 자동화할 수 있음을 알 수 있습니다.  
pod가 삭제되더라도 ReplicaSet을 통해 관리되는 pod들은 replicas에 선언된 개수를 유지하기 위해 자동으로 새루은 pod를 생성하기 때문입니다.  

또한 단순히 pod 하나가 삭제된 것이 아니라 노드 전체가 실패하는 경우가 생기더라도 ReplicaSet을 통해 관리되고 있는 Pod들은 스케쥴러의 도움을 받아 정상 동작중인 새로운 노드에 replicas 개수를 맞춰 다시 배포되게 됩니다.  


### ReplicaSet 삭제 실습  
다음으로는 ReplicaSet이 삭제되었을 때 Pod 목록에 어떤 변화가 생기는 지 확인해보도록 하겠습니다.  

ReplicaSet을 삭제할 때 유사해보이지만 조금 다르게 동작하는 두 가지 명령어를 확인해보도록 하겠습니다.  
1. ```sh
    $ kubectl delete rs blue-replicaset
   ```
   - 위 명령어를 수행하면 해당 replicaset을 통해 관리되고 있던 pod들도 함께 삭제됩니다.  
      ![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_delete_rs_without_any_options.png)
2. ```sh
    $ kubectl delete rs blue-replicaset --cascade=orphan
   ```
   - 위 명령어를 통해 ReplicaSet을 삭제할 경우 ReplicaSet이 삭제되어도 pod들은 삭제되지 않습니다.  
      ![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubect_delete_rs_blue_replicaset_cascade_orphan.png)
   - ```sh
      $ kubectl get pod <pod-name> -o jsonpath="{.metadata.ownerReferences[0].name}"
      ``` 
      위 명령어를 통해 pod의 owner를 확인할 수 있는데, 위의 명령어를 사용하기 전에는 owner로 replicaset의 이름이 조회되는 반면 replicaset을 삭제한 후에는 아무 내용도 출력되지 않는 것을 확인할 수 있습니다.  
        ![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/checking_owner_after_delete_rs_cascade_orphan.png)

그 외에도 pod를 먼저 제거하고 ReplicaSet을 삭제하는 방법도 있습니다.  
ReplicaSet 삭제 없이 pod를 없애려면 `scale` 명령어를 통해 pod의 개수를 0개로 조정하면 됩니다.  
```sh
$ kubectl scale rs/blue-replicaset --replicas 0
```
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_scale_replicas_0.png)

참고로 scale 명령어를 이용해 ReplicaSet이 관리하는 pod의 개수를 조절할 수도 있지만, 기존에 배포를 위해 작성했던 yaml파일의 replicas 항목 개수를 변경해 재배포해도 pod의 개수는 새로운 replicas 항목의 값으로 설정됩니다.  
이 때 ReplicaSet의 Selector 값을 그대로 유지해서 기존에 관리하던 pod들을 그대로 관리할 수 있도록 해야합니다.  

Replicaset의 replicas 개수를 한 개 줄인 후 재배포하면 새로 배포된 ReplicaSet은 기존에 돌아가고 있던 pod를 그대로 받아 관리하면서 개수를 줄이는 동작만하므로 describe를 통해 ReplicaSet이 한 일을 확인해보면 다음과 같이 pod가 한 개 삭제된 이벤트만 조회됩니다.   
![](/assets/img/2023/05/2023-05-30-kubernetes_replicaset_replication_practice/kubectl_describe_rs_after_deploying_again_with_decreased_replicas.png)
