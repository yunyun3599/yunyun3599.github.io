# 쿠버네티스 서비스 - Cluster IP 방식으로 Pod 띄우기

## Cluster IP 타입의 서비스  
Cluster IP 타입의 서비스는 Internal IP를 할당 받아 단일 엔드포인트로 파드 집합을 노출 할 수 있게 해줍니다.   
하지만 External IP가 아니기 떄문에 클러스터 외부에서 접근이 불가능합니다.   
요청이 들어왔을 때 Cluster IP 타입의 서비스가 동작하는 방식은 아래 그림과 같습니다.   
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/service_cluster_ip_diagram.png)  

Cluster IP 타입의 서비스 동작 방식을 확인해보기 위해 다음 내용들에 대해 실습을 진행해보도록 하겠습니다.  
1. ClusterIP 타입의 Service 생성  
2. 환경변수를 설정하여 Service IP로 파드 집합에 접근  
3. 쿠버네티스 DNS 서버를 이용하여 Service 이름으로 파드 집합에 접근  
4. MSA 환경에서 다른 네임스페이스에 있는 Service를 도메인 이름으로 접근  

## Service 생성 및 Endpoints 조회
서비스 생성 실습을 위해 order라는 서비스와 payment라는 Cluster IP 타입의 서비스 2개를 생성해보도록 하겠습니다.  
각 서비스를 생성하면 order라는 endpoints와 payment라는 endpoints가 함께 생성됩니다.   

### Service 오브젝트 선언  
Cluster IP 타입으로 `order`라는 서비스 오브젝트를 선언하기 위해 아래 내용으로 `order.yml` 파일을 작성합니다.  
```yml
apiVersion: v1
kind: Service
metadata:
  name: order
  namespace: snackbar     # snackbar라는 네임스페이스 안에 서비스를 생성
  labels:
    service: order
    project: snackbar
spec:
  type: ClusterIP         # Service의 디폴트 타입이 ClusterIP이므로 따로 선언하지 않아도 ClusterIP로 생성됨
  selector:               # 파드 집합 선택자
    service: order
    project: snackbar
  ports:
    - port: 80            # 노출할 서비스 포트
      targetPort: 8080    # 연결할 컨테이너 포트
```

위와 거의 유사한 형식으로 `payment` 서비스를 생성하기 위한 `payment.yml` 파일을 다음과 같이 작성해줍니다.   
```yml
apiVersion: v1
kind: Service
metadata:
  name: payment
  namespace: snackbar     # snackbar라는 네임스페이스 안에 서비스를 생성
  labels:
    service: payment
    project: snackbar
spec:
  type: ClusterIP         # Service의 디폴트 타입이 ClusterIP이므로 따로 선언하지 않아도 ClusterIP로 생성됨
  selector:               # 파드 집합 선택자
    service: payment
    project: snackbar
  ports:
    - port: 80            # 노출할 서비스 포트
      targetPort: 8080    # 연결할 컨테이너 포트
```

다음으로는 `order` 서비스를 위한 `deployment`를 배포하기 위해 다음과 같은 `order_deployment.yml` 파일을 작성해줍니다.   
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
  namespace: snackbar
  labels:
    service: order
    project: snackbar
spec:
  replicas: 2
  selector:
    matchLabels:
      service: order
      project: snackbar
  template:
    metadata:
      labels:
        service: order
        project: snackbar
    spec:
      containers:
      - name: order
        image: yoonjeong/order:1.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "64Mi"
            cpu: "50m"
```   

위와 유사하게 `payment` 서비스의 `deployment`를 위한 `payment_deployment.yml` 파일을 작성해줍니다.   
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
  namespace: snackbar
  labels:
    service: payment
    project: snackbar
spec:
  replicas: 2
  selector:
    matchLabels:
      service: payment
      project: snackbar
  template:
    metadata:
      labels:
        service: payment
        project: snackbar
    spec:
      containers:
      - name: payment
        image: yoonjeong/payment:1.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: "64Mi"
            cpu: "50m"
```   

