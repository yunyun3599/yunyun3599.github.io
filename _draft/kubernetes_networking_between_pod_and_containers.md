# Pod과 컨테이너들간의 통신
이번 포스트에서는 Pod이 컨테이너 혹은 다른 Pod과 어떻게 통신하는 지 알아보도록 하겠습니다.  

이를 위해 아래 두 가지 사항을 확인해보도록 하겠습니다.  
1. Pod 안에 서로 다른 컨테이너끼리 localhost로 통신
  - 하나의 Pod에 서로 다른 포트로 컨테이너 2개를 선언
2. 서로 다른 Pod끼리 Pod IP로 통신
  - Pod A에 있는 컨테이너 -> Pod B에 있는 컨테이너로 요청 전송 / 응답 확인  


## 컨테이너 통신을 확인하기 위한 과정
1. 총 2개의 Pod 선언 (컨테이너 2개인 Pod과 1개인 Pod를 각각 선언), 환경변수 설정
2. Pod 생성, 배포
3. Pod IP 할당 및 컨테이너 실행 확인
4. 컨테이너 환경변수 목록 확인
5. 같은 Pod 내에 있는 컨테이너 간 localhost 통신
6. 다른 Pod의 Pod IP로 통신
7. 포트포워딩을 통해 각 컨테이너로 요청 / 응답 확인

생성할 3개의 컨테이너는 엔드포인트를 갖게 되는데, `/hello`라는 엔드포인트를 공통으로 갖고, 그 외에 각각 하나씩의 엔드포인트를 더 갖게 됩니다.  
1번 컨테이너는 `/sky`, 2번 컨테이너는 `/tree`, 3번 컨테이너는 `/rose`라는 엔드포인트를 갖도록 하겠습니다.  

그리고 각각의 엔드포인트를 제공하는 애플리케이션 코드는 Node로 작성되었으며, 대표적으로 `/sky`와 `/hello` 엔드포인트를 갖는 컨테이너의 애플리케이션 코드는 다음과 같습니다.   
1. `/sky` 엔드포인트
  ```js
  const POD_IP = process.env.POD_IP
  const NODE_NAME = process.env.NODE_NAME
  const NAMESPACE = process.env.NAMESPACE

  app.get('/sky', (req, res) => {
    res.render('sky', {podIP: POD_IP, nodeName: NODE_NAME, namespace: NAMESPACE})
  })

  app.get('/hello', (req, res) => {
    res.json("Hello, This container is Sky container")
  })

  app.listen(PORT, () => {
    console.log(`Server is running on ${PORT}`)
  })
  ```

## pod 및 컨테이너 설정
### /sky 엔드포인트를 갖는 blue-app 설정
`/sky`라는 엔드포인트를 갖는 컨테이너를 blue-app이라는 이름으로 띄우기 위해 아래와 같은 yaml 파일을 작성하도록 하겠습니다.  
이 컨테이너는 blue-green-app 이라는 pod에서 8080 포트를 사용하게 됩니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-green-app
spec:
  containers:
  - name: blue-app
    image: yoonjeong/blue-app:1.0
    ports:
      - containerPort: 8080
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
```

### /tree 엔드포인트를 갖는 green-app 설정
다음으로는 `/tree`라는 엔드포인트를 갖는 컨테이너를 green-app이라는 이름으로 띄우기 위해 아래와 같은 yaml 파일을 작성하도록 하겠습니다.  
이 컨테이너는 blue-app과 함께 blue-green-app 이라는 pod 내에 생성되며 8081 포트를 사용하게 됩니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-green-app
spec:
  containers:
  - name: green-app
    image: yoonjeong/green-app:1.0
    ports:
      - containerPort: 8081
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
```

blue-app과 green-app은 같은 pod 내에 생성되므로 blue-green-app.yaml이라는 하나의 파일에 아래와 같이 작성하도록 하겠습니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blue-green-app
spec:
  containers:
  - name: blue-app
    image: yoonjeong/blue-app:1.0
    ports:
      - containerPort: 8080
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
  - name: green-app
    image: yoonjeong/green-app:1.0
    ports:
      - containerPort: 8081
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
```

### /rose 엔드포인트를 갖는 red-app 설정
다음으로는 `/rose`라는 엔드포인트를 갖는 컨테이너를 blue-app이라는 이름으로 띄우기 위해 아래와 같은 yaml 파일을 작성하도록 하겠습니다.  
이 컨테이너는 red-app 이라는 pod 내에 생성되며 8080 포트를 사용하게 됩니다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-app
spec:
  containers:
  - name: red-app
    image: yoonjeong/red-app:1.0
    ports:
      - containerPort: 8080
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
```

