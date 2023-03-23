---
title:  "Kubernetes 클러스터 실습"
excerpt: "GCP를 이용하여 쿠버네티스 클러스터를 생성해보는 실습을 진행합니다.  "

categories:
  - Kubernetes
tags:
  - [Kubernetes, Devops]

toc: true
toc_sticky: true
 
date: 2023-03-23
last_modified_at: 2023-03-23
---
# 쿠버네티스 클러스터 생성 및 실습 (w. GCP)

## GCP 활용하기
구글 클라우드에서 진행하기 위해 [구글 클라우드](https://cloud.google.com/) 에 접속합니다. 
다음 화면엔서 `$300의 무료 크레딧과 20여 개 제품에 대한 무료 사용량이 제공됩니다.` 라는 문구의 링크로 들어가면 무료로 사용할 수 있는 서비스에 대한 내용을 확인할 수 있습니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/GCP_free_tier_info.png) 

링크 이동 후에 확인해보면 Google Kubernetes Engine 이라는 무료 등급 제품 항목이 있는데, 이 것을 이용해 kubernetes 클러스터를 구축해볼 것입니다.  

위 페이지에서 무료로 시작하기를 눌러 정보를 입력하고 등록을 완료하면 구글 클라우드를 사용할 수 있게 됩니다.  


## 쿠버네티스 클러스터 구성
구글 클라우드에서 쿠버네티스 클러스터를 구성하기 위해 좌측 상단 메뉴 아이콘을 눌러 나온 메뉴바에서 Kubernetes Engine으로 들어가보도록 하겠습니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/kubernetes_engine_start_page.png)
위 페이지가 나오면 사용 버튼을 눌러 쿠버네티스 엔진을 사용하도록 합니다.  

활성화가 되었다면 클러스터 탭에 들어가 다음 화면에서 `만들기`를 선택합니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/kubernetes_start_page_cluster.png)

그 후에 아래 팝업이 뜨는데 여기서 Standard 모드를 선택해 구성 버튼을 누르도록 합니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/cluser_create_popup.png)
Autopilot 모드는 구글에서 클러스터 관리를 도와주는 것을 포함하고 있고, Standard 버전은 좀 더 일반적인 버전입니다.  

구성 버튼을 누른 후에는 다음 화면으로 변경됩니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/cluster_default_info.png)
이 화면에서는 원하는 대로 클러스터 이름을 변경할 수 있습니다.  

또한 좌측 메뉴에 default-pool 메뉴에 들어가보면 아래 화면을 볼 수 있습니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/default-pool.png)
여기에서는 몇 개의 서버를 사용해서 클러스터를 설정할 지 노드의 개수 등을 지정할 수 있습니다.  
저는 디폴트인 3개를 그대로 이용하도록 하겠습니다.  

그리고 화면 하단의 `만들기` 버튼을 누르면 설정 정보 대로 클러스터가 생성됩니다.  

클러스터 생성이 완료되면 다음과 같은 목록이 보입니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/cluster_creation_complete.png)


## 쿠버네티스 클러스터와 통신
로컬 머신에서 쿠버네티스 클러스터에 `kubectl` 명령어를 보내기 위해서는 위에서 생성한 클러스터의 API 서버의 주소를 알아야 합니다.  

따라서 `kubectl` 명령어 실행을 위해 gcp에 생성한 쿠버네티스 서버 접속 정보를 알아보도록 하겠습니다.  
이 때 접속 정보 구성 방법은 `gcloud` 명령어를 이용해 클러스터 접속 정보와 사용자 정보를 로컬에 구성할 수 있습니다.

접속에 대한 정보를 얻기 위해 다음 화면과 같이 원하는 클러스터의 메뉴의 연결 버튼을 클릭합니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/%08get_cluster_connection_info.png)

그러면 아래와 같은 팝업이 뜨게됩니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/cluster_connection_info_popup.png)
이 팝업에는 gcloud를 통해 클라우드 정보를 구성할 수 있는 명령어가 아래처럼 명시되어 있습니다.   
> gcloud container clusters get-credentials [클러스터 이름] --zone [리전] --project [프로젝트]  

이 명령어를 잘 복사해두도록 합니다.  
그 다음으로는 `CLOUD SHELL에서 실행`버튼을 눌러줍니다.  

그러면 아래와 같이 화면 하단에 shell이 생성됩니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/shell_before_activation.png)
계속 버튼을 누르면 프로비저닝이 되는 시간동안 잠시 대기했다가 아래와 같은 쉘 화면이 뜹니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/shell_after_provisioning.png)
참고로 이 쉘은 구글에서 제공하므로 gcloud명령어와 kubectl 명령어가 이미 설치되어 있습니다.  

### kubtctl configuration 파일
클러스터에 대한 접속 정보를 설정하기 전에 정상적으로 명령어가 수행되는 지 확인하기 위해 아래 명령어를 수행해보도록 하겠습니다.  
```sh
$ kubectl get pod
```
그 결과 아래 에러를 만나게 됩니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/kubectl_get_pod_error.png)
서버에 대한 연결이 "refused"되었다는 오류가 납니다.  
이는 연결을 보낼 마스터 서버가 vm 머신 내에 정의되지 않았기 때문에 요청을 보낼 곳이 없어 명령어가 거절당한 것입니다.  
따라서 클러스터에 대한 접속 정보를 세팅해야 한다는 것을 알 수 있습니다.  

