# 도커 볼륨

## 도커 레이어 아키텍처
- 컨테이너 레이어
    - Read and Write
    - 새로운 변경 사항이 있을 때 반영됨
    - 컨테이너가 종료되면 삭제됨 (임시 데이터 저장소)
- 이미지 레이어
    - 명세서 기반으로 빌드
    - Read-Only (변경사항 반영 X)
    - Ubuntu image 예시
        5. Update Entrypoint
        4. Source code
        3. Install in pip packages
        2. Changes in apt packages
        1. Base Ubuntu Layer

레이어 구조의 장점
- 추후 이미지에 변경사항이 있을 때 변경사항이 포함된 레이어 위쪽의 레이어들만 변경됨  
- 변경사항이 레이어들만 바뀌고 공통 부분은 함께 사용할 수 있으므로 저장공간을 많이 차지하지 않음  


## 호스트 볼룸
호스트의 디렉토리를 컨테이너의 특정 경로에 마운트  
```sh
$ docker run -v [마운트할 로컬 경로]:[/마운트할 컨테이너 경로] [이미지]
```
예시
로컬의 ~/study/docker-study 경로에 print.sh라는 파일을 아래와 같이 생성
```sh
$ vi ~/study/docker-study/volume/print.sh
```
내용은 다음과 같이 작성
```
#!/bin/bash

echo hello!!
```
해당 경로를 마운트하여 도커 실행
```sh
$ docker run -ti --name test -v ~/study/docker-study/volume:/data ubuntu bash
```
도커 컨테이너 내에서 아래 명령어 수행
```sh
$ /data/print.sh
```
앞에서 작성한 hello!!라는 문장이 프린트 되는 것을 확인할 수 있음  
![](/assets/img/2022-11-30-docker_volume/hostmount.png)

## 볼륨 컨테이너
특정 컨테이너의 볼륨 마운트를 공유   
```sh
$ docker run --volumes-from [volume 컨테이너] [이미지]
```
앞에서 호스트 볼륨과 마운트한 것과 동일하게 생성한 컨테이너를 하나 만듭니다.  
```sh
$ docker run -d -ti --name test --rm -v ~/study/docker-study/volume:/data ubuntu bash
```
그리고 위의 컨테이너를 볼륨으로 사용하는 새로운 컨테이너를 생성해보도록 하겠습니다.  
```sh
$ docker run -ti --volumes-from test ubuntu bash
```
컨테이너 내에서 /data 경로로 이동하면 마운트할 때 사용한 컨테이너의 /data 경로에 위치해있던 print.sh 파일이 새로 생성한 컨테이너 내에도 있음을 확인할 수 있습니다.   
![](/assets/img/2022-11-30-docker_volume/volumes-from.png)

## 도커 볼륨
도커가 제공하는 볼륨 관리 기능을 활용하여 데이터를 보존하는 방식입니다.  
데이터가 저장되는 기본 경로는 `/var/lib/docker/volumes/${volume-name}/_data` 입니다.  
> 참고로 Mac은 docker를 vm 위에서 실행하기 때문에 /var/lib/docker 경로가 존재하지 않는다고 합니다.  

도커 볼륨을 생성하는 명령어는 아래와 같습니다.  
```sh
$ docker volume create --name [볼륨 이름]
```
해당 볼륨을 이용하여 컨테이너에 마운트하는 방법은 아래와 같습니다.  
```sh
$ docker run -v [볼륨이름]:[마운트 경로] [이미지]
```

db라는 이름의 도커 볼륨을 생성한 후 mysql이미지를 이용해 생성한 컨테이너에 볼륨을 마운트해보도록 하겠습니다.  
```sh
$ docker volume create --name db
```
생성된 도커 볼륨에 대한 정보를 `docker volume inspect db` 명령어를 통해 확인해보았습니다.  
![](/assets/img/2022-11-30-docker_volume/docker_volume_inspect.png)
위에서 생성한 볼륨을 가지고 mysql 이미지를 run 한 컨테이너를 생성하도록 하겠습니다.  
```sh
$ docker run \
 -d \
 --name mysql \
 -e MYSQL_DATABASE=volumepractice \
 -e MYSQL_ROOT_PASSWORD=volumepractice \
 -v db:/var/lib/mysql \
 -p 3306:3306 \
 mysql
```
여기에서 `MYSQL_DATABASE`는 최초로 생성할 mysql database에 대한 환경변수이고, `MYSQL_ROOT_PASSWORD`는 mysql root계정의 password를 설정하는 환경변수로 저 두가지 변수가 세팅되어야 제대로 mysql 컨테이너가 실행됩니다.  


## 읽기 전용 볼륨 연결
읽기전용으로 볼륨을 마운트하려면 아래와 같은 명령어를 통해 컨테이너를 실행시키면 됩니다.  
```sh
$ docker run -v [볼륨이름]:[마운트경로]:ro [이미지명]
```
여기에서 ro는 read-only라는 뜻으로 이렇게 주로 변경이 되어서는 안되는 경로를 마운트 할 때 사용합니다.  

읽기전용으로 볼륨을 연결해보도록 하겠습니다.  
```sh
docker run -d -ti --name test --rm -v ~/study/docker-study/volume:/data:ro ubuntu bash
```
후에 마운트된 경로로 가서 새로운 파일을 만들려고 시도해보면 아래와 같은 결과가 나옵니다.  
![](/assets/img/2022-11-30-docker_volume/mount_read_only.png)
읽기 전용으로 마운트된 경로이기 때문에 새로운 파일 생성이나 수정이 불가능한 것을 확인할 수 있습니다.  