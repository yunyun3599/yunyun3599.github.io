---
title:  "Vue Router 설정"
excerpt: "Vue에서의 라우팅에 대해 알아보고, 사용해봅니다."

categories:
  - Vue
tags:
  - [Vue, Frontend, javascript]

toc: true
toc_sticky: true
 
date: 2022-11-18
last_modified_at: 2022-11-18
---

# 라우팅 개념  
웹 사이트를 사용하다보면 다른 페이지로 이동할 때 url 주소가 달라지는 것을 확인할 수 있습니다.  
Vue는 단일 페이지 애플리케이션이기 때문에, 페이지 이동 시에 서버에 요청하여 페이지를 새로 가져오는 것이 아니라, 클라이언트에서 미리 가지고 있던 페이지를 라우팅하여 화면을 새로 그리게 됩니다.  

따라서 **라우팅**이란 클라이언트에서 url 주소에 따라 페이지를 전환시키는 것이라고 볼 수 있습니다.  
vue를 이용해 프로젝트를 할 때 코드 내부에서 미리 url 주소를 정의하고, 각 주소마다 vue 페이지를 연결해 놓습니다.  
따라서 사용자가 원하는 페이지로 이동하고자 할 때 미리 정의된 url 주소와 매핑된 vue 페이지로 화면을 전환시키는 것입니다.  

vue에서 라우팅을 하기 위해 사용하는 플러그인은 <span style="background:#fffae0;">vue-router</span> 입니다.  

<br>

# Vue-Router 설치
vue-router 설치는 아래 명령어를 통해 진행합니다.   
```shell
$ vue add router
```
설치가 완료되면 src 폴더에 router, views 폴더와 파일이 생성됩니다.  

```shell
$ npm run serve
```
위 명령어를 통해 서버를 재시작하고 http://localhost:8080 에 접속하면 화면 상단에 Home|About 링크가 생성되는데요, 이 링크를 누르면 화면이 전환되는 것을 확인할 수 있습니다.  
![](/assets/img/2022-11/2022-11-18-vue_router/2022-11-18-vue_router_1.png)

vue-router를 설치하면서 몇 가지 파일의 내용이 바뀌었는데요, App.js부터 확인해보도록 하겠습니다.  
```js
<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view/>
</template>
```
위 코드의 `router-link to="/">Home</router-link>` 부분을 통해 다른 페이지로 이동할 수 있는 링크를 생성할 수 있습니다.  

다음으로 살펴볼 파일은 index.js 입니다.  
```js
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const routes = [
  {
    path: '/',
    name: 'home',
    component: HomeView
  },
  {
    path: '/about',
    name: 'about',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/AboutView.vue')
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router

```
`routes` 배열에 2개의 라우트가 등록되어 있는데 순서대로 home고 about 컴포넌트에 대한 부분입니다.    
1. > {path: '/', name: 'home', component: HomeView}
2. > {path: '/about', name: 'about', component: () => import('../views/AboutView.vue')}  

여기서 path는 브라우저에서 접속하는 url 주소입니다.   
component는 지정된 path로 들어왔을 때 보여줄 vue 컴포넌트로 앞으로 구현할 vue 파일을 연결하고, 해당 파일을 실행시킵니다. 

위에서 home과 about 라우트를 import하는 `component` 부분의 코드가 서로 달랐습니다.  
이 부분이 달랐기 때문에 home의 경우 사용자가 해당 path에 접근하지 않더라도 이미 vue 파일을 import를 하나 about의 경우 path에 접근하기 전까지는 vue에 대한 import가 일어나지 않는다는 차이점이 있습니다.  

about의 경우 라우트 배열에 등록된 부분에 주석 처리된 부분들이 있습니다.  
```js
{
    path: '/about',
    name: 'about',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/AboutView.vue')
  }
```
각 부분에 대한 내용은 다음과 같습니다.  
1. `// route level code-splitting`
    - 라우트 레벨에서 코드를 분할하는 방법입니다. 
2. `// this generates a separate chunk (about.[hash].js) for this route`
    - 이 라우트에 대한 chunk 파일이 분리되어 생성됩니다.
3. `// which is lazy-loaded when the route is visited.`
    - 이 라우트에 방문했을 때 lazy-load(지연 로드) 됩니다. 
