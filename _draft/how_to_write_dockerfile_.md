# Dockerfile 작성법 

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
![](/assets/img/2022-12-03-how_to_write_dockerfile/arg_is_not_maintained.png)

### USER
도커 컨테이너가 사용할 기본 사용자를 지정할 때 사용합니다.  
```dockerfile
USER <user>[:<group>]
```
도커 이미지 보안을 고려할 때 사용하게 됩니다.  