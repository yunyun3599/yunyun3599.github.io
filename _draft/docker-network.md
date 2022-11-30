# 도커 네트워크

## 네트워크 구조
도커에서 사용할 네트워크를 따로 지정하지 않으면 기본값으로 docker0 이라고 하는 브릿지 네트워크를 사용하게 됨.  
- **eth0**: 실제 호스트(도커에 의해 생성된 호스트X)에서 사용하고 있는 네트워크
- **docker0**: 도커 데몬에 의해 생성되는 기본 브릿지 네트워크
- **도커 컨테이너 내부**
    1. eth0: 컨테이너가 내부적으로 갖는 ip
    2. l0: 로컬 네트워크 (127.0.0.1)
- **veth**: 컨테이너 내부의 eth0과 대응되어 docker0에 연결될 수 있도록 해주는 가상 eth 

정리해보면, 컨테이너 내부의 eth0 하나하나마다 veth가 생성되고, veth를 통해 컨테이너의 eth0이 docker0에 연결되게 되는 것입니다.  


## 포트 포워딩
도커 컨테이너의 포트 노출은 `-p` 옵션을 통해 이루어집니다.   
사용 형태는 아래와 같습니다.   
```sh
$ docker run -p [HOST IP:PORT]:[CONTAINER PORT] [image]
```

호스트의 모든 IP의 80번 포트와 도커 컨테이너의 80번 포트를 매핑하고 싶다면 아래와 같이 하면 됩니다.  
```sh
$ docker run -p 80:80 [image이름]
```
만약 호스트의 127.0.0.1 IP의 80번 포트와 도커 컨테이너의 80번 포트를 매핑하고 싶다면 아래와 같이 하면 됩니다.  
```sh
$ docker run -p 127.0.0.1:80:80 [image이름]
```
호스트의 IP, Port를 지정하지 않으면 호스트의 사용 가능한 포트와 컨테이너의 포트를 매핑하여 진행합니다.  
```sh
$ docker run -p 80 [image이름]
```

nginx 이미지를 이용해 컨테이너를 생성하고 컨테이너의 80번 포트와 호스트의 80번 포트를 매핑해보도록 하겠습니다.  
```sh
$ docker run -d -p 80:80 nginx
```
컨테이너가 생성된 후 localhost:80번 포트에 접근해보도록 하겠습니다.  
```sh
$ curl localhost:80
```
![](/assets/img/2022-11-29-docke_network/curl_localhost80.png)
응답이 잘 오는 것을 확인할 수 있습니다. 

## expose / publish
expose는 문서화 용도
```sh
$ docker run -d --expose 80 nginx
```
publish는 실제 포트 바인딩
```sh
$ docker run -d -p 80 nginx
```
![](/assets/img/2022-11-29-docke_network/expose_and_publish.png)
expose를 이용한 컨테이너는 호스트와 포트가 바인딩되어있지 않은 것을 확인할 수 있습니다.  

## 도커 네트워크 드라이버
현재 존재하는 도커 네트워크 드라이버 목록 조회
```sh
$ docker network ls
``` 

### 도커 네트워크 드라이브 종류
**Native Driver**
- Single host networking: 단일 호스트에서 동작
    - bridge
        - docker0
        - 직접 생성한 user-defined bride network driver
    - host
    - none
- Multi host networking
    - overlay
        - 여러 서버가 있다고 할 때 각각의 서버 내에 있는 컨테이너들을 연결시켜주는 가상의 네트워크 
        - 멀티 호스트이므로 오케스트레이션 시스템(e.g. docker swarm)에서 많이 사용

**Remote Driver**
-  3rd-party plugins 설치하여 사용