위에서 작성한 yml파일을 이용해 아래 명령어들로 오브젝트들을 생성합니다.  
```sh
$ kubectl create namespace snackbar
$ kubectl apply -f order.yml
$ kubectl apply -f order_deployment.yml
$ kubectl apply -f payment.yml
$ kubectl apply -f payment_deployment.yml
```

### Service 관련 오브젝트 조회
모든 오브젝트를 생성했다면 다음 명령어를 이용해 `snackbar`라는 네임스페이스 내에 있는 모든 오브젝트들을 조회할 수 있습니다.  
```sh
$ kubectl get all -n snackbar
```
![](2023-10-09-17-02-12.png)
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_get_all_-n_snackbar.png)


생성한 `service`에 대한 상세 내용을 조회하고 싶으면 아래 명령어를 이용할 수 있습니다.  
```sh
$ kubectl get svc -o wide -n snackbar
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_get_svc_-o_wide_-n_snackbar.png)  
결과로 확인할 수 있듯 Cluster IP는 할당되었으나 외부에서 접근 가능하도록 하는 External IP는 할당되지 않았음을 확인할 수 있습니다.  


다음으로는 `snackbar` 네임스페이스의 모든 `Endpoints` 리소스를 확인해보도록 하겠습니다.  
Endpoints는 서비스를 배포하고 나면 쿠버네티스가 자동적으로 서비스의 selector에 정의되어있는 pod들의 주소 목록을 구성하는 것입니다.  
```sh
$ kubectl get endpoints -n snackbar
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_get_endpoints_-n_snackbar.png)  
위의 결과로 리턴된 IP 주소들은 order 서비스의 경우 해당 서비스에 해당하는 pod들의 IP 주소이며 payment 서비스 역시 해당 서비스의 pod들의 주소입니다.  
확인을 위해 아래 명령어를 통해 pod의 주소를 확인해보도록 하겠습니다.   
```sh
$ kubectl get pods -n snackbar -o wide
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubect_get_pods_-n_snackbar_-o_wide.png)  
조회된 pod들이 앞서 확인한 IP 주소들을 가지고 있음을 확인할 수 있습니다.  

> 서비스로 요청이 들어오는 경우 `Endpoints`에 정의되어있는 주소 맵핑을 통해 request를 처리하므로 service selector와 매치되는 파드 집합이 없는 경우 `Endpoints`가 제대로 구성되지 않아 요청에 실패할 수 있습니다.  

### 접속 테스트  
`service`와 `deployment`가 잘 배포되었으므로 이제 할당된 주소로 접속이 잘 이뤄지는 지 테스트해보도록 하겠습니다.  
먼저 로컬 머신에서 Cluster IP 타입의 Service IP로 접근이 되는지 확인해보도록 하겠습니다.  
```sh
$ curl <service의 Cluster IP 주소>
# 또는
$ curl $(kubectl get svc order -o jsonpath="{.spec.clusterIP}" -n snackbar)
$ curl $(kubectl get svc payment -o jsonpath="{.spec.clusterIP}" -n snackbar)
```
위의 명령어는 정상 동작하지 않을텐데요, 이는 앞서 확인했듯 Cluster IP로 구성된 서비스는 클러스터 외부에서 접근이 불가능하기 때문입니다.  

그러므로 다음으로 포트포워딩을 통해 Service IP로 접속하여 pod로 요청이 전달되는 지 확인해보도록 하겠습니다.  
```sh
# 외부에서 8080으로 접속하면 서비스의 80 포트로 연결
$ kubectl port-forward service/order -n snackbar 8080:80
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/order_-n_snackbar_8080_80.png)

