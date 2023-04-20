# 쿠버네티스 Label, Selector

## Label과 Selector란

**Label**
- 쿠버네티스 오브젝트를 식별하기 위한 key/value 쌍의 메타정보
- 쿠버네티스 리소스를 논리적인 그룹으로 나누기 위해 붙이는 이름표

**Selector**
- Label을 이용해 쿠버네티스 리소스를 선택하는 방법
- Label을 이용해 쿠버네티스 리소스를 필터링하고 원하는 **리소스 집합**을 구하기 위한 label query

Label과 Selector는 사용자가 쿠버네티스 리소스를 조회하기 위해 selector로 label값을 이용한 명령을 내리면, 쿠버네티스는 해당하는 쿠버네티스 리소스 집합을 구해서 리턴해주는 형식으로 사용하게 됩니다.  

### 리소스 집합을 구해야하는 상황  
다음으로는 어떤 상황에서 사용자가 특정 리소스 집합을 요청하게되는 지 알아보도록 하겠습니다.  

하나의 클러스터에서 서로 다른 서비스를 제공하는 수백개의 Pod가 동시에 실행되고 있다고 합시다.  
이 때 서비스 A의 트래픽을 A Pod로, 서비스 B의 트래픽을 B Pod로 라우팅해야한다면 각 서비스의 Pod가 어떤 Pod에 해당한지 알 필요성이 발생합니다.  
즉, 원하는 Pod의 집합을 Label 속성을 이용해 구할 수 있어야 하는 것입니다.  

또 다른 상황으로는 서비스 A에 대한 트래픽이 증가하는 상황을 들 수 있습니다.  
트래픽 증가로 인해 A 서비스 Pod를 수평 확장하려고 할 때 역시 어떤 Pod를 늘릴 지 알고 복제할 수 있어야하므로 서비스 A의 집합을 구해야 할 필요성이 생기게 됩니다.  

즉 정리해보면 어떤 리소스를 선택해서 명령을 실행하고지 할 때 Label과 Selector를 이용하게 됩니다.  


## Label
### Label 표현 방법  
label은 오브젝트 생성 시 yaml파일에 선언할 수 있습니다.  
label을 포함해 pod를 생성하는 yaml파일의 예시를 살펴보도록 하겠습니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: backend
    version: v1
    env: prod
spec:
  containers:
  - image: my-pod
    name: my-pod
```
위의 yaml 파일에서 metadata의 labels 영역을 확인하면 app, version, env 값을 갖습니다.  
이 yaml 파일을 통해 생성된 `my-pod` 라는 pod는 이 3가지 label 속성을 통해 식별될 수 있게 되는 것입니다.  

### Label을 포함한 Pod 생성
뒤에서 Label과 Selector를 이용한 명령어들을 수행해보기 위해 총 4개의 pod를 띄워보도록 하겠습니다.  

생성할 pod는 dog-app, cat-app, bird-app, tree-app 총 4가지 입니다.  
추후 pod label에 정의할 key는 group, location, species, legs 입니다.  
Pod 배포를 위한 yaml파일의 내용은 다음과 같습니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dog-app
  labels:
    group: animal
spec:
  containers:
  - name: dog-app
    image: ubuntu:20.04
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
```
위의 파일은 `dog-app` pod를 생성하기 위한 yaml 파일이며, `cat-app`, `bird-app` 에 대한 yaml 파일은 동일하게 작성하되, `metadata.name` 필드와 `spec.containers.name` 필드만 각각 `cat-app`, `bird-app`로 바꿔주도록 하겠습니다.  

`tree-app`에 대한 yaml 파일은 `metadata.labels`의 `group` 값을 `plant`로 변경해 작성하도록 하겠습니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tree-app
  labels:
    group: plant
spec:
  containers:
  - name: tree-app
    image: ubuntu:20.04
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
```

위의 yaml파일을 통해 pod를 생성하기 위해 다음 명령어를 수행하도록 하겠습니다.  
```sh
$ kubectl apply -f dog-app.yaml
$ kubectl apply -f cat-app.yaml
$ kubectl apply -f bird-app.yaml
$ kubectl apply -f tree-app.yaml
```

생성된 pod를 pod 별 label 값과 함께 확인해보면 다음과 같습니다.  
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/kubectl_apply_get_pods.png)

### Label 관련 명령어
생성된 pod의 label 속성을 확인해보기 위해서는 아래 명령어를 사용할 수 있습니다.  
```sh
# kubectl get <리소스 종류> <리소스 이름> --show-labels
$ kubectl get pod dog-app --show-labels
```

또한 새로운 label 추가를 하고싶다면 다음 명령어를 사용할 수 있습니다.  
```sh
# kubectl label <리소스 종류> <리소스 이름> key=value
$ kubectl label pod dog-app location=ground
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/add_label.png)

