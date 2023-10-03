# 쿠버네티스 서비스  

## 쿠버네티스 Service의 개념
### Pod의 한계  
서비스에 대해 알아보기 전에, Pod를 통해 애플리케이션을 관리할 때의 한계점에 대해 알아보도록 하겠습니다.  

1. Pod IP 변동성
- Pod는 새로 생성될 때마다 새로운 IP를 가지고 생성되기 때문에 Pod를 통해 서비스를 운영하게 되면 변동이 잦은 Pod의 IP를 항상 관리해야합니다.     
- 또한 이러한 특성으로 인해 만약 오프라인인 Pod의 IP에 클라이언트가 접근하려 한다면 요청은 실패하게 됩니다.   
- 이에 따라 특정한 파드 집합과 통신할 수 있는 단일 엔드포인트를 제공할 필요가 있게 됩니다.  

2. Pod IP 접근 범위
- Pod IP는 클러스터 내부에서만 접근이 가능하다는 특성도 있는데요, 이 때문에 클러스터 외부에서 접근 가능한 IP를 제공하는 것 역시 필요합니다.  

### Service 특성  
위와 같은 Pod의 한계를 보완할 수 있는 `Service`의 특성에 대해 알아보도록 하겠습니다.  

`Service`는 Pod를 추상화함으로써 단일 엔드포인트를 제공하며 로드밸런싱 기능을 제공합니다.   
`Service`를 사용하면 파드의 클라이언트는 `Service IP:Port`를 이용해 파드와 통신하게 됩니다.  
요청을 받은 `Service`는 Selector에 의해 선택된 파드 집합 중 임의의 파드로 트래픽을 포워딩하여 전달합니다.  
![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/service_pod_request_prcoess.png)

### Service 정의 방법  
Service 오브젝트를 정의하기 위해서는 다음 형식의 yaml 파일을 작성해야합니다.  
```yml
apiVersion: v1
kind: Service
metadata:
  name: order
  namespace: snackbar
  labels:
    app: order
spec:
  selector:
    app: order        # 유입된 트래픽을 전달할 Pod 집합
  ports:
  - port: 80          # 노출할 서비스 포트
    targetPort: 8080  # 서비스 포트와 연결할 컨테이너 포트 (containerPort와 일치해야함, pod 내에서 서비스가 떠있는 포트)
```

### Endpoints
`Endpoints`는 Service 객체를 생성하면 자동으로 생성되는 오브젝트 리소스입니다.  
`Endpoints`는 Service가 노출하는 Pod IP와 Port의 최신 목록을 관리하며, Service에 선언된 파드 집합이 변경될 때마다 Endpoints 목록도 업데이트됩니다.
시스템 운영 시에 Service가 트래픽을 받게 되면 이 트래픽을 `Endpoints` 중 하나로 리다이렉트 하게 됩니다.  
![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/kubernetes_service_endpoints.png)

파드 집합에 대한 단일 엔드포인트 서비스와 통신하는 데는 아래 2가지 방법이 있습니다.   
1. 컨테이너 환경변수에 Service IP와 Port를 설정해 활용하는 방법  
  - 쿠버네티스가 Pod를 생성할 때 컨테이너 환경변수에 모든 Service IP, Service Port를 등록  
    - `OOO_SERVICE_HOST`, `OOO_SERVICE_PORT`
  - 주의사항 
    1. 연결할 서비스를 클라이언트 Pod보다 먼저 생성해야함 (서비스가 생성되어 있어야 IP와 Port를 알 수 있음)   
    2. 다른 네임스페이스에 있는 서비스 환경변수는 설정되지 않음 (네임스페이스가 다르면 논리적으로 분리되어있기 때문에 외부에 있는 서비스이므로 IP와 Port를 알 수 없음) 
2. Service 이름으로 DNS 서버에 질의하여 Servie IP를 알아내는 방법  
  - 쿠버네티스를 실행하면 DNS 서버에 해당하는 Pod를 실행하게 되며 DNS 서버의 IP 주소를 컨테이너의 `/etc/resolve.conf` 파일에 등록함  
    ```conf
    # /etc/resolve.conf
    search snackbar.svc.cluster.local
    nameserver 10.80.0.10
    ```
  - 컨테이너에서 서비스의 이름으로 DNS 서버에 IP를 요청하면 해당 IP 주소를 리턴해주며, 컨테이너는 리턴받은 주소로 요청을 보냄  
    ![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/service_dns_server_diagram.png)


## Service 타입  
### Service 접근 범위에 따른 분류  
Service의 타입에 따라 클라이언트가 서비스에 접근할 수 있는 방식이 달라집니다.  

접근 범위에 따라 Service 타입을 분류하면 아래 3가지 타입으로 서비스를 분류할 수 있습니다.   
여기에서 LoadBalancer 타입은 NodePort와 ClusterIP 타입을 포함합니다.  
![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/service_type.png)  
1. LoadBalancer 
  - 클라우드 서비스의 로드 밸런서를 이용하여 생성되는 서비스입니다
  - External IP를 할당받아 클러스터 외부에서 클러스터 내부 서비스로의 접근이 가능합니다  
  - Internal IP도 할당받아 내부에서 특정 파드 집합으로의 단일 엔드포인트 제공 역시 받을 수 있습니다    
  - 클라이언트는 LoadBalancer IP를 통해 특정 서비스로 외부 트래픽을 포워딩할 수 있게 됩니다   
    ![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/service_load_balancer.png)
2. NodePort
  - Node에 Port를 할당받아놓고, 외부에서 접근할 수 있는 External IP가 아니라 NodePort를 할당받습니다  
  - 할당받은 Node Port를 통해 들어온 트래픽을 파드 집합으로 포워딩합니다  
    ![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/nodeport_diagram.png )
  - NodePort 타입의 경우 <특정 노드의 IP:Port> 로 트래픽을 보내므로 해당 노드가 다운된 상태이면 요청을 정상적으로 처리할 수 없게됩니다  
    따라서 로드밸런서를 이용해 건강한 노드로 트래픽을 전달할 수 있게 만드는 것이 필요합니다  
3. ClusterIP
  - 별 다른 옵션 없이 서비스 생성 시 생성되는 기본 타입입니다   
  - Pod IP처럼 외부에서 접근할 수 없는 IP를 할당받습니다   
  - 이 때 Service IP는 클러스터 내부 통신용으로 사용되며 굳이 외부로의 노출이 필요 없는 애플리케이션의 경우 Cluster IP 타입의 서비스를 사용할 수 있습니다   
    ![](/assets/img/2023/10/2023-10-03-kubernetes_about_service/cluster_ip_diagram.png )





