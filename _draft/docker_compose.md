# 도커 컴포즈

## 도커 컴포즈란 
도커 컴포즈는 단일 서버에서 여러 컨테이너를 프로젝트 단위로 묶어서 관리하기 위해 사용합니다.  
docker-compose.yml 이라는 YAML 형식의 파일을 통해 컨테이너들을 명시적으로 관리할 수 있습니다.  

도커 컨테이너를 통해 프로젝트 단위로 도커 컨테이너를 관리하면 아래와 같은 이점이 있습니다.  
- 도커 네트워크와 볼륨도 프로젝트 단위로 관리 가능  
- 프로젝트 내 서비스간 의존성 관리 가능  
    - e.g. 서비스 1은 서비스 2에 의존성이 있어 서비스 2가 다 뜬 후에 뜨게하기 등
- 도커 컴포즈 내에서는 서비스명을 이용해 네트워크 상에서 호출 가능 (서비스 디스커버리 자동화)  
- 컨테이너의 수평 확장 쉬움 (동일한 역할을 수행하는 같은 서비스에 대한 컨테이너 여러개 생성 가능)


## 도커 컴포즈 주요 개념
### 프로젝트
프로젝트는 도커 컴포즈에서 다루는 워크스페이스의 단위입니다.  
즉 함께 관리하는 서비스 컨테이너의 묶음이라고 볼 수 있습니다.  
프로젝트 단위로 기본 도커 네트워크가 생성됩니다.  

### 서비스
컨테이너를 관리하기 위한 단위힙니다.  
서비스의 scale 값에 따라 컨테이너를 여러개 띄울 수도 있습니다. (서비스 컨테이너 수 확장 가능)

### 컨테이너
서비스를 통해 관리됩니다.  
실질적으로 띄우게 되는 서버입니다.  