동일한 Label 키에 대해 값을 수정하고 싶을 때는 `--overwrite` 옵션을 주어 값을 변경해야 합니다.  
```sh
# 값 설정
$ kubectl label pod bird-app location=ground

# 값 변경
$ kubectl label pod bird-app location=sky --overwrite
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/set_label_overwrite.pn

여러 개의 Label 중 특정 값만을 확인하고 싶다면 아래 명령어를 통해 원하는 key값을 명시해주면 됩니다.  
```sh
# kubectl get <리소스종류>/<리소스이름> --label-columns key1,key2,key3 (--label_columns 대신 -L 옵션을 주어 확인도 가능)
$ kubectl get pod/bird-app --label-columns group,location
$ kubectl get pod/bird-app --L group,location
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/get_pod_label_columns.png)

설정된 label을 삭제하고 싶을 때는 다음 명령어를 사용하면 됩니다.  
```sh
# kubectl label <리소스종류>/<리소스이름> <삭제할 key값>-
$ kubectl label pod/tree-app leg-
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/delete_label.png)

## Selector  
label을 통해 리소스 집합을 구하기 위해 Selector를 사용하는 방법에 대해 알아보도록 하겠습니다.  

selector를 이용하기 전에 다양한 쿼리에 대해 서로 다른 결과를 확인하기 위해 다음 명령어들을 통해 label들을 추가적으로 등록해두도록 하겠습니다.  
```sh
# dog-app
$ kubectl label pod dog-app group=animal location=ground species=dog legs=four

# cat-app
$ kubectl label pod cat-app group=animal location=ground species=cat legs=four

# bird-app
$ kubectl label pod bird-app group=animal location=sky species=bird legs=two

# tree-app
$ kubectl label pod tree-app group=plant location=ground species=tree
```

조회해보면 최종적으로 label은 다음과 같이 설정되어 있습니다.  
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/labels_all_set.png)

### Selector 관련 명령어
아래는 selector를 통해 리소스 집합을 구하는 명령어입니다.  
```sh
# label query는 key=value 형식으로 작성  
$ kubectl get <오브젝트 타입> --selector <label query1, ..., label query N>
$ kubectl get <오브젝트 타입> --l <label query1, ..., label query N>
```

**Selector 등호 연산자**   
Selector를 통해 쿼리를 질의할 때 사용할 수 있는 연산자로는 `=`와 `!=`가 있습니다.  
- `=`는 key = value 형식으로 key의 값이 value인 label을 가진 리소스를 가져오고자 할 때 쓰입니다.  
- `!=`는 key != value 형식으로 key의 값이 value가 아닌 리소스를 가져오고자 할 때 쓰입니다.  

따라서 selector를 통해 특정 label 값을 갖는 pod 리소스들을 구하기 위해서는 다음과 같은 명령어를 사용하면 됩니다.  
```sh
# `env` 값이 prod인 리소스만 구하기
$ kubectl get pod --selector group=animal
# `env` 값이 prod가 아닌 리소스만 구하기
$ kubectl get pod --selector group!=animal
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_=_!=.png)

특정 pod에 레이블을 추가하는 방법에 대해서도 알아보도록 하겠습니다.  
아래는 dog-app, cat-app 2개의 pod에 `move=run`라는 레이블을 추가하는 명령어입니다.  
```sh
$ kubectl label pod dog-app cat-app move=run
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/add_label_to_multiple_pods.png)

**Selector 집합 연산자**  
label의 특정 key의 value가 어떠한 value 집합에 속하는 지 아닌 지를 통해서도 selector를 이용한 쿼리 질의가 가능합니다.  
- `'key in (value1, value2, ...)'`: key의 value가 (value1, value2, ...) 내에 속하는 리소스의 집합을 가져옵니다.  
- `'key notin (value1, value2, ...)'`: key의 value가 (value1, value2, ...) 내에 속하지 않는 리소스의 집합을 가져옵니다.  

집합 연산자를 이용해 selector를 통한 쿼리 질의를 한 명령어 예시는 아래와 같습니다.  
```sh
# `species` 값이 cat,dog에 속하는 리소스만 구하기
$ kubectl get pod --selector 'species in (cat,dog)'
# `species` 값이 cat,dog에 속하지 않는 리소스만 구하기
$ kubectl get pod --selector 'species notin (cat,dog)'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_in_notin.png)

또한 특정 key의 레이블을 갖는지 아닌지 여부를 가지고도 selector를 통해 질의할 수 있습니다.  
- `key`: label에 key가 있을 때
- `!key`: label에 key가 없을 때

