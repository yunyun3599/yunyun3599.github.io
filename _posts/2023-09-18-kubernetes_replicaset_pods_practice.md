---
title:  "Kubernetes ReplicaSet - Pod 확인 실습"
excerpt: "kubernetes의 ReplicaSet을 활용해 pod를 배포했을 때 Pod의 상태를 확인해보는 실습을 진행합니다."

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-09-18
last_modified_at: 2023-09-18
---

# ReplicaSet with Pods
ReplicaSet을 통해 배포된 Pod들이 특정 상황에 어떤 식으로 설정 내용이 적용되어 배포되는 지에 대해 알아보도록 하겠습니다.  

# ReplicaSet - Pod Template 변경
ReplicaSet을 통해 배포된 Pod들은 Pod Template이 변경돼도 영향을 받지 않습니다.  
변경된 내용을 반영하여 새로 pod를 생성하거나 제거하는 경우는 오직 ReplicaSet에 선언한 replicas 값이 변경되었을 경우입니다.  

이러한 상황을 확인해보기 위해 실습을 진행해보도록 하겠습니다.  
실습을 위한 각 단계는 다음과 같습니다.  
1. ReplicaSet 생성
2. ReplicaSet의 Pod Template 레이블 업데이트
3. 실행중인 Pod에는 변화가 없음을 확인
4. Pod 하나 삭제
5. 변경한 Pod Template으로 새로운 Pod가 생성되었는지 확인  

먼저 `replicas`를 2로 설정하여 아래처럼 작성된 my-app-replicaset.yaml 파일을 이용해 `ReplicaSet`을 배포해보도록 하겠습니다.  
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-replicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: yoonjeong/my-app:1.0
          ports:
            - containerPort: 8080
```
위 파일을 통해 replicaset을 배포해보도록 하겠습니다.  
```sh
$ kubectl apply -f my-app-replicaset.yaml
```
배포가 완료된 후 pod 목록을 조회해보면 아래와 같은 결과를 얻을 수 있습니다.  
```sh
$ kubectl get pod --show-labels
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_get_pod_--show-labels.png)

pod이 잘 생성된 것을 확인했으니, 이제 Pod Template에 label을 하나 더 추가해주도록 하겠습니다.  
추가하기 위해서는 다음 내용을 `template` 항목에 추가해주시면 됩니다.  
```yaml
template:
  metadata:
    labels:
      app: my-app
      env: production
```
그 후 replicaset을 다시 배포해보도록 하겠습니다.  
```sh
$ kubectl apply -f my-app-replicaset.yaml
```
그 후 pod 목록을 조회해보면 label이 추가되진 않고 기존 pod 설정이 그대로 있는 것을 확인할 수 있습니다.  
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_get_pod_--show-labels_after_apply_changed_template.png)

앞서 언급했던 것처럼 pod template이 바뀌어도 replicas 개수에 변화가 생기거나, 기존 pod가 삭제되는 등의 이유로 pod가 재생성되어야하는 경우가 아니라면 변경사항이 반영되지 않았음을 확인할 수 있습니다.  

그러므로 pod를 하나 삭제 후 새로 생성되는 pod에는 변경사항이 잘 반영되나 확인해보도록 하겠습니다.  
```sh
$ kubectl delete pod [pod 이름]
```
기존에 설정해둔 `replicas` 개수를 맞춰야하기 때문에 pod 삭제 직후에 새로운 pod가 생성되게 되는데요, pod 목록을 조회해보면 새로 생긴 pod에는 template의 변경사항이 잘 반영되었음을 알 수 있습니다.  
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/delete_pod_and_check_whether_template_change_is_applied.png)