그 후 로컬호스트의 8080 포트로 접속하면 서비스에서 응답이 오는 것을 확인할 수 있습니다.  
```sh
$ curl localhost:8080
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/curl_localhost_8080_order.png)


## 환경변수를 활용한 Service 호출  
다음으로는 특정 pod에서 다른 pod를 호출하고자 할 때 서비스를 통해 요청을 전달하는 방법 중 환경변수를 이용하는 방법에 대해 알아보도록 하겠습니다.  
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/access_to_pod_using_environment_variable.png)   
위의 그림처럼 `payment` 서비스를 먼저 생성한 후에 `order` pod를 띄우면 `order` pod 컨테이너 내에는 `payment` 서비스에 대한 Internal IP값을 `PAYMENT_SERVICE_HOST`로, 서비스의 포트를 `PAYMENT_SERVICE_PORT`로 갖게 됩니다.    
따라서 order 컨테이너에서 위의 두 환경 변수를 활용하여 `payment` Service IP와 Port로 `Payment`를 호출하고 응답을 확인할 수 있습니다.   

### order 컨테이너 환경변수 확인  
order 컨테이너 내부의 환경변수를 확인하기 위해서는 아래 명령어를 사용할 수 있습니다.  
```sh
# order 컨테이너 환경 변수 확인
$ kubectl exec <order pod name> -n snackbar -- env | grep PAYMENT
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_exec_order_--env_grep_payment.png)

만약 pod 내에 여러개의 컨테이너가 생성되어 있다면 `-c <container name>` 옵션을 통해 특정 컨테이너로 커맨드를 보낼 수 있습니다.  
```sh
$ kubectl exec <pod name> -c <container name> -n <namespace> -- <command>
```
`-c <container name>` 옵션을 주지 않은 경우에는 pod 내에서 가장 처음으로 생성된 container로 명령어가 전달됩니다.  

### order 컨테이너 sh 접속 후 payment 파드로 요청 보내기  
order 컨테이너에 쉘 접속을 위해 아려 명령어를 사용해보도록 하겠습니다.  
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- sh
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubect_exec_-ti_pod_--_sh.png)

위와 같이 접속에 성공했으면 다음 명령어를 이용해 order 컨테이너 내에서 payment의 pod로 요청을 보내고 응답을 확인해보도록 하겠습니다.  
```sh
$ curl $PAYMENT_SERVICE_HOST:$PAYMENT_SERVICE_HOST
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/curl_$PAYMENT_SERVICE_HOST:$PAYMENT_SERVICE_PORT.png)

다음과 같이 요청에 대한 응답이 잘 오는 것을 확인할 수 있습니다.  

더 응용하여 curl 커맨드를 이용해 payment 서비스에서 로드밸런싱이 잘 동작하고 있는 지 확인해볼 수도 있는데요, 다음 명령어를 통해 여러번 payment 서비스를 호출해보도록 하겠습니다.   
```sh
for i in `seq 1 10`; do curl -s $PAYMENT_SERVICE_HOST:$PAYMENT_SERVICE_PORT; done
```   
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/check_loadbalancing.png)
위와 같이 두 개의 pod에 분산되어 요청이 들어가는 것을 확인할 수 있습니다.  

## 서비스 이름을 이용해 Service를 호출하는 방법 - 쿠버네티스 DNS 서버   
다음으로는 쿠버네티스 DNS 서버를 이용해 서비스 이름을 이용해 서비스를 호출하는 방법에 대해 알아보도록 하겠습니다.  
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/call_by_service_name_diagram.png)   
위의 그림처럼 서비스 이름을 이용해 다른 서비스에 접근 할 때는 다음 단계를 통해 접근하게 됩니다.  
1. order 컨테이너의 `/etc/hosts, /etc/resolv.conf` 파일을 확인
    ```
    # /etc/resolv.conf
    search snackbar.svc.cluster.local
    nameserver 10.80.0.10
    ```
2. Service 이름, 'payment'로 Payment 호출, 응답 확인
> 쿠버네티스에서 사용하는 도메인 이름 규칙 - FQDN(Fully-Qualified Domain Name)

### 'payment' 서비스 이름으로 order -> payment 파드 요청 / 응답 확인  
서비스 이름으로 payment 파드에 요청을 보내기 위해 다음 명령어를 수행시켜보도록 하겠습니다.  
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- curl -s payment:80
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_exec_-ti_curl_-s_payment:80.png)

### 추가 사항 확인 - /etc/hosts, /etc/roslov.conf
다음 명령어를 통해 order 컨테이너의 `/etc/hosts` 파일 내용을 확인해보도록 하겠습니다.  
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- cat /etc/hosts
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubect_exec_-ti_cat_etc_hosts.png)
결과로 조회되는 정보로는 자기 자신의 호스트와 IP만 있음을 확인할 수 있습니다.  
그렇다면 order 컨테이너에서는 어떻게 `hosts` 파일에 정의되지 않은 도메인 이름으로 요청을 실행할 수 있는 것일까요?  

이처럼 `hosts` 파일에 정의되지 않은 도메인 이름으로 요청을 보낼 때 dns 서버를 활용하게 되므로 `kube-system` 네임스페이스의 모든 `kube-dns` 리소스를 조회하여 실제로 dns 서버가 동작중인지 확인해보도록 하겠습니다.  
```sh
$ kubectl get all -n kube-system | grep kube-dns
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/kubectl_get_all_-n_kube-system_|_grep_kube-dns.png)
위의 조회 결과에서 `kube-dns` 서비스가 `34.118.224.10` 주소로 동작하고 있음이 잘 조회되는 것을 확인할 수 있습니다.  

