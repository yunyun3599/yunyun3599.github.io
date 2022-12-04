# 도커 이미지 저장소

## 도커 허브
도커에서 공식적으로 지원하는 이미지 저장소입니다.  
도커허브를 사용하기 위해서는 [도커 허브](https://hub.docker.com/) 홈페이지에 가서 회원가입을 먼저 진행해야합니다.  

도커 허브에서 계정을 만들었으면 로컬 터미널을 이용해서 도커허브에 로그인을 할 수 있습니다.  
```sh
$ docker login
```
![](/assets/img/2022-12-04-docker_remote_repository/docker_login.png)
앞서 회원가입할 떄 사용한 usernamer과 password를 입력해서 로그인을 진행하면 됩니다.  

로컬에 있는 이미지를 도커허브에 올려보도록 하겠습니다.  
새로운 이미지를 도커허브에 push 하기 위해서 아래와 같은 명령어를 쓸 수 있습니다.  
```sh
$ docker tag local-image:tagname new-repo:tagname
```
예시로 nginx:latest 이미지를 tag 명령어를 이용해 도커허브에 올릴 이미지로 만들어보겠습니다.  
```sh
$ docker tag nginx:latest yoonjae/my-nginx:v1.0.0
```
해당 이미지를 도커 허브에 push하려면 아래 명령어를 수행하면 됩니다.  
```sh
$ docker push yoonjae/my-nginx:v1.0.0
```
![](/assets/img/2022-12-04-docker_remote_repository/docke_hub_push.png)`

push 된 이미지를 도커허브에서 확인 가능합니다.  
![](/assets/img/2022-12-04-docker_remote_repository/docker_pushed_repository.png)

<br>

## AWS ECR
AWS 홈페이지에서 Elastic Container Registry로 검색하면 AWS ECR 페이지로 접근할 수 있습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/aws_ecr.png)

왼쪽 메뉴 바를 열어 Repositories 메뉴에 들어가면 아래와 같은 화면을 볼 수 있습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/aws_ecr_repository_page.png)


리포지토리를 생성하기 전에 ECR에 대한 과금 정책을 확인해보도록 하겠습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_billing.png)

|리포지토리 종류|스토리지|인터넷 전송|
|---|---|---|
|프라이빗 리포지토리|월 500MB|수신 무료<br>송신 9.999TB까지 GB당 0.09USD|
|퍼블릭 리포지토리|월 50GB|익명 전송: 500GB까지 무료<br>AWS 계정으로 전송: 5TB까지 무료|
추가적인 요금 정책은 [이 페이지](https://aws.amazon.com/ko/ecr/pricing/)에서 확인하세요.

리포지토리 페이지에서 리포지토리 생성 버튼을 눌러 리포지토리를 만들어보겠습니다.  
리포지토리 이름은 my-nginx로 지어 진행했습니다.  
이름을 설정하는 부분 이외에는 모두 비활성화를 해두고 리포지토리를 생성하였습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_create_repository.png)

생성된 리포지토리는 아래와 같이 확인 가능합니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_repository_list.png)

리포지토리 이름을 클릭하여 아래 페이지에 접근한 후 푸시 명령 보기를 클릭해줍니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_repository_detail.png)

![](/assets/img/2022-12-04-docker_remote_repository/ecr_show_push_commands.png)
도커허브와 동일하게 docker tag명령어와 docker push 명령어를 통해 이미지를 push 합니다.  
단, 이 때 ECR 사용자 인증을 진행해야 합니다.  

위의 과정을 진행하려면 aws cli를 사용해야 하므로 aws cli 설치를 진행하도록 하겠습니다.  
aws cli는 다음 명령어를 통해 설치합니다.  
```sh
$ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
$ sudo installer -pkg AWSCLIV2.pkg -target /
```

aws에 접근하기 위해서는 credential 설정이 필요합니다.  
IAM 사용자에 대한 액세스 키를 생성하기 위해 [해당 페이지](https://console.aws.amazon.com/iam/)로 이동합니다.  
![](/assets/img/2022-12-04-docker_remote_repository/iam_dashboard.png)

위에서 사용자 메뉴로 접근합니다.  
만약 대시보드에 이미 생성된 사용자가 있다면 해당 사용자를 사용해도 되고, 없다면 우측 상단의 사용자 추가 버튼을 눌러 새로운 사용자를 생성해서 사용하게 됩니다.  
![](/assets/img/2022-12-04-docker_remote_repository/iam_user.png)

사용할 이름을 추가하고 액세스 유형으로 액세스 키를 선택했습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/user_add_1.png)

다음 단계에서는 권한에서 기존 정책 직접 연결을 선택하고 가장 위의 AdministratorAccess를 선택해주겠습니다.  
AdministratorAccess는 full access권한을 부여해주는 정책입니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_administrator_access.png)
 
다음 단계인 태그 단계와 검토 단계는 별다른 설정 없이 넘어가고 사용자 추가를 완료하도록 하겠습니다.   
![](/assets/img/2022-12-04-docker_remote_repository/user_add_finish.png)
생성된 사용자에 대해 액세스 키 ID와 비밀 액세스 키가 설정됩니다.  
.csv 다운로드 버튼을 통해 생성된 키페어를 안전한 장소에 저장합니다.  

그리고 다시 터미널로 돌아와 아래 명령어를 통해 aws 접근 권한을 설정해줍니다.  
```sh
$ aws configure import --csva file://credentials.csv
```
여기서 뒷 부분에는 본인이 csv 파일을 저장한 경로를 적어주면 됩니다.  
이 때 앞에 file:// 부분을 꼭 적어주셔야합니다.

완료가 되면 아래 명령어를 통해 잘 설정되었는지 확인해봅니다.  
```sh
$ aws sts get-caller-identity
```
만약 잘 설정되지 않았다면 AWS_PROFILE 환경변수를 설정해줍니다.  
```sh
$ export AWS_PROFILE=[user name]
```
여기서 user name은 사용자 추가 시에 만들었던 사용자의 이름입니다.  

설정이 다 되었으면 ECR로 돌아와 레지스트리에 대한 인증을 진행합니다.   
위에서 푸시 명령 보기 버튼을 눌러 팝업된 회면의 명령어 목록 중 가장 위의 명령을 수행하시면 됩니다.  
```sh
$ aws ecr get-login-password --region [사용할 리전] | docker login --username AWS --password-stdin [[aws 계정 id].dkr.ecr.[사용 리전].amazonaws.com]
```

그 후에는 tag 명령어를 통해 push할 이미지를 준비합니다.  
```sh
$ docker tag nginx:latest [aws 계정 id].dkr.ecr.[사용 리전].amazonaws.com/my-nginx:v1.0.0
```

태그가 완료되었다면 Push를 진행합니다.  
```sh
docker push [aws 계정 id].dkr.ecr.[사용 리전].amazonaws.com/my-nginx:v1.0.0
```

푸쉬가 완료되면 aws 페이지에서 업로드된 이미지를 확인할 수 있습니다.  
![](/assets/img/2022-12-04-docker_remote_repository/ecr_pushed_image.png)