---
title:  "vue component 사용하기(1)"
excerpt: "vue에서 컴포넌트를 사용할 수 있도록 props에 대해 알아봅니다."

categories:
  - Vue
tags:
  - [Vue, Frontend, javascript, component]

toc: true
toc_sticky: true
 
date: 2023-01-30
last_modified_at: 2023-01-30
---
# Vue Component

## 컴포넌트 내에 다른 컴포넌트 사용하기  
vue를 통해 작업할 때 컴포넌트 내에서 다른 컴포넌트를 import하여 재사용할 수 있습니다.  
다른 컴포넌트를 import해 사용하는 코드는 다음과 같습니다.  
```vue
<template>
    <div>
        <Subcomponent1/>
        <Subcomponent2/>
    <div>
</template>
<script>
    import Subcomponent1 from './Subcomponent1'
    import Subcomponent2 from './Subcomponent2'

    export default {
        components: {Subcomponent1, Subcomponent2}
    }
</script>
```

앞의 코드에서는 Subcomponent1, Subcomponent2를 import해 코드 내에서 재사용하고 있습니다.  
이 때 `components: {Subcomponent1, Subcomponent2}` 구문을 통해 사용할 컴포넌트를 등록해주는 과정이 필요합니다.  
컴포넌트를 사용할 때는 import한 컴포넌트 이름을 이용해 태그를 만들면 됩니다.  

vue에서 컴포넌트는 페이지 전체일 수도 있고, 페이지의 구성 요소 중 하나일 수도 있습니다.  
중요한 점은 컴포넌트 재사용을 통해 통일성을 보장받을 수 있으며, 변경 사항에 대한 수정도 용이해진다는 점입니다.  

## 부모 컴포넌트에서 자식 컴포넌트로 데이터 전달하기 (Props)
자식 컴포넌트에서 렌더링하는 데이터를 부모 컴포넌트에서 가지고 있으면 해당 데이터를 자식 컴포넌트로 넘겨주어야합니다.  
이 때 props를 통해 데이터를 전달합니다.  

데이터를 전달 받는 자식 컴포넌트의 코드는 다음과 같이 작성됩니다.  
{% raw %}
```vue
<template>
    <h2>{{ title }}</h2>
    <p>{{ content }}</p>
</template>
<script>
    export default {
        props: {
            title: {
                type: String,
                default: "페이지 제목"
            },
            content: {
                type: String,
                default: "페이지 내용"
            }
        }
    }
</script>
```
전달받을 데이터를 명시하기 위해 `props`에 전달받을 데이터들을 키로 갖는 오브젝트를 추가하였습니다.  
이 때 `props` 내의 각 데이터들에 데이터 타입과 데이터가 전달되지 않았을 때 사용할 default 값을 정의합니다.  

`props`에 정의된 키는 데이터값으로 사용되므로 `{{ 데이터 키 값 }}` 형태를 사용해 바인딩 처리를 할 수 있습니다.  
{% endraw %}

`props`를 통해 자식 컴포넌트에 데이터를 넘겨주기 위해서 부모 컴포넌트의 코드는 다음과 같이 작성하면 됩니다.  
```vue
<template>
    <div>
        <PageTitle title="전달받은 제목" content="전달받은 내용"/>
    </div>
</template>
<script>
    import PageTitle from "../components/PageTitle"
    export default {
        components: {PageTitle}
    }
</script>
```
`<PageTitle title="전달받은 제목" content="전달받은 내용"/>` 형태처럼 부모 컴포넌트에서 사용하는 자식 컴포넌트에 속성으로 넘겨줄 데이터 키와 값을 추가해줍니다.  
여기서 지정한 값이 자식 컴포넌트의 props의 해당 데이터에 전달됩니다.  

### 전달할 데이터 동적으로 할당하기
부모 컴포넌트에서 데이터를 동적으로 할당하여 전달할 때는 부모 컴포넌트의 data를 사용하면 됩니다.  
input text를 통해 입력값을 받은 후 해당 데이터를 자식 컴포넌트로 보내는 코드는 아래와 같습니다.  
```vue
<template>
    <input type="text" v-model="inputTitle" />
    <input type="text" v-model="inputContent" />
    <PageTitle :title="inputTitle" :content="inputContent"/>
</template>
<script>
    import PageTitle from "../components/PageTitle.vue"
    export default {
        data() {
            return {
                inputTitle: "제목 입력",
                inputContent: "내용 입력"
            }
        },
        components: {PageTitle}
    }
</script>
```
이 코드에서는 inputTitle와 inputContent로 바인딩 된 데이터를 자식 컴포넌트에 보내고 있습니다.  
참고로 이 때 `:`는 `v-bind`의 약어이기 때문에 `:title="inputTitle`와 `v-bind:title="inputTitle`은 동일하게 동작합니다.  

또한 만약 전달하려는 데이터가 숫자형이라면 위에서 정적 데이터를 전달할 때 처럼 보내는 것은 불가능하고. 무조건 v-bind를 사용하여 보내야합니다.  
```js
// 불가 (문자열로 전달됨)
<Subcomponent number="42" />
// 가능 (숫자형으로 전달됨)
<Subcomponent :number="42" />
```
숫자형 외에도 boolean, 배열, 객체(object)의 경우에도 꼭 v-bind를 사용해서 데이터를 전달해야합니다.  

### prop 유효성 검사
자식 컴포넌트에서 props 옵션을 정의할 때 전달받는 데이터 타입, 기본값, 필수 여부를 정할 수 있으며, 유효성 검사 합수를 통해서 유효성을 검사할 수 있습니다.  
유효성 검사를 동반한 경우 props의 형태는 아래와 같습니다.  
```vue
<script>
    export default {
        props: {
            name: {
                type: String,
                required: true,
                validator: function(value) {
                    return ['Olaf', 'Anna', 'Elsa'].indexOf(value) !== -1
                }
            }
        }
    }
</script>
```
