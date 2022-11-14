---
title:  "Vue.js 개발환경 세팅"
excerpt: "Vue.js를 이용힌 개발을 시작하기에 앞서 필요한 환경을 설정합니다. vscode, node, npm을 설치합니다."

categories:
  - Vue.js
tags:
  - [Vue.js, Frontend, javascript]

toc: true
toc_sticky: true
 
date: 2022-11-13
last_modified_at: 2022-11-13
---

# Visual Studio Code 설치
Visual Studio는 마이크로소프트에서 제공하는 개발 도구 입니다.  
VSCode를 이용하면 다양한 플러그인들을 설치할 수 있고 이런 플러그인들을  사용하여 개발 생산성을 높일 수 있습니다.  
이 외에도 개발에 필요한 여러 편의성을 제공하기 때문에 정말 많이 사용되는 IDE 입니다.  

설치 방법은 [vscode 공식 사이트](https://code.visualstudio.com/download)에서 본인 운영체제에 맞는 설치파일을 다운로드해주시면 됩니다.
![](/assets/img/2022-11-13-vue_setting/2022-11-13-vue_setting_1.png)

mac의 경우 설치파일을 다운 받은 후에 별도의 설치 과정이 없으므로 잘 받았지셨으리라 믿겠습니다.. (믿음의 코딩👍)

<br>  

# Node.js 설치
Node.js는 자바스크립트 런타임 환경으로 서버 프로그램 개발을 자바스크립트로 가능하게 해줍니다.  
자바스크립트는 클라이언트에서 작동하는 언어인데, Node.js는 서버에서 자바스크립트를 동작할 수 있도록 해주는 환경인 것입니다.  

사실 저도 무슨 말인지 잘 모르겠어서 생각을 좀 해보니 이런 말인 것 같습니다.  
저희가 브라우저를 켜서 개발자도구를 들어가면 (fn + f12) console창을 볼 수 있는데요, 여기서 자바스크립트 언어로 코드를 짤 수 있습니다. 
![](/assets/img/2022-11-13-vue_setting/2022-11-13-vue_setting_2.png)

이건 브라우저에서 자바스크립트를 실행시키는 것이겠죠?  
이런식으로 원래 자바스크립트가 구동되는 고향(?)은 브라우저인데 Node.js를 통해 이 친구를 서버로 이사시켜서 서버에서도 자바스크립트를 동작시킬 수 있는 환경을 만들어준다는 의미인 듯 합니다.  

또한 Vue로 개발을 진행할 때 수많은 라이브러리들을 설치해야하는데, 이런 라이브러리 설치를 위해서는 Node.js가 설치되어야 합니다.  

Node.js 설치는 [Node.js 공식 사이트](https://nodejs.org/ko/download/)에서 본인의 os 에 맞는 버전을 다운로드 받습니다. 
![](/assets/img/2022-11-13-vue_setting/2022-11-13-vue_setting_3.png)

다운 받은 파일을 클릭하고 이런 창이 뜨면 계속을 계속 눌러 설치를 완료합니다.
![](/assets/img/2022-11-13-vue_setting/2022-11-13-vue_setting_4.png)

Node.js가 잘 깔렸는지 확인하기 위해 버전을 확인해보겠습니다.  
```shell
node -v
```
> v.19.0.1 . 

저는 v18.12.1을 깔았는데 19.0.1이라는 결과가 나왔습니다.  
이전에 깔아뒀던 Node가 있어서 이 버전을 찾은 모양입니다.  

제가 쓰고싶은 버전의 node는 /usr/local/bin에 위치했기 때문에 node alias를 저 경로로 설정해서 문제를 해결해보도록 하겠습니다. 

```shell
vi ~/.zshrc
```
편집 모드에서 아래 부분을 추가해줍니다.  
이 때 경로는 본인이 원하는 node 파일이 위치한 경로를 잡으시면 됩니다.   
```
alias node="/usr/local/bin/node"
```
그 후에 터미널을 재시작해줍니다.  
```shell
source ~/.zshrc
```

그리고 다시 `node -v`로 버전을 확인하면 원하는 버전으로 바뀌어 있을 것입니다.
```shell
node -v
```
> v18.12.1

<br>   

# npm 설치
npm은 Node.js 기반의 자바스크립트 오픈소스를 등록하고 간단한 명령어를 통해 설치하여 사용할 수 있게 해주는 패키지 매니저 입니다.  
앞에서 Node.js를 설치할 때 npm도 자동으로 함께 설치가 되었는데요, 다음 명령어를 통해 설치된 NPM 버전을 확인할 수 있습니다.  

```shell
npm -v
```
> 8.19.2 

<br>   

# Vue 개발에 유용한 Vscode Extension 설치 
익스텐션은 vscode 좌측 탭에서 Extension(네모 모양 4개 아이콘)을 눌러 들어가거나 cmd+shift+X를 통해 들어갈 수 있습니다. 

![](/assets/img/2022-11-13-vue_setting/2022-11-13-vue_setting_5.png)

여기서 vetur, javascript debugger, prettier - code formatter를 설치하면 됩니다. 

### ventur
.vue 파일에 Syntax Highlighting 기능을 지원합니다.  
이 기능이 있으면 변수, 메소드 명 등의 색상 처리가 다르게 되어 코드를 일고 작성하기에 용이합니다.  
또한 코드에서 문법에 맞지 않는 오류를 알려주는 역할도 수행합니다.

### javascript debugger
vs code 안에서 디버깅을 할 수 있도록 해줍니다.  

### prettier - code formatter
vue 프로그램 구현 시 코드 포맷을 지정된 형태로 변환해줍니다.