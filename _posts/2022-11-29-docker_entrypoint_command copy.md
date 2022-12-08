---
title:  "Docker entrypoint, command, env variables"
excerpt: "Docker의 entrypoint, command와 환경 변수를 세팅하는 방법에 대해 알아봅니다.  "

categories:
  - Docker
tags:
  - [Docker, Devops]

toc: true
toc_sticky: true
 
date: 2022-11-29
last_modified_at: 2022-11-29
---

# 도커 엔트리포인트와 커맨드  
## 엔트리포인트 (Entrypoint)
- 도커 컨테이너가 실행될 때 고정적으로 실행되어야하는 스크립트 or 명령어입니다.  
- 생략이 가능합니다.  
- 생략 시에는 커맨드에 지정된 명령어를 수행합니다.  

## 커맨드 (Command)
- 단독 사용 시 -> 도커 컨테이너가 실행될 때 수행할 명령어로 작동합니다.    
- Entrypoint와 함께 사용 시 -> 엔트리포인트에 지정된 명령어에 대한 인자 값으로 작동합니다.  

## Entrypoint, Command 사용 예시
이름을 입력받아 print해주는 간단한 예제를 살펴보도록 하겠습니다.  
먼저 Dockerfile을 빌드할 환경을 만들기 위해 디렉터리를 생성해줍니다.  
```sh
$ mkdir practice
$ cd practice
```

entrypoint 수행 시점에 실행시킬 python 파일을 만들어보도록 하겠습니다.  
```sh
$ vi print_name.py
```
파이썬 파일의 이름은 print_name.py로 짓고, 아래 코드를 작성햅니다.  
```python
import sys
if __name__ == "__main__":
        name = sys.argv[1]
        print(f"Your name is {name}")
```
위 코드는 이름을 인자로 받아 단순히 출력하는 기능을 수행합니다.  

다음으로는 도커 파일을 만들도록 하겠습니다.  
```sh
$ vi Dockerfile
```
사용할 이미지는 python 이미지이며, Entrypoint로 파이썬 명령을 주고, command로 실행할 파이썬 파일과 해당 파일이 받는 인자값을 주었습니다.  
```Dockerfile
FROM python:3.7-slim
WORKDIR /
COPY print_name.py .

ENTRYPOINT ["python"]
CMD ["print_name.py", "yoonjae"]
```

코드 작업이 완료되었다면 디렉터리 내에서 `build` 명령어를 이용하여 이미지를 생성합니다.  
```sh
$ docker build -t test .
```
이미지를 실행시킵니다.  
```sh
$ docker run test
```
python 파일에서 print하도록 한 문장이 출력되면 됩니다.  
사용된 코드는 [이 주소](https://github.com/yunyun3599/DockerStudy/tree/master/entrypoint_and_command)에서 확인할 수 있습니다.  

## 도커 명령어에서 entrypoint와 command 전달  
도커 명령어에서 명시적으로 entrypoint와 command를 전달하는 경우 이미지에 지정되어있던 명령어를 override합니다.  

```sh
docker run --entrypoint echo ubuntu hello world
```
![](/assets/img/2022-11/2022-11-25-docker_entrypoint_command/docker_run_entrypoint_option_override.png)
entrypoint override 없이 그냥 동작시키면 bash가 실행되었는데, entrypoint와 command를 override 한 결과 `echo hello world` 가 실행됩니다.  

<br>

# 도커 컨테이너 환경변수  
## 도커 컨테이너 실행할 때 환경변수 설정하기  
환경 변수 관련하여 어떤 명령어들이 존재하는 지 보기 위해 help 옵션을 주어 설명을 확인해 보았습니다.  
```sh
$ docker run --help
```
![](/assets/img/2022-11/2022-11-25-docker_container_environment_variable/docker_run_help.png)
-e 부분을 보면 `--env` 와 `--env-file` 옵션이 있음을 확인할 수 있습니다. 

-e 옵션을 주어 환경변수를 컨테이너 생성 시점에 만들어보도록 하겠습니다.  
```sh
$ docker exec -ti -e name=hello ubuntu bash
```
도커 컨테이너 내부에서 아래 명령어를 치면 name 환경변수에 할당해놓은 hello 값이 출력되는 것을 볼 수 있습니다.  
```sh
$ echo $name
```
> 결과는 hello

만들 환경 변수를 파일 하나로 다 작성하고 모두 설정하려면 `--env-file` 옵션을 사용하면 됩니다.  
아래와 같이 환경변수 값을 가진 파일을 만들어보도록 하겠습니다.  
```sh
$ vi sample_env
```
```
VAR_NUM=123
VAR_STR=abc
```
그리고 난 다음에 ubuntu:focal 이미지를 이용해 컨테이너를 올리고 바로 env 명령어를 통해 위의 환경변수가 잘 들어갔는지 확인해보도록 하겠습니다.  
```sh
$ docker run -ti --env-file sample_env ubuntu:focal env
```
![](/assets/img/2022-11/2022-11-25-docker_container_environment_variable/2022-11-29-env_file.png)