다음으로는 `/etc/resolv.conf` 파일을 확인해보도록 하겠습니다.   
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- cat /etc/resolv.conf
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/cat_resolv.conf.png)
여기에서 `nameserver`는 컨테이너에서 사용할 DNS 서버 주소를 의미합니다.   
`search`는 클러스터의 기본 도메인으로, DNS 서버에서 호스트 네임을 찾을 때 사용할 검색 리스트입니다.  
쿠버네티스가 Pod를 생성할 때 컨테이너의 `/etc/resolv.conf` 파일에 nameserver와 search를 설정하게 됩니다.  

### Service 이름으로 DNS 쿼리하는 법  
Service 이름만 가지고 DNS에 쿼리하여 IP 주소를 알아내는 방식은 다음 과 같습니다.  
1. 파드 컨테이너에서 <서비스 이름>으로 요청을 실행  
2. `/etc/resolv.conf`에 정의된 search 경로를 붙여서 FQDN(Fully Qualified Domain Name)을 만듦
3. FQDN울 이용해 DNS 쿼리를 실행  
4. DNS 서버로부터 받은 Service IP를 이용해 요청을 완료  

이에 따라 kube-dns 파드에 order pod가 `http://payment` 형식으로 요청을 보내면 dns 서버는 DNS 쿼리를 실행해 결과로 얻은 `payment`의 Cluster IP 주소를 응답하게 됩니다.  

만약 생성된 full name인 FQDN으로 요청을 보낸다면 다음과 같은 형식이 됩니다.  
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- curl payment.snackbar.svc.cluster.local:80
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/curl_with_FQDN.png)

공통 접미사인 `.svc.cluster.local`을 떼고 다음과 같은 형식으로 요청하여도 동일한 결과를 얻을 수 있습니다.   
```sh
$ kubectl exec -ti <order pod name> -n snackbar -- curl payment.snackbar:80
```
![](/assets/img/2023/10/2023-10-09-kubernetes_service_clusterIP/curl_withoud_common_suffix.png)

앞에서 서비스 이름만 가지고 curl 명령어를 호출했던 것처럼 호스트 이름에 네임스페이스를 명시하지 않으면 호출하는 pod가 속한 네임스페이스에서 도메인에 해당하는 IP 주소를 찾게 됩니다.  



