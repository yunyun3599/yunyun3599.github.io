---
title:  "Kubectl 명령어"
excerpt: "kubectl의 기본 명령어에 대해 정리해봅니다."

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-05-05
last_modified_at: 2023-05-05
---
# Kubectl 명령어 

## 쿠버네티스 오브젝트 기본 정보 조회 명령어
- `kubectl api-resources`
    - 쿠버네티스 클러스터에서 사용할 수 있는 오브젝트 목록을 조회하는 명령어입니다.  
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_api-resources.png)
    - 각 필드들은 아래 사항들을 의미합니다.  
        - SHORTNAMES: 간단하게 표기할 수 있는 이름
        - APIVERSION: 쿠버네티스 API 인터페이스 버전
        - NAMESPACE: 오브젝트가 네임스페이스 별로 구분이 가능한 지 여부   
          (namespace = 클러스터의 리소스들을 특정 단위로 구분하기 위한 논리적인 그루핑)
- `kubectl explain <type>`
    - 쿠버네티스 오브젝트 설명과 1 레벨 속성들의 설명을 조회하는 명령어입니다.
    - apiVersion, kind, metadata, spec, status
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_explain_pod.png)
    - 명령 수행 결과 필드를 확인하면 오브젝트 생성 시 어떤 것을 작성할 지를 알 수 있습니다.
        - pod의 경우 apiVersion, kind, metadata, spec, status 등의 속성을 갖는다는 것을 알 수 있습니다. (참고 - status는 쿠버네티스에서 작성해주는 속성)
- `kubectl explain <type>.<fieldName>[.<fieldName>]`
    - 사용 예) kubectl explain pods.spec.containers
    - 쿠버네티스 오브젝트 속성들의 구체적인 설명을 확인하는 명령어입니다. (Json 경로를 explain 뒤에 붙여 원하는 필드만 확인 가능)
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_explain_pod_spec_containers.png)


## 오브젝트 실행 관련 명령어
- `kubectl get nodes`
    - 쿠버네티스 클러스터에 속한 노드 목록을 조회하는 명령어입니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_get_nodes.png)
- `kubectl apply -f <object-file-name>`
    - 사용 예) kubectl apply -f deployment.yaml
    - 쿠버네티스 오브젝트 생성/변경하는 명령어입니다.  
    - 명령어 수행 전 미리 deployment.yaml을 아래와 같은 내용으로 작성하고 실행합니다.
        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
        name: nginx-deploy
        spec:
        selector:
            matchLabels:
            app: nginx
        replicas: 2
        template:
            metadata:
            labels:
                app: nginx
            spec:
            containers:
            - name: nginx
                image: nginx:1.14.2
                ports:
                - containerPort: 80
                resources:
                limits:
                    memory: "64Mi"
                    cpu: "50m"
        ```
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_apply_-f_deployment.yaml.png)  
- `kubectl get pods`
     - 실행 중인 Pod(컨테이너) 목록을 조회하는 명령어입니다.
     ![](assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_get_pods.png)
    

## 배포된 오브젝트 관리 명령어
- `kubectl scale -f <object-file-name> --replicas=N`
    - 사용 예) kubectl scale -f deployment.yaml --replicas=3
    - 애플리케이션 배포 개수를 조정하는 명령어입니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_scale_-f_deployment.yaml_--replicas=3.png)
- `kubectl diff -f <object-file-name>`
    - 사용 예) kubectl diff -f deployment.yaml
    - 현재 실행 중인 오브젝트 설정과 입력한 파일의 차이점 분석할 때 사용하는 명령어입니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_diff_-f_deployment.yaml.png)
- `kubectl edit <type>/<name>`
    - 사용 예) kubectl edit deployment/nginx-deployment
    - text editor를 실행시켜 쿠버네티스 오브젝트의 spec을 편집할 수 있도록 해주는 명령어입니다.
    - `kubectl get deployment`로 deployment 목록을 조회해서 어떤 deployment를 수정할 지 name 확인이 가능합니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_edit_deployment_nginx-deploy.png)

## 동작 확인 및 모니터링을 위한 명령어
- `kubectl port-forward <type>/<name> <local-port>:<container-port>`
    - 사용 예) kubectl port-forward pod/nginx-deployment-abcdefg-hijk 8080:80
    - 파드에서 실행중인 컨테이너 포트로 로컬 포트를 포워딩할 때 사용할 수 있는 명령어입니다.
    - `kubectl get pod` 명령어를 통해 pod 목록 조회가 가능합니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_port-forward.png)
    - 포트포워드 성공 후 포트포워딩한 포트로 curl 명령어를 수행해보면 정상적인 응답이 오는 것을 확인할 수 있습니다.  
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_port-forward_curl_result.png)
- `kubectl attach <type>/<name> -c <container-name>`
    - 사용 예) kubectl attach deployment/nginx-deployment -c nginx
    - 현재 실행중인 컨테이너 프로세스에 접속하여 로그를 확인하기 위한 명령어입니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_attach.png)
- `kubectl logs <type>/<name> -c <container-name>`
    - 사용 예) kubectl logs deployment/nginx-deployment -c nginx
    - 현재 실행중인 컨테이너 프로세스의 모든 로그를 출력하는 명령어입니다.
    ![](/assets/img/2023/02/2023-02-25-kubernetes-kubectl_commands/kubectl_logs_deployment.png)