4. `component: () => import(/* webpackChunkName: "about" */ '../views/AboutView.vue')`
    - 라우트 레벨에서 코드를 분할한 후 별도의 chunk 파일을 생성하고, 실제 이 라우트를 방문했을 때 리소스를 로드합니다. 여기서 컴포넌트 import 시 `/* webpackChunkName: "about" */` 러눈 주석으로 chunk 파일 이름을 정의했기 때문에 chunk파일은 about이라는 이름으로 생성됩니다.

<br>

# Lazy Load 적용하기  
Vue Cli를 통해 빌드하면 소스코드가 하나의 파일로 합쳐지는데, 큰 프로젝트에서는 그 결과 파일 용량이 매우 커집니다.  
파일 용량이 크면 사용자가 처음 페이지에 접속했을 때 다운로드 받는 속도가 느려지게 된다는 단점이 있습니다.  

Lazy Load는 리소스를 컴포넌트 단위로 분리하여 컴포넌트나 라우터 단위로 필요한 것들만 필요한 시점에 다운받을 수 있게 해주는 방법입니다.  

## prefetch
**prefetch란?** 
- 미래에 사용될 수 있는 리소스(비동기 컴포넌트)를 캐시에 저장하는 기능으로 사용자가 접속했을 때 리소스를 빠르게 내려줄 수 있다는 장점이 있다.  
- 하지만 당장 필요하지 않은 리소스까지 비동기 컴포넌트로 정의된 모든 리소스를 캐시에 담는 비용이 발생한다.  

**prefetch를 사용하면**  
생성된 모든 chunk 파일 각각에 대한 request가 발생하며 다운로드 받은 파일을 캐시에 저장합니다.  
때문에 잘못하면 prefetch를 통해 렌더링 시간을 줄이려는 원래 목적과 다르게, 오히려 늘어난 request 요청으로 인해 렌더링 시간이 증가하는 부작용이 생길 수 있습니다.  

> Lazy Load가 적용된 컴포넌트에 대해서는 prefetch의 기본값이 true입니다.  
-> 모든 chunk에 대한 request가 발생함으로, 요청 수가 많아 집니다.  

**따라서 결론적으로**  
prefetch를 사용하면 모든 리소스를 다 받고 첫 화면에 사용되는 리소스를 마지막으로 받은 후 렌더링이 완료되기 때문에 초기 렌더링 속도가 느릴 수 있습니다.  
따라서 정말 필요한 컴포넌트에 한해서만 prefetch를 사용하는 것이 필요합니다.  

앞에서 Lazy Load된 컴포넌트에 한해 prefetch가 기본적으로 켜져있다고 했는데 이 기본값을 바꾸고 싶다면 다음과 같이 설정하면 됩니다.   

**Vue.config.js 파일에 아래 내용 추가**
```js
module.exports = {
  chainWebpack: config => {
    config.plugins.delete('prefetch');
  }
}
```

prefetch 기능을 삭제해도 Lazy Load로 컴포넌트를 사용할 수 있는데, 이 때 설정해야하는 부분은 다음과 같습니다. 
```js
import(/* webpackPrefetch: true" */ '../views/AboutView.vue')
```
이렇게 import 부분에 주석을 추가해주면 해당 컴포넌트에 대해서는 prefetch가 적용됩니다.  

# import router 및 main.js 설정
앞에서 router/index.js에서는 라우트들의 정보를 가진 router를 생성했었습니다.  
```js
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})
```
여기서는 `vue-router`에서 import한 `createRouter, createWebHistory`를 사용했었습니다.  
이렇게 생성한 라우터를 main.js에 등록해줘야 실제로 적용이 되어 사용할 수 있습니다.  

main.js의 내용은 다음과 같습니다.  
```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```
createApp에 최상위 컴포넌트인 App.vue로 app을 생성하고, use(router) 코드를 추가해서 앞에서 생성한 router가 사용되도록 하였습니다.  
그리고 App.vue를 public 폴더의 index.html에 정의되어있는 id='app'인 html element에 마운트 시키게 됩니다.  

밑의 코드가 index.html인데 `<!-- built files will be auto injected -->` 부분에 들어가게 되는 것입니다. 
```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>

```

라우터를 살펴보면서 
1. 특정 url로 이동하면
2. 해당 페이지의 .vue 파일이 호출되고
3. 이 코드가 실행되며 화면에 나타난다는 것을 알 수 있었습니다.  

여기서 실행되는 .vue 파일을 <span style="color:red">컴포넌트</span>라고 합니다. 