# 리눅스 사용자, 그룹 및 권한  

## 사용자 및 권한 관련 명령어
### 계정 종류
리눅스 계정을 크게 두종류로 나누어보면 아래와 같습니다.  
- root 유저
- 사용자 계정

#### 계정 살펴보기  
계정을 살펴보기 위해 참고할 수 있는 파일은 아래와 같습니다.  
- /etc/passwd
- /etc/shadow
- /etc/group 

자신의 권한 및 계정에 대해 알아보기 위해 활용할 수 있는 명령어는 아래 두 가지가 있습니다.  
```sh
$ whoami
$ id
```

그룹 계정 및 권한을 살펴보기 위해 사용할 수 있는 명령어는 다음과 같습니다.  
```sh
$ sudoer
$ sudo
```

## 사용자 및 그룹 생성, 권한 관리
### 생성 및 삭제에 대한 기본 명령어
```sh
$ adduser
$ useradd
$ usermod
$ deluser
$ userdel
$ addgroup
$ delgroup
```

### 파일에 대한 권한
```sh
$ chmod
$ chown
$ chgrp
$ unmask
```

### 파일 다루기 심화
```sh
$ setuid
$ setgid
```