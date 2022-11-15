---
title:  "Docker를 이용한 Hadoop Cluster 생성 (2)"
excerpt: "하둡 베이스 이미지를 사용하여 하둡 클러스터를 생성합니다."

categories:
  - Hadoop
tags:
  - [Hadoop, Docker, Docker-compose]

toc: true
toc_sticky: true
 
date: 2022-11-15
last_modified_at: 2022-11-15
---

# 네임노드의 도커 이미지 생성
## 작업 디렉토리 생성
가장 먼저 도커 환경을 설정할 디렉토리를 하나 생성해줍니다.  
저는 홈디렉토리에 HadoopStudy라는 이름의 디렉토리를 만들어보았습니다.  
```shell
$ mkdir ~/HadoopStudy
$ cd ~/HadoopStudy
```

## Dockerfile 작성
네임노드로 띄울 컨테이너에는 [이전 포스트](http://localhost:4000/hadoop/hadoop_cluster_setting/)에서 생성한 base 이미지를 그냥 사용해서는 안됩니다.  
다음 2가지 설정을 더 해주어야하기 때문입니다.  
1. $HADOOP_CONFIG_HOME/masters 파일 생성
2. $HADOOP_CONFIG_HOME/workers 파일 생성  

이 과정을 매번 도커가 뜰 때마다 하기 번거로우므로, 네임노드의 도커파일을 생성해서 이 과정을 자동화하도록 하겠습니다.  

먼제 namenode 도커 이미지 생성을 위한 디렉토리를 만들겠습니다.  
```shell
$ mkdir ~/HadoopStudy/namenode
$ cd ~/HadoopStudy/namenode
```
총 4개의 파일을 만들건데, masters, workers, start.sh, Dockerfile 파일을 만들겠습니다.  

```shell
$ vi masters
```
```
secondarynode
```
masters 파일에는 그냥 `secondarynode`라고만 적어주시면 됩니다.  
어떤 컨테이너가 secondarynode의 역할을 수행할 지 호스트명을 적어주는 부분이기 때문입니다.  

```shell
$ vi workers
```
```
secondarynode
datanode01
datanode02
```
workers 파일에는 데이터노드들의 호스트명을 적어주도록 하겠습니다. 

다음으로 start.sh입니다.  
```shell
$ vi start.sh
```
```
#!/bin/bash

/hadoop_home/hadoop/sbin/start-all.sh
/bin/bashv
```
하둡을 실행시키는 `/hadoop_home/hadoop/sbin/start-all.sh` 명령이 종료된 후에도, 컨테이너가 종료되지 않도록 /bin/bash를 추가적으로 적어주었습니다.  

마지막으로 도커파일을 만들도록 하겠습니다. 
```shell
$ vi Dockerfile
```
```Dockerfile
FROM {원격리포지토리명}/hadoop:base

WORKDIR /

ARG HADOOP_CONFIG_HOME=/hadoop_home/hadoop-3.3.4/etc/hadoop

COPY masters ${HADOOP_CONFIG_HOME}/masters
COPY workers ${HADOOP_CONFIG_HOME}/workers
COPY start.sh /start.sh
RUN chmod +x /start.sh
```
사용할 베이스 이미지와 수행할 디렉토리를 명시한 후 앞서 만든 workers와 masters 파일을 COPY하여 넣습니다. 

이렇게 하면 namenode의 도커 이미지를 만들 준비가 끝이 납니다.  

<br>   

# docker-compose 설정
## docker-compose 파일 작성
만든 작업 디렉토리 최상단에 docker-compose.yaml 파일을 만듭니다.  
```yaml
vi docker-compose.yaml
```

가장 먼저 파일 내에 버전을 명시해주도록 하겠습니다.  
```yaml
version: '3.4'
```

지금부터 
- 네임노드 1개 
- 데이터노드 겸 secondarynode인 노드 1개 
- 데이터 노드 2개   

총 4가지의 서비스를 생성하도록 하겠습니다.  

### 네임노드
가장 먼저 네임노드 설정입니다.  
```yaml
services:
  namenode:
    image: {원격리포지토리명}/namenode:hadoop
    container_name: namenode
    build:
      context: ./namenode
    hostname: namenode
    ports:
      - "50070:50070"
    extra_hosts:
      - "secondarynode: 172.28.0.3"
      - "datanode01: 172.28.0.4"
      - "datanode02: 172.28.0.5"
    tty: true
    depends_on:
      - secondarynode
      - datanode01
      - datanode02
    entrypoint: /start.sh
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.2
```
사용할 이미지는 앞서 namenode 디렉토리 내에 만든 Dockerfile을 통해 생성되는 namenode:hadoop 이미지입니다.  

build 부분을 통해 이미지 생성 경로를 표시합니다.  
```yaml
build:
      context: ./namenode
```
이 내용은 namenode 컨테이너를 위한 `{원격리포지토리명}/namenode:hadoop` 이미지를 빌드하기 위한 파일들은 namenode라는 디렉토리 하에 있다는 것을 뜻합니다.  

```yaml
extra_hosts:
  - "secondarynode: 172.28.0.3"
  - "datanode01: 172.28.0.4"
  - "datanode02: 172.28.0.5"
```
또한 데이터노드들의 호스트명을 인식할 수 있도록 `extra_host`에 호스트명과 ip주소들을 매핑해 주었습니다. 

```yaml
depends_on:
  - secondarynode
  - datanode01
  - datanode02
```
depends_on을 통해 다른 데이터 노드로 사용할 컨테이너들이 다 뜬 후에 namenode 컨테이너가 뜨도록 했습니다.  
이런 우선순위는 하둡 실행 시에 데이터노드로 사용할 컨테이너들을 제대로 인식할 수 있도록 하기 위해서 입니다.  

```yaml
entrypoint: /start.sh
```
entrypoint로는 앞에서 작업한 `/start.sh` 파일 주어서 컨테이너가 뜨면 하둡을 실행하도록 합니다. 

```yaml
networks:
  hadoop-network:
    ipv4_address: 172.28.0.2
```
`networks` 설정은 같은 네트워크를 공유하는 컨테이너들끼리 통신할 수 있게 도와줍니다.  
여기서 `ip4_address` 를 명시해서 `docker-compose up` 시에 항상 같은 ip 주소로 해당 컨테이너가 뜨도록 하였습니다.  
이렇게 하는 이유는 각종 환경 설정 파일 내에 ip 주소를 써야 하는 경우가 있기 때문에, 사전 세팅된 ip 주소 값들이 틀어지는 경우를 방지하기 위함입니다.  

### 데이터 노드
다음으로는 데이터노드에 대한 부분을 작성하도록 하겠습니다.  
```yaml
  datanode01:
    image: yoonjae/hadoop:base
    container_name: datanode01
    hostname: datanode01
    tty: true
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.4
```
네임노드보다 훨씬 간단합니다.  
모든 부분이 네임노드의 내용과 겹치므로 자세한 설명은 생략하도록 하겠습니다.  

### 네트워크
각 컨테이너간 통신을 할 수 있도록 앞에서 컨테이너가 hadoop-network라는 네트워크를 사용하도록 하였습니다.  
이 hadoop-network를 생성하는 코드는 아래와 같습니다.  
```yaml
networks:
  hadoop-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
```
bridge 네트워크를 생성해 주었습니다.  
bridge 는 도커에서 사용되는 가장 기본적인 형태의 network driver인 것 같습니다.  
네트워크에 추가로 ipam 설정을 해주어서 어떤 서브넷을 사용하게 될 지를 명시해주었습니다.  

### docker-compose.yaml 파일 전체
docker-compose.yaml 파일 전체 내용을 보면 아래와 같습니다.  
```yaml
version: '3.4'

services:
  namenode:
    image: yoonjae/namenode:hadoop
    container_name: namenode
    build: 
      context: ./namenode
    hostname: namenode
    ports:
      - "50070:50070"
    extra_hosts:
      - "secondarynode: 172.28.0.3"
      - "datanode01: 172.28.0.4"
      - "datanode02: 172.28.0.5"
    tty: true
    depends_on:
      - secondarynode
      - datanode01
      - datanode02
    entrypoint: /start.sh
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.2
        
  secondarynode:
    image: yoonjae/hadoop:base
    container_name: secondarynode
    hostname: secondarynode
    tty: true
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.3
  
  datanode01:
    image: yoonjae/hadoop:base
    container_name: datanode01
    hostname: datanode01
    tty: true
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.4
  
  datanode02:
    image: yoonjae/hadoop:base
    container_name: datanode02
    hostname: datanode02
    tty: true
    networks:
      hadoop-network:
        ipv4_address: 172.28.0.5
  

networks:
  hadoop-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16

```

<br>

# 클러스터 실행시켜보기  
모든 설정이 완료되었으므로 컨테이너를 모두 실행시켜 클러스터 구축이 잘 되었나 확인해보도록 하겠습니다.

```shell
$ docker compose build
```
위 명령어를 통해 namenode 이미지를 빌드합니다. 

```shell
$ docker images
```
이미지를 조회했을 때 `{원격리포지토리}/namenode:hadoop`이 조회된다면 이미지가 잘 빌드된 것이니 원격 리포지토리에 push해주도록 합시다.  
```shell
$ docker push {원격리포지토리}/namenode:hadoop
```
이제 모든 컨테이너를 생성해 클러스터를 실행시켜봅시다!!
```shell
$ docker compose up &
```

다음 명령어를 통해 컨테이너가 잘 떴는지 확인해봅시다. 
```shell
$ docker ps
```
`namenode, secondarynode, datanode01, datanode02`가 모두 잘 떴나요?

잘 떴다면 namenode 컨테이너 내부로 들어가보도록 하겠습니다.  

```shell
$ docker exec -ti namenode bash
```

컨테이너 내부에서 다음 명령어로 실행중인 자바 프로세스를 조회해봅니다.  
```
$ jps
```
이러한 결과가 나오면 됩니다.  
```
209 NameNode
1124 NodeManager
789 ResourceManager
1323 Jps
382 DataNode
```

데이터 노드도 잘 구동이 되는 지 궁금하니까 datanode도 들어가보도록 하겠습니다.
```shell
$ ssh datanode01
$ jps
```
아래 결과가 잘 나오는 지 확인해주세요!
```
68 DataNode
184 NodeManager
314 Jps
```