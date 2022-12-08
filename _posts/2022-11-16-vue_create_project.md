---
title:  "Vue 프로젝트 생성"
excerpt: "Vue 프로젝트를 생성하고, 프로젝트 구조를 확인합니다. "

categories:
  - Vue.js
tags:
  - [Vue.js, Frontend, javascript]

toc: true
toc_sticky: true
 
date: 2022-11-16
last_modified_at: 2022-11-16
---

# Vue CLI 설치  
Vue CLI는 Vue 프로젝트를 빠르게 구성하고 빌드 및 deploy 할 수 있도록 도와주는 도구입니다.  
Vue CLI 설치는 아래 명령어를 통해 진행합니다. 
```shell
$ npm install -g @vue/cli
```
npm은 라이브러리를 패키지 형태로 바로 설치해서 사용할 수 있도록 도와주는 도구입니다.  
npm 설치 관련 내용은 [이전 포스트](https://yunyun3599.github.io/vue.js/vue_setting/#nodejs-%EC%84%A4%EC%B9%98)에 있습니다.  

`npm install`을 통해 패키지를 설치할 때 여러 옵션을 줄 수 있습니다.  
예를 들어  `npm install -g 패키지` 명령어를 통해 설치를 하면 해당 패키지가 어디서나 사용할 수 있는 global 패키지로 등록됩니다.  
위에서 Vue CLI는 vue 프로젝트를 시작할 때마다 쓰게 되므로, global로 설치한 것입니다.  

`npm install 패키지명 --save` 를 통해 설치를 진행하면 현재 작업중인 디렉토리 내의 <span style="background:#fffae0;">./node_modules</span>에 패키지를 설치합니다.  
그리고 package.json 파일의 dependencies 객체에 설치된 패키지 정보를 추가합니다.  
이렇게 되면, 해당 프로젝트에서 사용중인 모든 패키지를 .node_modules 에서 확인할 수 있습니다.  

여러명과 프로젝트를 할 때 --save 옵션을 주어 설치를 하게 되면 자동으로 설치된 패키지 관리가 되기 때문에, 따로 패키지 설치 목록을 공유하는 번거로움을 줄일 수 있습니다.

또한 package.json에 설치된 모든 파일 정보가 추가되어 있으므로, 새로 추가된 패키지들을 한번에 설치할 때도 `npm install` 명령어를 통해 손쉽게 진행할 수 있습니다. 

<br>

# 프로젝트 설치하기
Vue CLI가 설치되었으므로 vue 프로젝트를 생성해보도록 하겠습니다.  
```shell
$ vue create {프로젝트명}
```
명령어를 치면 아래와 같은 선택지가 나옵니다.  
![](/assets/img/2022-11/2022-11-16-vue_create_project/2022-11-16-vue_create_project_1.png)  
사용할 vue 버전을 선택하시면 됩니다.  
저는 vue 3 버전을 설치하였습니다.  

설치가 완료되면 아래와 같은 화면을 볼 수 있습니다.  
![](/assets/img/2022-11/2022-11-16-vue_create_project/2022-11-16-vue_create_project_2.png)

생성된 프로젝트로 들어가보도록 하겠습니다.  
```shell 
$ cd {프로젝트명}
```

프로젝트 내에서 서버를 실행하려면 다음 명령어를 실행하면 됩니다.  
```shell
$ npm run serve -- --port 3000
```
만약 다른 포트를 사용하고 싶다면 `port {원하는 번호}` 로 설정하면 됩니다.  
실행이 완료되면 http://localhost:3000 에 웹 페이지가 뜨게 됩니다.  
![](/assets/img/2022-11/2022-11-16-vue_create_project/2022-11-16-vue_create_project_3.png)  
위 화면이 보인다면 프로젝트 생성이 잘 된 것입니다.  

<br>

# vue 프로젝트 파일 구조
생성된 프로젝트의 구조는 아래와 같습니다.  
![](/assets/img/2022-11/2022-11-16-vue_create_project/2022-11-16-vue_create_project_4.png){: width="80%"}  
목록 중 주요한 파일의 역할을 알아보도록 하겠습니다.  
- node_modules/ : npm으로 설치된 패키지 파일이 모여있는 디렉토리  
- public/ : 정적 리소스가 모여있는 디렉토리  
- src/assets/ : 이미지, css, 폰트 등을 관리하는 디렉토리  
- src/components/ : Vue 컴포넌트 파일을 모아두는 디렉토리  
- App.vue : 최상위 컴포넌트 (Root Component)
- main.js : 가장 먼저 실행되는 자바 스크립트 파일. Vue의 인스턴스를 생성하는 역할을 수행  
- .gitignore : 깃허브에 업로드하지 않을 파일들의 정보를 적어두는 파일
- babel.config.js : 바벨(Babel) 설정 파일
- jsconfig.json : vscode에서 프로젝트를 작업할 때 해당 파일이 있는 곳이 js 프로젝트의 루트 디렉토리임을 나타내는 파일
- package-lock.json : 설치된 package의 dependency 정보를 관리하는 파일  
- package.json : 프로젝트에 필요한 package를 정의하고 관리하는 파일  
- README.md : 프로젝트에 대한 설명을 적어두는 파일

<br>

# package.json 내용
프로젝트 생성 직후 package.json의 내용은 다음과 같습니다.  
```json
{
  "name": "vue-project",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
  },
  "dependencies": {
    "core-js": "^3.8.3",
    "vue": "^3.2.13"
  },
  "devDependencies": {
    "@babel/core": "^7.12.16",
    "@babel/eslint-parser": "^7.12.16",
    "@vue/cli-plugin-babel": "~5.0.0",
    "@vue/cli-plugin-eslint": "~5.0.0",
    "@vue/cli-service": "~5.0.0",
    "eslint": "^7.32.0",
    "eslint-plugin-vue": "^8.0.3"
  },
  "eslintConfig": {
    "root": true,
    "env": {
      "node": true
    },
    "extends": [
      "plugin:vue/vue3-essential",
      "eslint:recommended"
    ],
    "parserOptions": {
      "parser": "@babel/eslint-parser"
    },
    "rules": {}
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```
- private: true인 경우 해당 프로젝트를 npm으로 올릴 수 없음.
- scripts: 프로젝트 실행과 관련된 명령어를 등록. 개발자가 정의한 script는 npm run 명령어로 사용하고, npm에서 제공하는 명령어는 npm 명령어로 사용  
- dependencies: 사용중인 패키지 정보를 입력
- devDependencies: 프로젝트 배포 시에는 필요 없고 개발에만 사용되는 패키지 정보 등록
- eslintConfig: Eslint란 코드 작성을 도와주기 위해 코드에서 발견된 패턴을 개발자에게 알려주는 플러그인임. 구문 분석을 위해 @babel/eslint-parser를 파서로 사용  
- browserlist: 전 세계 사용 통계 속에서 상위 1% 이상 사용된 브라우저, 각 브라우저의 최신 버저 2개를 지원하도록 함  

<hr/>
<br>

이 외에도 직접 프로젝트를 설정하여 생성하는 방법과 Vue 프로젝트 매니저를 이용해서 gui 환경에서 프로젝트를 생성할 수도 있습니다.  

**직접 생성**
1. `vue create vue-project-manually` 명령어를 통해 설치를 시작하고, 
2. 설치 옵션에서 'Manually select features'를 선택한 후에 
3. 필요한 피처들을 직접 선택 / 설정하여 진행합니다.  

<br>   
**프로잭트 매니저**  
`vue ui` 명령어를 통해 vue 프로젝트 매니저를 실행시킨 후 기본 포트인 8000번 포트로 브라우저 내에서 프로젝트 매니저를 사용할 수 있습니다.  
