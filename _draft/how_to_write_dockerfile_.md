# Dockerfile 작성법 및 도커 이미지 경량화

## 기본 구조
도커 파일의 기본 구조는 아래와 같습니다.  
```dockerfile
# 주석
INSTRUCTION arguments
```

많이 사용되는 Instruction을 포함한 Dockerfile의 기본 구조는 아래와 같습니다.  
```dockerfile
FROM [베이스 이미지]                    # 사용할 베이스 이미지
LABEL maintainer="이미지 관리자"        # LABEL을 통해 이미지에 대한 메타 정보를 관리 
WORKDIR /app                        # 작업을 수행할 경로
COPY [source 경로] [dest 경로]        # 호스트의 파일을 명시된 이미지의 경로로 COPY
RUN [실행할 명령]                     # 이미지상에서 수행할 명령
EXPOSE [포트번호]                    # 도커 이미지가 명시된 포트를 사용한다고 문서화하는 용도
CMD ["수행할", "명령"]               # 컨테이너를 실행할 때 수행할 명령
```

### ENV
```dockerfile
FROM busybox        # 사용할 base image
ENV FOO=/bar        # 환경변수 세팅
WORKDIR ${FOO}      # WORKDIR /bar
ADD . ${FOO}        # ADD . /bar
COPY \$FOO /quux    # COPY $FOO /quux
```
위에서 설정된 FOO는 환경변수로, 컨테이너에 속해있습니다.  
이미지 빌드 단계에서 `ENV` 명령어를 통해 지정한 환경변수는 이미지를 빌드할 때와 이를 실행시킨 컨테이너에서 모두 사용 가능합니다.  

### ARG
argument 지시어를 통해 환경변수를 전달할 때 사용합니다.  
```dockerfile
ARG <name>[=<default value>]
```
ARG로 정의된 변수는 기본 value값을 가져도 되고 따로 할당해놓지 않아도 됩니다.  

```dockerfile
FROM busybox
ARG user1=user
ARG buildno
```
값을 할당해놓지 않았더라도 도커 빌드 시점에 아래와 같이 값을 할당할 수 있습니다.  
```sh
$ docker build --build-arg buildno=1 .
```
> 참고로 도커 빌드 시점에 --build-arg로 argument를 주는 경우에는 무조건 Dockerfile에 해당 argument가 ARG 명령어를 통해 정의되어 있어야 합니다.  

이미지를 컨테이너화한 후에 ENV로 설정한 변수는 그대로 사용이 가능하지만 ARG로 설정한 변수는 사용이 불가능합니다.  
확인해보기 위해 아래와 같은 Dockerfile을 생성합니다.  
```Dockerfile
FROM ubuntu
ENV greeting=hello
ARG name
ENTRYPOINT echo ${greeting} ${name}
```
그리고 해당 도커파일을 통해 greeting:v1라는 이름의 이미지를 빌드합니다.  
```sh
docker build -f Dockerfile2 --build-arg name=이름 -t greeting:v1 .
```
앞에서 만든 이미지로 컨테이너를 만들어봅니다.  
```sh
docker run --rm greeting:v1
```
도커파일에서 `ENTRYPOINT echo ${greeting} ${name}` 라고 설정을 했기 때문에 `hello 이름` 이 출력되어야 할 것 같으나 name은 ARG로 선언한 값이기 때문에 `hello`만 출력되는 것을 확인할 수 있습니다.  
![](/assets/img/2022-12/2022-12-03-how_to_write_dockerfile/arg_is_not_maintained.png)

### USER
도커 컨테이너가 사용할 기본 사용자를 지정할 때 사용합니다.  
```dockerfile
USER <user>[:<group>]
```
도커 이미지 보안을 고려할 때 사용하게 됩니다.  

 <br>

## 도커 이미지 경량화
도커 이미지 경량화란 빌드를 통해 생성되는 도커 이미지의 크기를 줄이는 작업을 이야기합니다.  
도커 이미지 용량이 작아지면 push, pull 속도가 빨라지며 container를 띄우는 속도도 빨라집니다.  

도커 이미지를 경량화할 수 있는 방법 중 몇 가지를 알아보도록 하겠습니다.  

### 꼭 필요한 패키지 / 파일만 추가
서버에 필요할 것 같은 패키지를 모두 설치하지 말고 꼭 필요한 부분만 설치해야함.  
컨테이너는 하나의 프로세스를 실행하는 데 중점이 된 기술이므로 해당 프로세스를 실행하는 데 필요하지 않은 패키지나 파일은 굳이 추가하지 않는 것이 좋습니다.  

### 컨테이너 레이어 수 줄이기  
컨테이너 레이어 수는 도커 파일의 지시어 수와 동일합니다.  
따라서 Dockerfile 내에 사용하는 지시어 수를 줄일 수록 컨테이너의 레이어 수를 줄일 수 됩니다.  
예를 들어 RUN 명령어가 여러번 등장한다면 하나의 RUN 지시어로 통합하여 하나의 레이어로 통합해서 사용하는 방법이 있습니다.  

```dockerfile
RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y nginx
RUN echo finish!
```
위와 같이 여러 RUN 명령어를 하나로 통합하면 아래와 같은 형태가 됩니다.  
```dockerfile
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y nginx && \
    echo finish!
```
두 이미지 각각을 image:expand와 image:compress 라는 이름으로 빌드했는데 약소하지만 용량에 차이가 생겼음을 확인할 수 있습니다.  
![](/assets/img/2022-12/2022-12-03-how_to_write_dockerfile/docker_run_compress.png)

또한 패키지 설치 시 이미지에는 cache가 필요하지 않기 때문에 캐시를 남기지 않는 옵션을 추가해주는 것이 좋습니다.  


### 경량 베이스 이미지 사용하기  
베이스 이미지 자체를 경량 이미지를 사용하여 이미지의 용량을 줄일 수도 있습니다.  
많이 사용하는 경량 이미지의 예로는 아래와 같은 것이 있습니다.  
- debian 계열의 slim
- alpine 이라는 리눅스 배포버전들
- stretch 라는 이미지: 파일 시스템만 존재하는 비어있는 이미지라 용량이 굉장히 작음.  

### 멀티 스테이지 빌드 사용  
멀티 스테이지 빌드 기능을 이용하면 빌드 스테이지와 릴리즈 스테이즈를 나누어서 진행하게 됩니다.  
따라서 빌드에 필요한 의존성들은 빌드 스테이지에서 사용하고, 릴리즈 스테이지에서는 빌드 결과물만 복사해서 사용하여 릴리즈 이미지의 용량을 줄일 수 있게 됩니다.   

멀티 스테이지를 사용한 Dockerfile은 다음과 같은 구조를 가집니다.  
```dockerfile
FROM node:16-alpine AS base
LABEL maintainer="author"
LABEL description="Dockerfile to practice multistage"

WORKDIR /app

COPY package*.json

# build phase
FROM base AS build
RUN npm install

# release phase
FROM base AS release
# --from=build를 통해 빌드 스테이지에서 파일을 복사해옴
COPY --from=build /app/node_modules ./node_modules
# app 소스코드 복사
COPY . .

EXPOSE 8080
CMD ["node", "server.js"]
```

앞의 `FROM` 절에서 사용할 베이스 이미지를 지정하고 해당 이미지를 `AS base` 구문을 통해 base라고 지정해두었습니다.  
그리고 그 후의 build와 release 단계에서 이 이미지를 베이스 이미지로 사용해 각 단계에 필요한 작업을 수행하는 것을 볼 수 있습니다. 