클러스터에 대한 접속 정보는 `~/.kube/config`파일을 생성해 명시합니다.  
저 위치에 config 파일을 만드는 이유는 kubectl은 위의 경로에 있는 파일을 참조해 연결 정보를 알게 되기 때문입니다.  
`~/.kube/config` 파일을 생성하기 위해 쉘에 앞에서 복사했던   
> gcloud container clusters get-credentials [클러스터 이름] --zone [리전] --project [프로젝트]   

명령어를 붙여넣기 후 실행합니다.  

실행 후에 `~/.kube/config` 파일을 출력해보면 접속 정보가 생성되어 있음을 확인할 수 있습니다.  

`~/.kube/config`파일의 내용을 간략하게 살펴보자면 아래와 같습니다.  
```yml
apiVersion: v1
clusters:
    - cluster:
        certificate-authority-data: <인증서를 base64로 인코딩한 데이터>
        server: <마스터 노드의 API Server 주소>
  name: <클러스터 이름>
contexts:   # 클러스터와 user의 조합에 대한 내용 (어떤 user가 cluster에 접근할지 등)
    - context:
        cluster: <클러스터 이름>
        user: gke_<project명>_us-central1-c_<클러스터이름>
    name: gke_<project명>_us-central1-c_<클러스터이름>
current-context: gke_<project명>_us-central1-c_<클러스터이름>
kind: Config
preferences: {}
    users:
    - name: gke_<project명>_us-central1-c_<클러스터이름>
    user:
        exec:
        apiVersion: client.authentication.k8s.io/v1beta1
        command: gke-gcloud-auth-plugin
        installHint: ...
        provideClusterInfo: true
        ...
```

설정 정보 구성 후에 다시 아래 명령어를 보내보도록 하겠습니다.  
```sh
$ kubectl get pod
```
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/kubect_get_pod_working.png)
아까와 같은 에러가 사라지고 정상 동작하는 것을 알 수 있습니다.  
아직 아무 것도 하지 않은 상태이므로 default namespace에서 아무 resource도 찾을 수 없는 것이 정상입니다.  


### 로컬 머신에서 클러스터에 접근
google에서 제공하는 cloud shell이 아니라 로컬 머신에서 클러스터에 접속할 수 있도록 설정하는 방법에 대해서 추가적으로 알아보도록 하겠습니다.  

가장 먼저 gcloud를 명령어를 통해 접속 정보를 구성하므로 gcloud를 설치해야합니다.  
[이 웹사이트](https://cloud.google.com/sdk/docs/install?hl=ko#installation_instructions)에 들어가시면 os별로 gcloud sdk를 설치하는 방법에 대해 잘 설명되어 있으니 내용을 따라 진행하면 문제없이 잘 설치될 것입니다.  

그 후에는 로컬머신에 kubectl 패키지를 설치해야합니다.  
kubectl 패키지는 [이 웹페이지](https://kubernetes.io/ko/docs/tasks/tools/#kubectl)를 통해 설치할 수 있습니다.  
해당 페이지에도 가이드가 상세하게 적혀있으니, 가이드를 따라 쉽게 설치할 수 있을 것입니다.  

두 가지 cli가 정상적으로 설치되었다면 위의 클러스터 구성 정보를 생성하는 `gcloud` 명령어를 로컬 터미널에 입력하여 수행시키면 됩니다.  

결과로 ~/.kube/config 파일이 생성되긴 했으나 아래 경고가 발생합니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/gcloud_config_creation_warning.png)
경고에서 알려주는 대로 [이 웹페이지](https://cloud.google.com/blog/products/containers-kubernetes/kubectl-auth-changes-in-gke?hl=en)에 접속하여 `gke-gcloud-auth-plugin`를 추가적으로 설치해줍니다.  

위 단계를 모두 수행한 후에 
```sh
$ kubectl get pod
```
명령어를 수행하면 앞에서와 동일하게 `No resources found in default namespace.`라는 정상적인 결과를 얻을 수 있습니다.  


## 쿠버네티스 리소스 생성을 위한 YAML 작성
먼저 kubernetes에 대한 설정 파일은 yaml 파일을 사용하기 때문에 해당 파일을 더 손쉽게 편집하기 위해 visual studio code의 kubernetes extension을 설치하도록 하겠습니다.  
vscode의 왼쪽 메뉴바의 marketplace에 들어가 kubernetes를 검색하면 나오는 아래 extension을 설치하면 됩니다.  
![](/assets/img/2023/02/2023-02-23-kubernetes-cluster-setting/kubernetes_extension.png)

이 extension을 설치하면 yaml파일 작성 시 작성할 속성이나 스펙 등이 자동완성되는 기능을 이용할 수 있습니다.  
