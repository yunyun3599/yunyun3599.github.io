# 쿠버네티스 Replica Set

## ReplicaSet이란
ReplicaSet은 Pod 복제본을 생성하고 관리하는 역할을 하는 오브젝트입니다.  
따라서 ReplicaSet을 사용하면 여러 개의 Pod을 관리할 수 있습니다.  

ReplicaSet을 통해 여러개의 Pod를 띄우기 위해서는 ReplicaSet 오브젝트를 정의하고 원하는 Pod의 개수를 replicas 속성으로 선언하면 됩니다.  
ReplicaSet을 통해 생성된 Pod들은 선언된 개수를 유지하도록 쿠버네티스에서 관리됩니다.   

## ReplicaSet 필요성
### Pod에 문제가 생겼을 때  
Pod에 문제가 생겨 Pod가 종료되고 클라이언트의 요청을 처리할 수 없는 상황이 발생했다고 합시다.  
이런 경우 자동으로 Pod의 상태를 확인하고 구동시킬 Pod 개수를 유지시켜주는 기능을 제공받지 못한다면 클러스터 관리자가 직접 매 순간마다 Pod 상태를 감시하고 복구해야 합니다.  
ReplicaSet을 이용하면 Pod에 문제가 생겨 특정 Pod가 다운되더라도, 다시 기존에 선언해둔 개수를 맞추기 위해 다른 Pod를 자동으로 생성해주기 때문에 위와 같은 수고를 덜 수 있습니다.   

### 소프트웨어의 내결함성 (fault tolerance)
내결함성이란 소프트웨어나 하드웨어 실패가 발생하더라도 소프트웨어가 정상적인 기능을 수행할 수 있어야한다는 의미입니다.  
내결함성이 없는 소프트웨어는 고객의 요구사항을 만족시킬 수 없다는 문제점을 갖습니다.   
ReplicaSet을 이용하면 사람의 개입 없이도 문제 상황에서 자동으로 Pod를 관리하여 서비스의 정상 동작을 보장함으로써 내결함성 있는 소프트웨어 구축을 도와줍니다.  

### Pod 수 조정의 자동화  
결과적으로 ReplicaSet을 사용하면 원하는 개수만큼의 Pod 실행을 보장받고, 이를 위해 Pod의 복제 및 복구 작업을 자동화할 수 있다는 장점이 있습니다.  
클러스터 관리자는 ReplicaSet을 만들어 필요한 Pod의 개수를 쿠버네티스에게 선업합니다.  
그러면 쿠버네티스는 ReplicaSet 요청서에 선언된 replicas 항목을 읽고 그 수만큼의 Pod 실행을 보장해주게 되는 것입니다.  

ReplicaSet으로 Pod 개수를 관리하고자 할 때는 아래 항목들을 명시해주어야 합니다.  
- Replicas: 유지하고 싶은 Pod의 개수
- Pod Selector: ReplicaSet이 어떤 Pod를 관리해야하는 지에 대한 pod label에 대한 값 명시
- Pod Template: 따로 Pod yaml을 작성할 필요 없이 Pod Template에 Pod 생성 관련 정보를 표시할 때 사용

## ReplicaSet 오브젝트 표현 방법
ReplicaSet 오브젝트를 생성할 때는 아래 항목들을 정의해야합니다.  
```yaml
apiVersion: apps/v1     # Kubernetes API 버전

kind: ReplicaSet        # 오브젝트 타입

metadata:               # 오브젝트 식별을 위한 정보
  name: sample-app-rs   # 오브젝트 이름 (ReplicaSet의 이름)
  labels:               # 오브젝트 집합을 구할 때 사용할 이름표
    app: sample-app

spec:                   # 사용자가 원하는 Pod의 상태s
  selector:             # ReplicaSet이 관리하고자 하는 Pod를 선택하기 위한 label query
  replicas:             # 원하는 Pod 복제본 개수
  template:             # Pod 실행 정보 - Pod Template과 동일 (Pod 배포를 위해 작성했던 yaml 파일과 동일한 내용 작성) 
```

`spec` 부분을 더 자세히 살펴보도록 하겠습니다.  
```yaml
spec:
  selector:
    matchLabels:            # 하위 항목에 Pod 레이블의 key: value 값을 정의해 해당 레이블을 갖는 pod들을 배포함
      app: sample-app       # match되는 label이 뭘지 Pod label query 작성
    replicas: 3             # 생성할 Pod의 개수 (선언하지 않으면 기본값은 1)
    template:               # ReplicaSet을 통해 생성하고자하는 Pod 오브젝트 정보를 선언
      metadata:
        labels:
          app: sample-app   # ReplicaSet selector에 정의한 label을 포함해야함
        spec:
          containers:       # pod로 생성할 컨테이너 정보 작성
          - name: sample-app
            image: sample-app:1.0
```
