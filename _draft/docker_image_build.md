# 도커 이미지 빌드  

## 도커 이미지 구조
도커 이미지: 레이어 아키텍처  
새로운 사항이 생기면 새로운 레이어가 위에 쌓이는 구조
![](/assets/img/2022-12-02-docker_image_build/docker_image_structure.png)

이미지를 이용해서 컨테이너를 생성하는데, 이미지 레이어에 속해있는 부분은 Read Only이고, 그 위에 쌓이는 컨테이너의 레이어는 Read/Write가 모두 가능합니다.  

이미지 상세정보를 확인하는 명령어
```sh
$ docker image inspect [이미지명]
```
inspect 한 결과에서 Layer 부분을 보면 아래와 같이 각 레이어별로 해쉬값을 볼 수 있습니다.  
![](/assets/img/2022-12-02-docker_image_build/image_inspect.png)

## Dockerfile 없이 이미지 생성
기존 컨테이너를 기반으로 새 이미지를 생성하는 방법 -> docker commit 이용  
컨테이너 내부에서 만든 변경사항을 이미지로 저장하는 것
```sh
$ docker commit [OPTIONS] [CONTAINER] [REPOSITORY:[:TAG]]
```
옵션 종류
- -a: author. 누가 변경점을 남기는지 히스토리로 관리 가능
- -m: 커밋 메세지

예시
먼저 ubuntu:focal 이미지를 기반으로 하는 컨테이너를 하나 생성합니다.  
```sh
$ docker run -ti --name practice_ubuntu ubuntu:focal
```
컨테이너의 터미널에서 간단한 파일 하나를 만들어보도록 하겠습니다.  
```sh
$ cat > practice
```
원하는 내용을 적어주시고 ctrl+c를 눌러 편집을 마칩니다.   
파일이 생성되었다면 컨테이너를 종료하지 않고 터미널을 나오기 위해 ctrl+p+q를 눌러 컨테이너의 터미널에서 빠져나오도록 합니다.  
터미널로 나왔다면 앞에서 생성한 컨테이너로 이미지를 만들어보도록 하겠습니다.  
```sh
$ docker commit -a yoonjae -m "Add practice file" practice_ubuntu practice-ubuntu:v1
```

이미지가 잘 생성되었다면 생성된 이미지를 확인해보도록 하겠습니다.  
{% raw %}
```sh
$ docker image inspect -f {{.RootFS.Layers}} practice-ubuntu:v1
```
결과로 `practice-ubuntu:v1` 이미지의 레이어를 확인할 수 있습니다.  
`practice-ubuntu:v1` 이미지는 `ubuntu:focal` 이미지를 기반으로 만들어졌기 때문에 `ubuntu:focal` 이미지의 Layer도 확인해보도록 하겠습니다.  
```sh
$ docker image inspect -f {{.RootFS.Layers}} ubuntu:focal
```
![](/assets/img/2022-12-02-docker_image_build/layer_of_ubuntu_and_practice.png)
이미지는 앞에서 말했듯이 레이어 구조이기 때문에, `practice-ubuntu:v1` 이미지가 `ubuntu:focal` 이미지의 레이어를 포함하고 있음을 확인할 수 있습니다.  

{% endraw %}


## Dockerfile 이용하여 이미지 생성
Dockerfile의 구조
```Dockerfile
FROM [이미지명]
RUN [명령어]
WORKDIR [작업 수행 경로]
CMD ["수행할", "명렁어"]
```

도커 파일 기반의 이미지 생성 명령어
```sh
docker build [OPTIONS] PATH
```
옵션 종류 
- -t: tag. 빌드한 이미지의 tag를 지정해 줍니다.  
` docker build -t my_image:v1 .`
- -f: file 현재 경로에 있는 Dockerfile 이외에 다른 Dockerfile을 사용하고 싶을 때 사용  
`docker build -t my_image:v1 -f another_dir/Dockerfile .`

예시
간단하게 sh파일 하나를 실행하는 이미지를 만들어보도록 하겠습니다.  
작업할 디렉토리 생성 후 아래와 같이 Dockerfile을 작성해줍니다.  
```DOCKERFILE
FROM ubuntu:latest
WORKDIR ~/
COPY example.sh .
RUN chmod +x example.sh
CMD ["/bin/bash", "./example.sh"]
```

그리고 나서 위의 도커파일에서 사용하는 example.sh를 디렉토리 내에 생성해주도록 하겠습니다.  
```
#!/bin/bash

echo this is example
```

그 후에 아래 명령어로 도커 이미지를 생성합니다.  
```sh
$ docker build -t example:v1 .
```
![](/assets/img/2022-12-02-docker_image_build/docker_image_build.png)
마지막으로 생성한 이미지를 실행합니다.  
```sh
$ docker run example:v1
```
위의 쉘 스크립트에서 작성한 내용이 수행되는 것을 확인할 수 있습니다. 
![](/assets/img/2022-12-02-docker_image_build/docker_run_result.png)

## 빌드 컨텍스트
빌드 컨텍스트는 도커 빌드 명령 수행 시 현재 디렉토리를 의미합니다.  
빌드 컨텍스트는 이미지 빌드에 필요한 정보를 도커 데몬에 전달하기 위해 사용됩니다.  

도커 빌드를 수행할 때 해당 디렉토리의 모든 정보가 도커 데몬에 넘어가는 것이므로 해당 디렉토리의 용량이 크다면 빌드에 오랜 시간이 걸립니다.  

## .dockerignore
dockerignore는 gitignore와 비슷한 역할을 합니다.  
도커 빌드 시에 빌드 컨텍스트에 특정 디렉토리 또는 파일을 제외하는 데 사용됩니다.  
문법은 gitignore와 동일하게 사용됩니다.  
```
*/temp*
*/*/temp*
*.md
temp?
!README.md
```
참고로 *는 앞뒤로 모든 문자열을 포함하고, ?는 한 글자의 문자열을 의미합니다.  
`*/temp*`는 한 뎁스 내부에 있는 temp라는 문자열로 시작하는 파일을 의미하고, `*/*/temp*`는 두 뎁스 내에 있는 파일을 의미합니다.  
!가 붙은 파일은 예외사항을 표시하는 파일로, *.md파일이 있으므로 원래대로면 `README.md`파일도 ignore되어야 하지만 `!README.md` 라인이 있기때문에 README.md파일은 빌드 컨텍스트에 포함되어 도커 데몬으로 보내질 것입니다.  