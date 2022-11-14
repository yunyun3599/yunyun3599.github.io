---
title:  "Docker를 이용한 Hadoop Cluster 생성 (1)"
excerpt: "하둡 클러스터 구축을 위한 베이스 이미지를 생성합니다."

categories:
  - Hadoop
tags:
  - [Hadoop, Docker, Docker-compose]

toc: true
toc_sticky: true
 
date: 2022-11-14
last_modified_at: 2022-11-14
---

# Hadoop 클러스터 생성
Hadoop은 네임노드와 데이터노드로 이루어져 있는데요,  
네임노드로 쓸 컨테이너 1개와 데이터 노드로 쓸 컨테이너 3개를 포함하여 하둡 생태계를 만들어 보도록 하겠습니다. 

<br>   

# 베이스 이미지 준비
## 도커 base image pull & run
저희는 Centos 환경의 도커에 hadoop을 설치할 계획이기 때문에 centos 이미지를 pull 받아오겠습니다. 
```shell
docker pull centos:7
```

받은 이미지를 실행시켜 줍니다.
```shell
docker run -ti --name base centos:7 /bin/bash
```

<br>

# 컨테이너 내부 설정
## 필요 패키지 설치
위의 명령어를 통해 이미지를 실행시키면 실행과 동시에 컨테이너 환경으로 들어갈 수 있습니다.  
컨테이너 내에서 하둡 기본 이미지 생성을 위한 몇 가지 단계를 수행하도록 하겠습니다.  
아래 명령어를 하나씩 컨테이너 내에서 실행합니다.
```shell
yum update
yum install wget -y
yum install vim -y
yum install openssh-server openssh-clients openssh-askpass -y
yum install java-1.8.0-openjdk-devel.aarch64 -y
```
java의 경우 만일 위의 버전을 지원하지 않는다면 `yum list java*jdk-devel` 명령어를 통해 지원하는 자바 버전을 확인 후 설치하시면 됩니다.  

설치된 자바의 환경 변수 설정을 해주어야 합니다.  
```shell
readlink -f $(which javac)
```
위의 명령 결과로   
>/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.352.b08-2.el7_9.aarch64/bin/javac  

와 유사한 경로가 나올텐데, 이 경로에서 bin 디렉토리 전까지의 경로를 JAVA_HOME으로 추가합니다. 
```shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.352.b08-2.el7_9.aarch64
```

## ssh 설정
여기까지 완료가 됐다면 각 노드들이 패스워드 입력 단계 없이 서로 접속할 수 있도록 공개키를 생성해주도록 하겠습니다.  
```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
ssh-keygen -f /etc/ssh/ssh_host_rsa_key -t rsa -N ""
ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -t ecdsa -N ""
ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -t ed25519 -N "" 
```

## 하둡 설치
먼저 하둡을 설치할 디렉토리를 만들어주도록 하겠습니다. 
```
mkdir /hadoop_home
cd /hadoop_home
```

저는 hadoop-3.3.4.tar.gz 를 받았는데요, wget을 통해서 아래처럼 다운받을 수 있습니다.   
```shell 
wget "https://dlcdn.apache.org/hadoop/common/stable/hadoop-3.3.4.tar.gz"
```

설치 후에는 압축을 풀고 하둡을 설치합니다.  
```
tar xvfz hadoop-3.3.4.tar.gz
```
설치 후에는 hadoop-3.3.4 디렉토리에 대한 심볼릭 링크를 생성합니다. 
```
ln -s hadoop-3.4.3 hadoop
```

설치가 완료됐다면 아래와 같이 ~/.bashrc 파일에 내용을 추가하여 환경변수 설정을 해줍니다.  
```
vim ~/.bashrc
```
아래 내용을 추가해주세요.
```
export HADOOP_HOME=/hadoop_home/hadoop-3.3.4   
export HADOOP_CONFIG_HOME=$HADOOP_HOME/etc/hadoop  
export PATH=$PATH:$HADOOP_HOME/bin  
export PATH=$PATH:$HADOOP_HOME/sbin  
/usr/sbin/sshd  
```   


변경사항을 반영하기 위해 터미널을 재시작해보도록 하겠습니다.
```shell
source ~/.bashrc
```

## 하둡 설정파일 변경  
하둡 설정 파일을 변경하기 위해 설정 파일이 있는 디렉토리로 이동합니다.  
```
cd $HADOOP_CONFIG_HOME
```

### core-site.xml  
```shell
vi core-site.xml
```
아래 내용을 적어줍니다.  
```xml
<configuration>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/hadoop_home/temp</value>
        </property>

        <property>
                <name>fs.default.name</name>
                <value>hdfs://namenode:9000</value>
                <final>true</final>
        </property>
</configuration>
```
여기서 `hdfs://namenode:9000` 부분의 namenode는 추후 namenode로 삼을 도커 컨테이너의 호스트명을 적어준 것입니다.  

hadoop.tmp.dir의 value에 적은 경로를 생성해주어야 하므로 아래와 같이 디렉토리를 생성해줍니다. 
```shell
mkdir /hadoop_home/temp
```

### hdfs-site.xml
```shell
vi hdfs-site.xml
```
```xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
                <final>true</final>
        </property>

        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/hadoop_home/namenode_dir</value>
                <final>true</final>
        </property>

        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/hadoop_home/datanode_dir</value>
                <final>true</final>
        </property>
        <property>
                <name>dfs.http.address</name>
                <value>namenode:50070</value>
        </property>
        <property>
                <name>dfs.secondary.http.address</name>
                <value>secondarynode:50090</value>
        </property>
</configuration>
```
여기서 namenode와 secondarynode 역시 추후 사용할 도커 컨테이너의 호스트 명입니다.  

dfs.namenode.name.dir과 dfs.datanode.data.dir의 value에 적은 경로를 생성해주어야 하므로 아래와 같이 디렉토리를 생성해줍니다. 
```shell
mkdir /hadoop_home/namenode_dir
mkdir /hadoop_home/datanode_dir
```

### mapred-site.xml
```shell
vi mapred-site.xml
```
```xml
<configuration>
        <property>
                <name>mapred.job.tracker</name>
                <value>namenode:9001</value>
        </property>
</configuration>
```

<br>

# 도커 이미지화
네임노드를 포맷해줍니다.
```shell
hadoop namenode -format
```  
그 후 다른 터미널을 열고 아래 명령어로 현재 컨테이너를 이미지화 해줍니다.
```shell
docker commit base {원격리포지토리명}/hadoop:base
```

```docker images``` 명령어를 치면 hadoop:base 이미지가 생성되었음을 확인할 수 있습니다.  

해당 이미지를 원격 리포지토리에 업로드하고 여러 장소에서 사용할 수 있도록 이미지를 push 해보도록 하겠습니다. 
```shell
docker push {원격리포지토리명}/hadoop:base
```