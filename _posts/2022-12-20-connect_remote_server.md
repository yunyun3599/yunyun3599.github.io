---
title:  "노트북 외부에서 ssh 연결하기"
excerpt: "외부에서 집 네트워크에 연결되어있는 노트북에 ssh 연결하는 법에 대해 알아봅니다."

categories:
  - Server
tags:
  - [Server, Etc]

toc: true
toc_sticky: true
 
date: 2022-12-20
last_modified_at: 2022-12-20
---

# 집의 우분투 서버에 ssh 접근하기
집에 있는 우분투 노트북에 외부에서 ssh 접근하는 법을 알아보겠습니다.  

## 우분투 정상 작동 확인
아래 명령어를 차례대로 수행하고 정상 작동하는 지 우선 확인해줍니다.  
```sh
$ sudo apt-get update
$ sudo apt-get upgrade
$ ping google.com
```

## ssh 연결
다음으로 랜카드의 논리적 이름을 알아내는 명령어를 터미널에서 수행합니다.  
```sh
sudo lshw -c network
```

그 결과로 아래와 같은 내용을 볼 수 있습니다.  
```
*-network
       description: Ethernet interface
       product: RTL~~~....
       vendor: RealTek Semiconductor Co., Ltd.
       physical id: 0
       bus info: pci@0000:00:~~
       logical name: enp4s0
       version: ~~
       serial: 00:00:00:00:00:00
       size: 1Gbit/s
       ~~~~
```
결과에서 logical name을 기억해둡니다.  

### 랜카드 활성화
```sh
$ sudo vi /etc/netplan/01-network-manager-all.yaml
```

파일에 아래 내용 작성하여 수정해줍니다.
```yaml
network:
    version: 2
    renderer: NetworkManager
    ethernets:
        <앞서 확인한 logical name>:
            dhcp4: no
            addresses:
                    - 192.168.**.**/24
            gateway4: 192.168.**.**
            nameservers:
                    addresses:
                            - 8.8.8.8
```
여기서 addresses에는 사용할 고정 ip 주소를 적어주었습니다.   
저장 후에 아래 명령어를 통해 설정을 적용해줍니다.  

```sh
sudo netplan apply
sudo reboot now
```

### ip 주소 알아내기
아래 명령어로 ip 주소를 알아냅니다.  
```sh
$ ip a
```

결과에서 앞서 확인한 논리명인 enp4s0 부분을 확인합니다.   
```
2: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether **:**:**:**:**:** brd **:**:**:**:**:**
    inet 192.168.**.**/24 brd 192.168.*.255 scope global dynamic eno1
       valid_lft 3538sec preferred_lft 3538sec
    inet6 ****::****:****:****:****/64 scope link
       valid_lft forever preferred_lft forever
```
위 결과에서 inet 부분의 `192.168.**.**` 애 해당하는 주소로 같은 네트워크 하에서는 ssh 연결을 진행할 수 있습니다.  

### 서버에 openssh-server 설치해두기
아래 명령어로 openssh-server를 설치해 둡니다.  
```sh
$ sudo apt-get install openssh-server
```

## 같은 네트워크 하에 있지 않은 경우
외부 네트워크를 사용하고 있을 때도 서버에 연결을 하고 싶다면 포트 포워딩이 필요합니다.  
무선 랜을 이용해 서버에 네트워크 연결을 해 둔 경우에는 공유기 포트 포워딩 + 모뎀 포트 포워딩을 진행해야 합니다.  

### 공유기 포트 포워딩
먼저 공유기 포트포워딩을 진행하자면 다음과 같습니다.  
저는 sk에서 제공되는 공유기로 인터넷을 사용중인데, 공유기 설정 주소인 192.168.35.1 로 접속합니다.  
만약 접속이 안되면 아래 명령어를 통해 기본 게이트웨이로 나온 주소로 접근합니다.  
```sh
$ ipconfig /all
```

설정 페이지가 나왔다면 로그인 정보는 다음과 같습니다.  
- id: admin 
- pwd: 유선Mac뒷6자리_admin  

여기서 유선 Mac 값은 공유기 기기의 뒷면에 적혀 있습니다.  

로그인 후 고급설정 > NAT 라우터 관리 > 포트포워드 메뉴로 들어갑니다. 
![](/assets/img/2022-12/2022-12-20-home_server_ssh/portforward.png)
그리고 각 항목을 아래와 같이 설정하고 추가 버튼을 누르면 포트 포워딩이 완료됩니다.  
- 프로토콜: Tcp 
- 외부포트: 서버로 진입할 때 사용할 포트로 임의 지정
- 포워딩 ip 주소: 포워딩할 서버의 내부 ip
- 내부포트: 내부에서 개방할 포트 범위


### 모뎀 포트포워딩
공유기 앞단의 모뎀도 포워딩을 해줘야 외부 네트워크에서 접근이 가능합니다.  
모뎀의 경우는 랜선을 통해 인터넷을 이용할 때의 gateway 주소로 들어가면 됩니다.  
저 같은 경우에는 192.168.75.1 로 접근 가능했습니다.  

모뎀 설정 페이지의 로그인 정보는 다음과 같습니다. 
- 아이디: admin
- 비밀번호: Mac 주소 뒷 여섯자리_admin

모뎀의 Mac 주소 뒷 여섯자리 역시 모뎀 기기에 부착되어 있습니다.  

좌측 메뉴의 방화벽 -> 포트포워딩 으로 접근합니다.   
그리고 아래 사진처럼 만약 무선 네트워크를 쓴다면 ip를 공유기의 ip 주소로 잡고, 공유기단에서 포트포워딩을 해줬던 포트를 포워딩해줍니다.  
만약 유선을 사용하는 경우에는 내부 ip 주소에 노트북의 ip 주소를 바로 적고 포트포워딩을 진행하면 됩니다.  
![](/assets/img/2022-12/2022-12-20-home_server_ssh/portforwarding_modem.png)

외부 네트워크에서 접속을 할 때는 `<모뎀의 외부 ip 주소>:<포트포워딩된 포트>` 로 접속하면 됩니다.  
참고로 모뎀의 외부 ip 주소는 집의 와이파이나 랜선에 연결된 단말에서 브라우저에 ip 주소 확인 이라고 쳤을 때 나오는 주소입니다.  