# ReplicaSet - Pod 개수 조정
다음으로는 ReplicaSet의 replicas의 속성을 변경했을 때 실제로 배포된 pod의 개수가 잘 바뀌는 지에 대해 확인해보도록 하겠습니다.  
먼저 앞서 배포한 replicaset을 삭제하도록 하겠습니다.  
```sh
$ kubectl delete rs/my-app-replicaset
```
그 후에는 위에서 작성한 `my-app-replicaset.yaml` 파일에서 `replicas` 속성 값을 1로 변경 후 재배포해보도록 하겠습니다.  
```sh
$ kubectl apply -f my-app-replicaset.yaml 
```

생성된 pod 및 replicaset의 정보를 확인해보겠습니다.  
```sh
$ kubectl get pod --show-labels
$ kubectl get rs my-app-replicaset -o wide
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/get_pod_and_rs_after_apply_replicas_1.png)

잘 생성되었음을 확인했다면 `scale` 명령어를 이용해 `replicas`의 개수를 3개로 증가시켜보도록 하겠습니다.  
```sh
$ kubectl scale rs my-app-replicaset --replicas=3
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_scale_rs_replicas_3.png)
pod의 개수가 3개로 잘 증가했음을 알 수 있습니다.  

다음으로는 다시 replicas의 개수를 1개로 줄이고, replicaset의 event목록을 조회해보도록 하겠습니다.  
```sh
$ kubectl scale rs my-app-replicaset --replicas=1
$ kubectl describe rs my-app-replicaset
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_describe_rs_my-app-replicaset.png)
이벤트 히스토리를 확인해보면 3개의 pod가 생성되었다가 `replicas=1`로 설정을 변경함에 따라 2개의 pod가 삭제되었음을 확인할 수 있습니다.  

모든 과정을 확인했다면, 생성했던 replicaset을 삭제하도록 하겠습니다.  
```sh
$ kubectl delete rs my-app-replicaset
```

# ReplicaSet - Pod 이미지 변경을 통한 롤백
다음으로는 ReplicaSet을 이용해 배포한 pod를 롤백하는 방법에 대해 알아보도록 하겠습니다.  
Pod를 롤백해야하는 상황은 주로 실행중인 pod에서 장애나 결함이 발견되었을 때이며, 이런 경우에는 주로 이전 버전으로 롤백을 진행하게 됩니다.  

그렇다면 Pod를 롤백시키는 방법에는 어떤 것이 있을까요?  
앞서서 살펴본 실습 방법을 활용해 Pod를 롤백시킬 수 있습니다.  
앞에서 수행해본 것처럼 ReplicaSet을 배포하기 위해 작성한 yaml 파일에서 Pod Template 이미지를 문제가 없었던 버전의 이미지로 변경한 후 replicas를 조정하거나 기존재한 pod들을 삭제시켜 이전 이미지를 사용하는 Pod들이 다시 배포될 수 있도록 조치하는 것입니다.  

## 오류 발생 pod 배포
먼저 오류가 발생하는 버전의 이미지를 이용해 replicaset에 대한 bad-replicaset.yaml파일을 작성하고, 해당 내용을 배포해보도록 하겠습니다.  
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: yoonjeong/my-app:2.0-unhealthy
          ports:
            - containerPort: 8080
```

ReplicaSet을 배포하기 위해 아래 명령어를 수행합니다.  
```sh
$ kubectl apply -f bad-replicaset.yaml
```
pod와 replicaset을 조회해보면 yaml파일의 내용대로 잘 배포가 되었음을 확인할 수 있습니다.   
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_apply_-f_bad-replicaset.yaml.png)
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_apply_-f_bad-replicaset.yaml.png)
위의 yaml파일을 통해 배포된 pod 내의 컨테이너들은 `yoonjeong/my-app:2.0-unhealthy` 이미지를 기반으로 동작합니다.  