키만 명시하여 해당 키를 가진 리소스만 가져오도록 쿼리 질의를 한 명령어 예시는 아래와 같습니다.  
```sh
# `legs` key를 갖는 리소스만 구하기
$ kubectl get pod --selector legs
# `legs` key를 갖지 않는 리소스만 구하기
$ kubectl get pod --selector '!legs'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_with_without_key.png)

마지막으로 앞서 나온 여러 쿼리 방식을 조합한 명령어 예시를 확인해보도록 하겠습니다.  
```sh
# group key의 값이 animal이며 species가 dog나 bird인 리소스
$ kubectl get pod --selector 'group=animal,species in (dog,bird)'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_group_species.png)

```sh
# legs라는 키를 가지고 있으며 legs의 값은 4가 아닌 리소스
$ kubectl get pod --selector 'legs,legs notin (four)'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_legs_legs_notin_four.png)

```sh
# legs 키의 값이 4가 아닌 리소스 (legs라는 키를 아예 갖지 않는 리소스들도 함께 출력됨)
$ kubectl get pod --selector 'legs notin (four)'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/selector_legs_notin_four.png)

마지막으로 생성했던 pod들을 삭제하기 위해 delete 명령어를 사용할 때도 selector를 통해 원하는 리소스만 골라와 삭제할 수 있습니다.  
```sh
$ kubectl delete pod --selector 'group'
```
![](/assets/img/2023/04/2023-04-15-kubernetes_label_selector/kubectl_delete_pod_with_selector.png)


## nodeSelector로 선택한 특정 노드 집합에 Pod 배포  
nodeSelector라는 속성을 pod에 추가하여 특정 label을 갖는 노드에만 pod를 배포하는 방법에 대해 알아보도록 하겠습니다.  

### 특정 노드에 pod를 배포하기 위한 정의 사항 
원하는 node를 골라 해당 node에 pod를 배포하기 위해서는 배포 시점에 아래와 같은 설정을 해주어야 합니다.  
1. 노드에 Label 추가: 노드를 Selector로 선택하기 위해 Label 값이 필요합니다.
2. Pod yaml 파일에 spec.nodeSelector를 선언: 쿠버네티스가 Pod를 생성할 때 nodeSelector를 보고 pod를 배포할 노드를 선택합니다.

위와 같은 설정이 되어있으면, Pod를 배포할 때 `scheduler`가 선언된 nodeSelector 정보를 이용해 node를 선택합니다.  

이러한 점을 참고하여 특정 노드에만 pod을 배포해보기 위해 `dog-app` 이라는 pod를 `env: animal`이라는 레이블을 가진 node에만 배포해보도록 하겠습니다.  

#### 노드 목록 조회 및 노드에 Label 추가  
노드 목록 조회 명령어는 다음과 같습니다.  
```sh
$ kubectl get node
```
![](/assets/img/2023/04/2023-04-21-kubernetes_deploy_pod_with_nodeSelector/kubect_get_node.png)

조회된 node에 label을 추가하는 명령어는 다음과 같습니다.  
```sh
# kubectl label node <노드 Name> <label_key=label_value>
$ kubectl label node gke-my-cluster-default-pool-18f2f739-7h0n env=animal
```
![](/assets/img/2023/04/2023-04-21-kubernetes_deploy_pod_with_nodeSelector/kubectl_label_node.png) . 

node에 label이 잘 생성되었는 지 확인해보도록 하겠습니다.  
```sh
$ kubectl get node -L env
```
![](/assets/img/2023/04/2023-04-21-kubernetes_deploy_pod_with_nodeSelector/kubect_get_node_-L_env.png)

#### Pod 배포  
다음으로는 dog-app pod를 배포하도록 하겠습니다.   
pod는 yaml파일을 작성해서 `kubectl apply -f <파일명>` 명령어를 통해 배포할 수도 있지만 다음 명령어로 간단하게 배포할 수도 있습니다.  
```sh
$ kubectl run <pod name> --image <image name>
```

위의 명령어를 이용하여 `env=animal` 레이블을 가진 모든 노드에 pod을 배포하기 위해 아래 명령어를 사용합니다.  
```sh
$ kubectl run dog-app-1 \
  --labels="species=dog" \
  --image=ubuntu:14.04 \
  --overrides='{"spec":{"nodeSelector":{"env":"animal"}}}'   # 원하는 spec을 json 형태로 표현 가능
```

같은 방법으로 dog-app-1 ~ dog-app-5 까지 생성 후에 `kubectl get pod -o wide`로 pod 목록을 확인해보도록 하겠습니다.  
![](/assets/img/2023/04/2023-04-21-kubernetes_deploy_pod_with_nodeSelector/kubect_get_pod_-o_wide.png)

`env=animal`인 node 2개에만 pod가 배포되고 있음을 확인할 수 있습니다.  
