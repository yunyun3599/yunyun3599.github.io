---
title:  "Vue Component(1)"
excerpt: "Vue에서 컴포넌트를 생성하고 각 요소를 어떻게 다루는 지 알아봅니다. "

categories:
  - Vue.js
tags:
  - [Vue.js, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2022-11-21
last_modified_at: 2022-11-21
---

# 컴포넌트
Vue에서 컴포넌트는 View, Data, Code로 이루어져 있습니다.  
컴포넌트에는 HTML코드와 함께 해당 코드 내에 들어갈 데이터, 그리고 코드를 실행하기 위한 자바스크립트 코드가 함께 어우러져 있습니다.   

기본적으로 컴포넌트는 아래와 같은 형식을 띄고 있습니다.  
{% raw %}
```vue
<template>
    <h1>Hello, {{title}}!</h1>
</template>

<script>
    export default {
        data() {
            return {
                title: 'World'
            };
        }
    }
</script>
```
{% endraw %}
가장 위의 `template` 태그 부분이 html 코드로, 전반적인 페이지의 구조를 담당합니다.  
밑의 `script` 부분에는 로직과 데이터가 들어가게 됩니다.  
위의 코드는 아주 간단한 코드이기 때문에 아직 딱히 메서드나 로직은 들어가지 않았지만, title이라는 변수명의 데이터는 포함하고 있습니다.  

위의 코드와 같은 페이지는 아래 사진과 같은 모습의 페이지를 렌더링하게 됩니다.  
![](/assets/img/2022-11-21-vue_component/2022-11-21-vue_component_1.png)

## 컴포넌트 기본 구조
많은 상황에서 사용되는 컴포넌트의 기본 구조는 아래와 같은 형태입니다. 
```vue
<template>
    <div></div>
</template>
<script>
    export default {
        name: '',           // 컴포넌트 이름
        components: {},     // import 할 다른 컴포넌트
        data() {            // 사용할 데이터
            return {
                name: ''
            };
        },
        setup() {},         // 컴포지션 API를 구성
        created() {},       // 컴포넌트가 생성되면 실행
        mounted() {},       // html 코드 렌더링 후 실행
        unmounted() {},     // unmount 후 실행
        methods: {}         // 컴포넌트 내에서 사용할 메서드 정의
    }
</script>
```

<br/>

# 데이터 바인딩
Vue는 양방향 데이터 바인딩을 지원합니다.  
양반향 데이터 바인딩은, 모델의 데이터와 뷰의 데이터가 상호 연동된다는 뜻입니다.  
즉 모델에서 바뀐 데이터는 뷰에 바로 적용이되고, 뷰에서 바뀐 데이터 역시 모델에 바로 적용이 된다는 뜻으로 받아들이시면 됩니다.  

위에서 봤던 코드를 다시 한번 보도록 하겠습니다.  
{% raw %}
```vue
<template>
    <h1>Hello, {{ title }}!</h1>
</template>

<script>
    export default {
        data() {
            return {
                title: 'World'
            };
        }
    }
</script>
```
여기서 `{{template}}`은 아래 `data() { }` 내에서 정의된 title 데이터를 뜻합니다.  
즉 아래의 모델 데이터와 뷰 데이터가 서로 연동되어 있다는 것입니다.  
{% endraw %}

위에서 만든 파일의 이름을 SimpleExample.js라고 짓고 프로젝트의 /src/views 경로에 저장했다고 합시다.  
이 페이지를 화면에서 보고싶다면 라우터에 해당 페이지를 추가해야합니다.  
라우터에 해당 페이지를 추가하기 위해서는 /router/index.js 파일의 routes 리스트에 관련 내용을 추가해야합니다.  
```js
{
    path: '/simpleexample',
    name: 'SimpleExample',
    component: () => import ('../views/SimpleExample.vue')
}
```
위의 내용이 추가된 전체 코드는 아래와 같습니다.  

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
    component: () => import(/* webpackChunkName: "about", webpackPrefetch: true*/ '../views/AboutView.vue')
  },
  {
    path: '/simpleexample',
    name: 'SimpleExample',
    component: () => import ('../views/SimpleExample.vue')
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

## HTML
html 태그를 바인딩 할 때는 앞의 데이터를 처리할 때처럼 {% raw %}`{{ }}`을 사용하지 않습니다.  
{% endraw %}
HTMl 출력을 위해서는 <span style="color:red;">v-html</span> 디렉티브를 사용합니다.  
html 태그를 바인딩 한 예시 코드는 아래와 같습니다.  
{% raw %}
```vue
<template>
    {{htmlString}}
    <div v-html="htmlString"></div>
</template>
<script>
    export default {
        data() {
            return {
                htmlString: '<div style="display:inline-block;">\
                <h1 style="background-color:#EEDC9A;">여러 요소 종합적으로 사용해보기</h1>\
                </div>'
            }
        }
    }
</script>
```

해당 페이지는 아래와 같습니다.  
![](/assets/img/2022-11-21-vue_component/2022-11-21-vue_component_2.png)  

앞에서처럼 ``{{ }}``를 사용한 경우는 html 문서의 내용이 그대로 출력되었으나, v-html를 사용한 부분은 원하는대로 html이 잘 적용되어 나타난 것을 확인할 수 있습니다. 

## Input type=text  
사용자로부터 텍스트를 입력받는 input태그의 경우, 입력받은 값은 value에 저장이 됩니다.  
이 때 v-model에 data()에 정의해둔 데이터의 변수명을 넣어주면 데이터 바인딩이 적용됩니다.  
```vue
<template>
    <div name="input-text">
        <input type="text" v-model="inputData" />
        <p>{{inputData}}</p>        
    </div>
</template>
<script>
    export default {
        data() {
            return {
                inputData: '입력받을 데이터'
            }
        }
    }
</script>
```
코드에서 볼 수 있듯 input 태그 내에 v-model을 **"inputData"**로 주었습니다.  
여기서 **"inputData"**는 아래 `data() { }` 내부에 정의되어 있는 모델입니다.  
화면에서 input창의 값을 변경하면 `<p>` 태그 내에서 출력하고 있는 inputData 데이터의 내용도 실시간으로 바뀌는 것을 확인할 수 있습니다.  

![](/assets/img/2022-11-21-vue_component/2022-11-21-vue_component_3.png){: width="70%"" }

## Input type=number
input에서 number 타입을 받고자 하는 경우에는 text를 받을 때와 약간의 차이가 있습니다.  
앞에서는 `v-model: "데이터 키값"` 의 형태로 태그 내에 데이터가 저장될 모델 정보를 주었습니다.  
이에 반해 number를 받는 경우는 `v-model.number="데이터 키값`의 형태로 데이터 바인딩을 진행합니다.  
```vue
<template>
    <div name="input-number">
        <input type="text" v-model.number="inputNumber" />
        <p>{{inputNumber }}</p>        
    </div>
</template>
<script>
    export default {
        data() {
            return {
                inputNumber: 0
            }
        }
    }
</script>

```
앞에서는 text 이번 코드에서는 number를 받으려고 했으므로 두 데이터를 화면에 보여줄 때 동일하게 +1을 해보았는데요, 아래와 같은 결과가 나왔습니다.  

![](/assets/img/2022-11-21-vue_component/2022-11-21-vue_component_4.png){: width="70%"}  
텍스트로 받았을 때는 1이 문자열로 뒤에 붙었고, 숫자로 받았을 때는 계산되었음을 확인할 수 있습니다.  
{% endraw %}
## Textarea
textarea는 \<textarea\> 태그와 함께 v-model을 명시하여 사용할 수 있습니다.  
{% raw %}
```vue
<template>
    <div name="text-area">
        <textarea v-model="inputText"></textarea> 
    </div>
</template>
<script>
    export default {
        data() {
            return {
                inputText: '텍스트를 입력하세요'
            }
        }
    }
</script>
```
위와 같이 코드를 작성하면 여러 줄을 작성할 수 있는 textarea가 화면에 보여지며, 입력되는 값은 inputText 모델과 바인딩 됩니다. 
{% endraw %}