### 컨테이너별 환경변수 설정  
각 컨테이너별로 사용할 환경변수를 설정해주기 위해 아래와 같이 yaml 파일을 설정해주도록 하겠습니다.  
앞에서 작성한 yaml 파일의 아래에 각각 밑의 내용을 추가해주도록 하겠습니다.  
```yml
spec:
  containers:
  - env:
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
```

## pod/컨테이너간의 통신 확인
실습 진행 이전에 아래 두가지 명령어를 통해 kubernetes 설정이 잘 되어있는 지 확인해보도록 하겠습니다.  
```sh
$ kubectl config current-context
$ kubectl get pod
```
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubernetes_working_confirmation.png)

### Pod 생성 및 컨테이너 실행
앞에서 작성한 yaml 파일을 통해 pod를 띄우고 컨테이너를 생성하도록 하겠습니다.  
pod를 띄우기 위해서는 아래 명령어를 수행합니다.  
```sh
$ kubectl apply -f blue-green-app.yaml
$ kubectl apply -f red-app.yaml
```
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubectl_apply_-f_blue-green-app.yml.png)

실행된 컨테이너가 정상적으로 동작하는 지 등을 확인하기 위해 로그를 보고 싶다면 아래 명령어를 이용할 수 있습니다.  
```sh
# kubectl logs <pod명> -c <컨테이너명>
$ kubectl logs blue-green-app -c blue-app
```
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubectl_logs.png)

컨테이너 내에 환경변수가 잘 세팅되었는지 확인하기 위해서는 아래 명령어를 이용할 수 있습니다.  
```sh
$ kubectl exec blue-green-app -c blue-app -- printenv POD_IP NAMESPACE NODE_NAME
```
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubectl_exec_printenv.png)

### 같은 pod의 컨테이너 간의 통신 확인  
가장 먼저 같은 pod내에 배포되어있는 blue-app과 green-app 간의 통신에 대해 알아보도록 하겠습니다.  
blue-app에서 curl 명령어를 통해 green-app에 접근해보도록 하겠습니다.  
```sh
$ kubectl exec blue-green-app -c blue-app -- curl -vs localhost:8081/tree
```
아래와 같이 요청에 대한 응답이 잘 오는 것을 볼 수 있습니다.  
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubectl_exec_curl_blue_green.png)

### 다른 pod의 컨테이너 간의 통신 확인
다음으로는 red-app 컨테이너에서 blue-app 컨테이너로 접근해보며 다른 pod에 위치한 컨테이너간의 통신에 대해 알아보도록 하겠습니다.  
다른 pod에 위치한 컨테이너에 접근하기 위해서는 ip 주소를 알아야 하므로 아래 명령어를 통해 `BLUE_POD_IP` 라는 환경변수에 blue-green-app pod의 ip 주소를 저장해두도록 하겠습니다.  
```sh
$ export BLUE_POD_IP=$(kubectl get pod blue-green-app -o jsonpath="{.status.podIP}")
```
그리고 난 후에 위에서 구한 IP 주소를 이용해 red-app에서 blue-app 에 접근해보도록 하겠습니다.  
```sh
$ kubectl exec red-app -c red-app -- curl -vs $BLUE_POD_IP:8080/hello
```
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/kubectl_exec_from_red-app_to_blue-app.png)

### 포트포워딩을 통해 웹 브라우저에서 각 컨테이너 접근  
다음으로는 pod 내의 컨테이너들을 포트포워딩하여 로컬의 웹 브라우저에서 각 컨테이너에 접근할 수 있도록 설정해보도록 하겠습니다.  
포트포워딩을 위한 명령어는 아래와 같습니다.  
```sh
$ kubectl port-forward blue-green-app 8080:8080
$ kubectl port-forward blue-green-app 8081:8081
$ kubectl port-forward red-app 8082:8080
```
포트포워딩을 한 후에 해당 포트의 제공되는 url로 접근해보면 아래와 같은 화면을 볼 수 있습니다.  
![](/assets/img/2023/04/2023-04-12-kubernetes_pod_networking/blue-app_sky_endpoint_web_browser.png)
