---
title:  "Docker 로그"
excerpt: "Docker에서 컨테이너의 로그를 어떻게 확인하고 활용할 수 있는 지 알아봅니다.  "

categories:
  - Docker
tags:
  - [Docker, Devops]

toc: true
toc_sticky: true
 
date: 2022-12-04
last_modified_at: 2022-12-04
---

# 도커 로그  

## STDOUT / STDERR
어플리케이션에서 로그를 다룰 때는 주로 해당 언어의 프레임워크에서 제공하는 로그 기능을 이용해서 표준 출력, sys.log, 외부 저장소 등으로 로그를 내보내왔습니다.  
  
도커 컨테이너 경우에는 어플리케이션의 로그를 표준 출력(stdout)과 표준 오류(stderr)로 내보내는 것을 표준으로 삼아야 합니다.  

어플리케이션에서 표준 출력 or 표준 오류로 로그를 내보내면 도커가 해당 로그를 쌓아서 logging driver가 처리할 수 있도록 하게 됩니다.  
> 도커의 로깅 드라이버는 여러 종류로 본인의 사용 목적에 맞게 선택하여 활용하면 됩니다.  
로깅 드라이버의 종류로는 `json-file, none, local, syslog, journald` 등이 있습니다.  

<br>

## 로그 확인하기 
도커 로그 관련된 명령어 및 각종 옵션은 아래와 같습니다.  
- 도커의 전체 로그 확인하는 명령어  
```sh
$ docker logs [container]
```

- 로그의 마지막 10줄만 확인하는 옵션  
```sh
$ docker logs --tail 10 [container]
```

- 해당 컨테이너의 로그 스트림을 실시간으로 확인하는 옵션  
```sh
$ docker logs -f [container]
```

- 로그마다 타임 스탬프를 표시하는 옵션  
```sh
$ docker logs -f t [container]
```

<br>

## 호스트 운영체제의 로그 저장 경로
호스트 운영체제의 로그 저장 경로에 로그 내역이 저장되는 것은 로그 드라이버를 json-file로 사용할 때만 유효합니다.  
이 떄 저장 경로는 `/var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log`입니다.  

참고로 mac의 경우 도커가 macOS 바로 위에서 돌지 않고, container를 hyperkit 위에서 돌리게 됩니다.  
따라서 mac에서 위의 경로로 진입해 로그를 보려고 하면 파일이 없는 것을 확인할 수 있습니다.    

이럴 때는 아래 경로에 있는 debug-shell.sock을 통해 도커가 돌아가고 있는 가상환경에 진입하여 로그를 확인할 수 있습니다.  
```sh
$ nc -U ~/Library/Containers/com.docker.docker/Data/debug-shell.sock
```

터미널에 진입이 되었다면 `/var/lib/docker/containers/${CONTAINER_ID}/${CONTAINER_ID}-json.log` 파일을 확인할 수 있을 것입니다.  

<br>

## 로그 용량 제한하기
로그 용량은 컨테이너 단위로 제한을 걸 수 있습니다.  
뿐만 아니라 도커 엔진단에서 기본 설정으로 제한하는 것도 가능합니다. (운영 환경에서는 이런 로그 용량 제한을 필수로 설정하는 것이 좋습니다.)  

로그 용량을 제한하여 도커 컨테이너를 실행시키는 방법은 아래와 같습니다.  
```sh
$ docker run \
--log-drivier=json-file \
--log-opt max-size=3m \
--log-opt max-file=5 [이미지명]
```
앞의 명령어는 한 로그 파일 당 최대 크기를 3Mb로 제한하고, 최대 로그 파일 3개로 로테이팅 한다는 뜻입니다.  

<br>

## 도커 로그 드라이버
가장 많이 사용되는 기본적인 로그 드라이버는 json-file입니다.  
json-file 드라이버를 사용해 쌓이는 로그는 아래와 같은 특성을 갖습니다.   
- inline json 형식의 파일로 저장되며 호스트 운영체제에 담긴다. 
- 따로 호스트 운영체제에 logstash 같은 로그 에이전트를 설치하여 본인이 사용하는 중앙화된 로그 시스템으로 로그를 수집하여 쌓을 수 있다.  

그 외에도 journald를 이용해 system journal 형태로 로그를 쌓는다던지, Syslog나 fluentd 등의 여러 방법을 이용해 로그를 쌓을 수 있습니다.