배포된 컨테이너에 오류를 발생시키기 위해 컨테이너의 8080 포트를 포트포워딩 하도록 하겠습니다.  
```sh
# port-foward 명령어: kubectl port-forward rs/<replicaset-name> <host-port>:<container-port>
$ kubectl port-forward rs/my-app-replicaset 8080:8080
```
포트포워딩 된 컨테이너에 5회 반복해서 접근해보도록 하겠습니다.  
```sh
for i in {0..5};
do curl localhost:8080;
done
```
5회 연속해서 접근한 결과 컨테이너 내에서 오류를 반환한 것을 확인할 수 있습니다.  
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/curl_5_times_to_crashed_container.png)

## label 변경을 통한 pod 이미지 업데이트
새로 배포된 이미지에서 문제가 발생한다는 것을 확인했으므로, 배포된 pod의 컨테이너에서 사용되는 이미지 버전을 이전 버전으로 변경하도록 하겠습니다.  
문제가 없었던 이전 버전인 `my-app:1.0` 이미지를 기반으로 동작하는 컨테이너를 가진 Pod들을 새로 배포하도록 하겠습니다.   
이 때 실행중인 Pod는 변경하지 않고 pod template의 이미지만 변경해보도록 하겠습니다.  
```sh
# 이미지 변경 명령어: kubectl set image rs/<replicaset-name> <container-name>=<image>
$ kubectl set image rs/my-app-replicaset my-app=yoonjeong/my-app:1.0
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_set_image_1.0_ver.png)
replicaset에서 사용하는 이미지가 잘 변경되었음을 확인할 수 있습니다.  
그러나 이미 관리되고 있는 Pod들의 개수가 replicas의 개수와 같으므로 `my-app:1.0` 이미지를 가지고 동작하는 pod는 하나도 새로 생성되지 않을 것입니다.  

이를 확인해보기 위해 앞서 `bad-replicaset.yaml`을 배포하면서 생성된 pod 목록과 pod의 owner 정보를 확인해보도록 하겠습니다.  
```sh
$ kubectl get pod -o wide

# pod의 Owner 오브젝트 확인: kubectl get pod <pod-name> -o jsonpath="{.metadata.ownerReferences[0].name}"
$ kubectl get pod <pod_name> -o jsonpath="{.metadata.ownerReferences[0].name}"
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/check_owner_of_pod_before_change_label.png)
새로운 이미지를 이용해 replicaset을 새로 배포했으나 여전히 이전에 생성된 pod의 owner로 replicaset의 이름인 `my-app-replicaset`이 출력되는 것을 확인할 수 있습니다.  

변경된 이미지를 이용한 pod를 띄우기 위해서 기존에 떠있던 pod들의 label을 변경하는 방식을 활용할 수 있습니다.  
pod의 레이블을 변경하면 replicaset의 `spec.selector.matchLabels`에 정의된 값과 기존 pod의 label값이 맞지 않기 때문에 ReplicaSet은 관리되고있는 pod의 개수가 감소한 것으로 인식하고 새로운 pod들을 띄울 것이기 때문입니다.  
따라서 위에서 확인한 Pod의 label 값을 변경해보도록 하겠습니다.  
```sh
# pod 레이블 변경 명령어: kubectl label pod <pod-name> <label-key>=<label-value> --overwrite
$ kubectl label pod <pod_name> app=to-be-fixed --overwrite
```
label을 변경한 후에 다시 pod의 owner를 조회해보면 다음처럼 아무 값도 조회되지 않는 것을 알 수 있습니다.  
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/check_pod_owner_after_change_label.png)
owner 오브젝트의 값이 더이상 조회되지 않는 이유는 replicaset이 관리하는 pod들은 label 값이 `app=my-app`인 label들인데, 위에서 해당 pod의 label 값을 `app=to-be-fixed`로 바꿨기 때문에 replicaset의 관리 범주에서 해당 pod가 빠졌기 때문입니다.  

replicaset을 배포할 때 `replicas` 속성을 `3`으로 설정했기 때문에 replicaset은 라벨이 변경된 pod를 관리 범주에서 제외하고 개수를 맞추기 위해 새로운 pod를 생성하려고 할 것입니다.  
따라서 pod 목록을 다시 조회해보도록 하겠습니다.  
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_get_pod_after_change_label.png)
label을 바꾼 pod로 인해 `app=my-app` label을 갖는 새로운 pod가 하나 생성되었음을 확인할 수 있습니다.  