## docker-compose.yml
docker-compose.yml 파일에는 최상위 옵션 4가지가 존재합니다.  
- version
    - `version: "3"` 식으로 작성
    - 도커 엔진 및 도커 컴포즈 버전에 따라 호환되는 버전으로 사용 필요   
    (호환되는 버전 정보는 [이 페이지](https://docs.docker.com/compose/compose-file/compose-file-v3/)에서 확인 가능)
    - 버전 3부터 Docker Swarm과 호환 가능
        - Docker Swarm은 컨테이너 오케스트레이션 시스템
- services
    - 프로젝트 내에 사용되는 여러 서비스들을 서브키를 통해서 관리 가능
    - 내부 설정 가능 옵션
        - `image`: 사용할 이미지 지정 가능
        - `volumes`: 이용해 마운트할 볼륨을 배열 형태로 지정 가능
        - `restarts`: 컨테이너 재시작 옵션으로 always라면 오류로 인해 컨테이너가 종료되면 항상 재시작함.  
        - `environment`: 환경 변수 지정
            1. 배열형식으로 지정 
                ```yaml
                environment:
                    - KEY1=VALUE1
                    - KEY2=VALUE2
                ```
            1. 오브젝트형식으로 지정 
                ```yaml
                environment:
                    KEY1: VALUE1
                    KEY2: VALUE2
                ```
        - `networks`: 연결할 네트워크 지정 (여러 네트워크에 연결도 가능)
        - `depends_on`: 컨테이너 실행 순서 지정 가능 (해당 섹션에 지정된 컨테이너가 실행된 이후에 실행되도록 함)
        - `ports`: port publish 진행 가능 (docker의 -p 옵션과 동일)
    - 예시 형식 
    ```yaml
    services:
        web: 
            images: mysql:latest
            volumes:
            - db:/var/lib/mysql
            retarts: always
        database:
            ... 
    ```
- networks 
    - 프로젝트마다 독립적으로 구성
    - 정의를 하지 않는다면 default라는 이름으로 프로젝트 내에 기본 브릿지 네트워크가 생성되어 사용됨
    -  `networks: ... ` 형식
    - 네트워크를 아래와 같은 형식으로 정의하면 기본 bridge 네트워크로 network_name이라는 이름의 네트워크가 생성됨
        ```yaml
        networks:
            network_name: {}
        ```
- volumes
    - 프로젝트마다 독립적으로 구성
    -  `volumes: ... ` 형식
    - volume을 아래와 같은 형식으로 정의하면 기본 volume으로 db라는 이름의 볼륨이 생성됨
        ```yaml
        volumes:
            db: {}
        ```

## docker-compose 기본 명령어
### 예제 프로젝트 생성
docker compose 명령어를 알아보기에 앞서 docker-compose로 구성되는 간단한 프로젝트를 하나 생성해보도록 하겠습니다.  
이 프로젝트에는 redis와 간단한 flask 앱이 포함되어 있습니다.  
작업할 디렉터리 하에 build 디렉터리를 만들고 app.py, docker-compose.yml, Dockerfile, requirements.txt 총 4가지 파일을 생성합니다.  
![](/assets/img/2022-12-05-docker_compose/docker_compose_file_list.png)

각 파일을 아래와 같이 작성해줍니다.  
```python
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return f'Hello! I have been seen {count} times.\n'
```

docker-compose.yml
- 서비스로는 web, redis 2가지가 있으며 web은 밑에서 작성할 Dockerfile을 통해 사용할 이미지를 직접 빌드합니다.  
```yaml
version: "3.9"
services:
  web:
      build: .
      ports:
      - "5000"
  redis:
    image: "redis:alpine"
```

Dockerfile
```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

requirements.txt
```
flask
redis
```

### 프로젝트 시작 관련 명령어
작성한 상태에서 docker-compose 기반으로 서비스를 올리는 명령어인 docker-compose up 명령어를 build 디렉터리 내에서 실행해줍니다.  
```sh
$ docker compose up
```
![](/assets/img/2022-12-05-docker_compose/docker_compose_up_log.png)
redis 서비스의 경우 필요한 redis 이미지를 pull 받아오고, web 서비스의 경우 이미지를 새로 빌드해야 하므로 Dockerfile을 통해 이미지를 빌드하는 단계를 거쳐 서비스를 띄우는 것을 확인할 수 있습니다.  

로그의 중간 부분을 보면 default network를 생성하는 부분도 확인할 수 있습니다.  
![](/assets/img/2022-12-05-docker_compose/create_default_network.png)
build_default라는 이름으로 브릿지 네트워크를 생성했는데, 여기에서 build는 프로젝트명을 의미합니다.  
프로젝트 명을 따로 설정하지 않은 경우에 프로젝트명은 디렉터리 명을 사용하게 됩니다.  

서비스의 경우에도 build-web-1, build-redis-1 서비스가 올라온 것을 확인할 수 있습니다.  
이때 build는 역시 프로젝트 명이고, redis/web은 서비스명이며 뒤의 1은 컨테이너 인덱스인데, 해당 서비스 상에서 컨테이너의 순서를 의미합니다.  

ctrl+c를 눌러 프로젝트를 종료한 후에 프로젝트 이름을 명시적으로 지정하여 다시 `docker-compose up`을 진행해보도록 하겠습니다.  
```sh
$ docker compose -p my-project up -d
```
-d 옵션을 통해 docker compose up 명령어를 백그라운드에서 실행시켰습니다.  

프로젝트가 다 올라왔다면 ls를 통해 docker-compose 프로젝트 목록을 확인해보도록 합니다.  
```sh
$ docker compose ls
```
![](/assets/img/2022-12-05-docker_compose/docker-compose-ls.png)
지정한 이름으로 프로젝트가 잘 생성된 것을 확인할 수 있습니다.  

### 프로젝트 종료 관련 명령어
돌아가고 있는 docker-compose 프로젝트를 종료하려면 아래 명령어를 사용하면 됩니다.  
```sh
$ docker compose down
```
`down`명령어만 사용하면 프로젝트의 컨테이너, 네트워크를 종료하고 제거합니다.  
만약 볼륨까지 제거하고 싶다면 `-v` 옵션을 추가로 주면 됩니다.  
```sh
$ docker compose down -v
```

## docker-compose 관련 기타 명령어
### 서비스 확장
```sh
$ docker compose up --scale [서비스명]=[개수]
```
단, 이렇게 서비스당 여러개의 docker container를 띄우는 경우 host의 포트를 지정해서 포트바인딩을 해놓으면 여러 컨테이너가 동일한 포트를 사용하려고 시도하여 문제가 될 수 있습니다.  
```dockerfile
# 호스트 포트 지정 O
versions: "3.9"
services:
    web:
        build: .
        ports:
        - "5000:5000"
```
위의 경우 처럼 호스트의 포트를 지정해두면 스케일링이 불가능하므로 ports 부분에 대해 호스트의 포트를 지정해두지 않아야 합니다.  
```dockerfile
# 호스트 포트 지정 X
versions: "3.9"
services:
    web:
        build: .
        ports:
        - "5000"
```

또한 서비스에 대해 컨테이너 네임을 지정해두는 경우도 스케일링이 불가능하기 때문에 컨테이너 네임을 지정하는 것도 지양해야합니다.  
```dockerfile
# 컨테이너 네임 지정으로 인해 스케일링 불가
versions: "3.9"
services:
    web:
        container_name: "web"
        build: .
        ports:
        - "5000"
```

### 그 외 명령어
- 프로젝트 내 서비스 로그 확인
    ```sh
    $ docker compose logs
    ```
- 프로젝트 내 컨테이너 이벤트 확인
    ```sh
    $ docker compose events
    ```
- 프로젝트 내 이미지 목록
    ```sh
    docker compose images
    ```
- 프로젝트 내 컨테이너 목록
    ```sh
    docker compose ps
    ```
- 프로젝트 내 실행중인 프로세스 목록
    ```sh
    docker compose top
    ```

## 주요 사용 목적
1. 로컬 개발 환경 구성
    - 개발하는 프로젝트가 의존성을 많이 가지고 있는 경우 프로젝트의 의존성을 지키면서 각 서비스들을 띄울 수 있게 됩니다.  
    - 의존도를 만족하는 로컬 개발 환경을 쉽게 구성할 수 있게 됩니다.  
2. 자동화된 테스트 환경 구성
    - CI/CD 파이프라인 중 격리된 테스트 환경을 만들고자 할 때 사용할 수 있습니다.  
    - 격리된 환경에서 특정 테스트에 필요한 부분을 구성한 뒤 해당 환경 내에서 테스트 코드를 돌릴 수 있게 됩니다.  
3. 단일 호스트 내 컨테이너를 선언적 관리
    - 단일 서버에서 컨테이너를 관리할 때 YAML 파일을 통해 선언적으로 각 서비스들을 관리할 수 있게 됩니다.  