### none 드라이버
아래 명령어로 none 드라이버를 사용하는 컨테이너를 생성해보도록 하겠습니다.  
```sh
docker run -ti --net none ubuntu:focal
```
그리고 `inspect` 를 이용해 위에서 생성한 도커 컨테이너의 IP주소를 확인해보도록 하겠습니다.  
{% raw %}
```sh
docker inspect -f "{{json .NetworkSettings.Networks}}" [container_name]
```
{% endraw %}
![](/assets/img/2022-11-29-docke_network/docker_inspect_none.png)
결과로 나온 부분에서 IPAddress 부분이 비어있으며, DriverOpts 값도 null인 것을 확인할 수 있습니다.   

None 드라이버는 해당 컨테이너가 네트워크 기능을 사용할 필요가 없거나 Custom Networking을 사용해야 할 때 이용합니다.  

### Host 드라이버  
도커가 제공하는 가상 네트워크가 아니라 실제 호스트의 네트워크에 붙어서 사용하는 개념  
포트바인딩을 하지 않아도 호스트 드라이버를 이용하므로 바로 접속 가능

Host 드라이버의 경우 linux host에서만 사용이 가능하고, Docker Desktop에서느 사용이 불가능하다.  

linux라고 가정하고 아래 명령어를 통해 컨테이너를 띄운다.  
```sh
$ docker run -d --network host nginx
```
host 네트워크로 띄웠으므로 localhost를 통해 nginx에 접근하려고 해도, 접근이 가능할 것이다.  
```sh
$ curl localhost:80
```

### Bridge 드라이버
사용자 정의 bridge 네트워크를 생성해보도록 하겠습니다.  
```sh
$ docker network create --driver=bridge bridgenetwork
```

앞에서 생성한 bridgenetwork 네트워크를 사용하는 nginx 컨테이너 하나, grafana 컨테이너 하나를 띄워보도록 하겠습니다.  
```sh
$ docker run -d --network=bridgenetwork --net-alias=nginx nginx
$ docker run -d --network=bridgenetwork --net-alias=grafana grafana/grafana
```
--net-alias=[도메인명] 옵션은 bridge 드라이버에서 사용 가능한 옵션으로, 동일한 bridge network를 사용하는 컨테이너간에 도메인으로 컨테이너 ip를 찾을 수 있게 해줍니다.  

위에서 생성한 nginx 컨테이너로 들어가보도록 하겠습니다.  
```sh
$ docker exec -ti [nginx_container_id] bash
```
여기에서 curl명령어를 이용해 grafana 컨테이너에 접근해보도록 하겠습니다.  
```sh
$ curl grafana:3000
```
![](/assets/img/2022-11-29-docke_network/curl_grafana.png)
제대로 응답이 오는 것을 확인할 수 있습니다.  

--net-alias 옵션을 주지 않고 grafana 컨테이너를 다시 띄워보도록 하겠습니다.  
```sh
$ docker stop [grafana_container_id]
$ docker run -d --network=bridgenetwork grafana/grafana
```

다시 nginx 컨테이너에 들어가 이전과 같이 grafana에 curl을 통해 응답을 받아오려고 해보았습니다.  
```sh
$ docker exec -ti [nginx_container_id] bash
$ curl grafana:3000
```
그 결과 아래와 같은 응답이 올 것입니다.  
> curl: (6) Could not resolve host: grafana

그러나 같은 네트워크를 쓰고 있으므로 아예 접근이 불가능 한 것은 아닙니다.  
아래 명령어를 통해 grafana의 IPAddress를 구해봅시다.  
```sh
$ docker inspect [grafana_container_id] | grep IPAddress
```
![](/assets/img/2022-11-29-docke_network/docker_inspect_grafana.png)
저는 172.18.0.3 주소를 가지고 있으니 해당 주소를 이용해서 다시 curl 명령을 날려보도록 하겠습니다.  
```sh
$ curl 172.18.0.3:3000
```
![](/assets/img/2022-11-29-docke_network/curl_ipaddress_grafana.png)

도메인 설정을 하지 않았으므로 grafana라는 도메인 명으로는 접근이 불가능하지만 ip주소를 통해서는 접근이 가능함을 알 수 있습니다.  