문제가 되었던 이미지를 기반으로 동작하고 있는 나머지 2개의 pod들도 label값을 변경하여 replicaset이 관리하는 모든 Pod가 새로운 이미지를 기반으로 동작하도록 수정해보도록 하겠습니다.  
```sh
$ kubectl label pod <pod_name_1> app=to-be-fixed --overwrite
$ kubectl label pod <pod_name_2> app=to-be-fixed --overwrite
```

모든 pod들을 `my-app:1.0` 이미지를 이용하도록 변경하여 새로 생성하였으므로 다시 8080번 포트로 포트포워딩을 하고 curl 명령어를 통해 컨테이너에 접근해보도록 하겠습니다.  
```sh
$ kubectl port-forward rs/my-app-replicaset 8080:8080
$ curl localhost:8080
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/curl_localhost_8080_after_rollback_image.png)
버전 1 이미지의 응답이 정상적으로 오는 것을 확인할 수 있습니다.  

### replicas 조정을 통한 pod 이미지 업데이트
새로운 이미지를 사용하는 pod를 배포하는 방법에는 앞서 살펴봤듯이 이미 배포되어있는 Pod들의 label을 변경하여 ReplicaSet의 관리 범주에서 제외하는 방법 외에도 replicas를 조정하여 변경사항이 반영된 pod들을 신규 생성하는 방법을 이용할 수 있습니다.   

확인해보기 위해 기존에 생성해둔 pod와 replicaset을 모두 삭제하고 앞서 작성했던 `bad-replicaset.yaml`을 다시 배포하도록 하겠습니다.  
```sh
# 리소스 삭제
$ kubectl delete rs my-app-replicaset
$ kubectl delete pod --all

# replicaset 재배포
$ kubectl apply -f bad-replicaset.yaml
```

오류를 발생시키기 위해 앞에서와 동일하게 8080 포트로 포트포워딩을 진행하고 5번의 curl 명령을 통해 컨테이너에 접근해보도록 하겠습니다.  
```sh
# 포트포워딩 진행
$ kubectl port-forward rs/my-app-replicaset 8080:8080

# pod 접근
$ for i in {1..5};
$ do curl localhost:8080;
$ done
```


이전에 확인했던 것처럼 오류가 반환되는 것을 확인할 수 있습니다.  

따라서 문제가 없는 `yoonjeong/my-app:1.0` 이미지로 replicaset에서 사용하는 이미지를 변경하도록 하겠습니다.  
```sh
$ kubectl set image rs/my-app-replicaset my-app=yoonjeong/my-app:1.0
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/change_image_to_ver1.0.png)

replicaset의 이미지 변경이 잘 이루어졌다면 replicas의 개수를 0으로 조정하여 Pod 개수를 0개로 줄인 후 다시 3개로 늘려보도록 하겠습니다.  
```sh
# ReplicaSet replicas 수 변경: kubectl scale rs/<replicaset-name> --replicas <number-of-pods>
$ kubectl scale rs my-app-replicaset --replicas 0
$ kubectl scale rs my-app-replicaset --replicas 3
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/kubectl_scale_to_0_and_3.png)

변경 사항이 잘 반영되었는지 8080 포트로 포트포워딩을 진행 후 curl 명령어를 통해 pod에 접근해보도록 하겠습니다.  
```sh
# 포트포워딩
$ kubectl port-forward rs/my-app-replicaset 8080:8080

# pod 접근
$ curl localhost:8080
```
![](/assets/img/2023/06/2023-06-10-kubernetes_replicaset_pods_practice/curl_after_deploying_new_pod_by_scaling.png)   
이전 버전의 이미지로 롤백이 잘 이루어졌음을 확인할 수 있습니다.  

